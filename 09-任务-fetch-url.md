# 09 · 任务书：把 `fetch_url` 从 TODO 变成能跑的代码

> 目标：给 `ai-job-radar/job_radar/fetcher.py` 补上「重试、限速、断点续跑」三件事。
> 工期：90–120 分钟。做完这一份，你就有了「工程化的网络请求」这一块能力，面试也问得到。

---

## 0. 先看清约束（很重要，不然你会卡死）

| 约束 | 说明 |
| --- | --- |
| **这台机器连不上 PyPI** | 不能用 `pip install requests` / `httpx` / `tenacity`。**只能用标准库 `urllib`**。 |
| 不许改已有的 35 个测试 | 它们是回归网，改坏了说明你把别的地方弄崩了。 |
| 公共 API 不能乱改 | 已有调用方（`cli.py`）用的是 `RawJob`，别换返回类型。 |
| 不做真实站点适配 | 这个任务只做「请求的可靠性」，选择器/解析留给下一份任务。 |

---

## 1. 最终要达到的接口

```python
def fetch_url(
    url: str,
    *,
    delay: float = 2.0,     # 两次请求之间的最小间隔（秒）
    timeout: float = 10.0,  # 单次请求超时
    retries: int = 3,       # 最多尝试次数（含第一次）
    backoff: float = 1.0,   # 退避基数：第 n 次重试前等 backoff * 2**(n-1)
) -> RawJob: ...


def load_seen(path: str | Path) -> set[str]: ...
def mark_seen(path: str | Path, url: str) -> None: ...
```

**行为约定**（测试就是按这个写的）：

1. 成功 → 返回 `RawJob(text=页面文本, source=url, url=url)`，编码按响应头 `charset`，没有就 utf-8。
2. **5xx 和网络错误要重试**，最多 `retries` 次；全失败抛 `FetchError`。
3. **4xx 不重试**（客户端错了，重试一百次也没用），直接抛 `FetchError`。
4. 两次请求之间至少隔 `delay` 秒（同一进程内全局）。
5. `load_seen` 读一个每行一个 URL 的文件，返回集合；文件不存在返回空集。
6. `mark_seen` 追加一行，父目录自动创建，重复写不报错。

---

## 2. 三个增量，一个一个来

### 增量 1 · 重试与异常（40 分钟）

要点：

- 用一个 `for attempt in range(1, retries + 1)` 循环。
- `except urllib.error.HTTPError as e:` 先判断 `400 <= e.code < 500` → 立刻 `raise FetchError`。
- `except (urllib.error.URLError, TimeoutError, OSError)` → 可重试。
- 重试前 `time.sleep(backoff * 2 ** (attempt - 1))`，最后一次失败直接抛。
- 自定义 `class FetchError(Exception)`，消息里带上 URL 和最后一次的错误。

伪代码：

```python
last_error = None
for attempt in range(1, retries + 1):
    try:
        return _do_request(url, timeout)
    except HTTPError as exc:
        if 400 <= exc.code < 500:
            raise FetchError(f"{url} 客户端错误 {exc.code}") from exc
        last_error = exc
    except (URLError, TimeoutError, OSError) as exc:
        last_error = exc
    if attempt < retries:
        time.sleep(backoff * 2 ** (attempt - 1))
raise FetchError(f"{url} 重试 {retries} 次仍失败：{last_error}") from last_error
```

### 增量 2 · 限速（20 分钟）

要点：

- 模块级变量 `_last_request_at: float = 0.0`。
- 每次真正发请求前算 `wait = delay - (time.monotonic() - _last_request_at)`，`wait > 0` 才 sleep。
- 请求发出后立刻更新 `_last_request_at`（不是等响应回来）。
- **不要**无脑 `sleep(delay)`：第一次请求会被白等 2 秒。

> 为什么用 `time.monotonic()` 不用 `time.time()`：系统时间会被校时跳变，`monotonic` 不会。

### 增量 3 · 断点续跑（20 分钟）

要点：

- `seen.txt` 每行一个 URL，UTF-8。
- `load_seen`：`Path(path).read_text().splitlines()`，去掉空行和首尾空白，返回 `set`。
- `mark_seen`：`path.parent.mkdir(parents=True, exist_ok=True)` 然后 `open("a")` 追加。
- 在 `cli.py` 的 `run` 里用上：跳过 `load_seen` 里已有的 URL，抓完一条 `mark_seen` 一条。

---

## 3. 测试（复制到 `tests/test_fetcher.py`）

先让它们全红，再让它们全绿。**测试就是需求文档**，别改测试去迁就代码。

```python
"""fetch_url 的可靠性：重试、限速、断点续跑。"""

from __future__ import annotations

import time
import urllib.error
from unittest import mock

import pytest

from job_radar.fetcher import FetchError, fetch_url, load_seen, mark_seen


def _fake_response(body: bytes = b"ok", charset: str | None = "utf-8"):
    resp = mock.MagicMock()
    resp.read.return_value = body
    resp.headers.get_content_charset.return_value = charset
    resp.__enter__.return_value = resp
    return resp


def test_returns_text_and_url(monkeypatch):
    with mock.patch("urllib.request.urlopen", return_value=_fake_response("你好".encode("utf-8"))):
        job = fetch_url("https://example.com/a", delay=0, backoff=0)
    assert job.text == "你好"
    assert job.url == "https://example.com/a"


def test_retries_on_5xx_then_succeeds(monkeypatch):
    calls = []

    def fake(url, *args, **kwargs):
        calls.append(url)
        if len(calls) < 3:
            raise urllib.error.HTTPError(url, 503, "busy", {}, None)
        return _fake_response(b"ok")

    monkeypatch.setattr("urllib.request.urlopen", fake)
    job = fetch_url("https://example.com/b", delay=0, backoff=0, retries=3)
    assert job.text == "ok"
    assert len(calls) == 3


def test_4xx_is_not_retried(monkeypatch):
    calls = []

    def fake(url, *args, **kwargs):
        calls.append(url)
        raise urllib.error.HTTPError(url, 404, "not found", {}, None)

    monkeypatch.setattr("urllib.request.urlopen", fake)
    with pytest.raises(FetchError):
        fetch_url("https://example.com/c", delay=0, backoff=0, retries=3)
    assert len(calls) == 1


def test_network_error_gives_up_after_retries(monkeypatch):
    calls = []

    def fake(url, *args, **kwargs):
        calls.append(url)
        raise urllib.error.URLError("connection refused")

    monkeypatch.setattr("urllib.request.urlopen", fake)
    with pytest.raises(FetchError):
        fetch_url("https://example.com/d", delay=0, backoff=0, retries=3)
    assert len(calls) == 3


def test_rate_limit_spaces_out_requests(monkeypatch):
    with mock.patch("urllib.request.urlopen", return_value=_fake_response()):
        started = time.monotonic()
        fetch_url("https://example.com/e", delay=0.2, backoff=0)
        fetch_url("https://example.com/f", delay=0.2, backoff=0)
        elapsed = time.monotonic() - started
    assert elapsed >= 0.18


def test_seen_file_roundtrip(tmp_path):
    path = tmp_path / "nested" / "seen.txt"
    assert load_seen(path) == set()

    mark_seen(path, "https://a.com/1")
    mark_seen(path, "https://a.com/2")
    mark_seen(path, "https://a.com/1")  # 重复写不报错

    assert load_seen(path) == {"https://a.com/1", "https://a.com/2"}
```

跑：

```powershell
cd "D:\xueru\Documents\github\ai-job-radar"
.\.venv\Scripts\python.exe -m pytest tests\test_fetcher.py -q
```

一开始会 `ImportError: cannot import name 'FetchError'` —— **这就是起点**。

---

## 4. 验收（全部满足才算完成）

- [ ] `pytest -q` 全绿（35 + 6 = 41 个），CI 上也是绿的
- [ ] `tests/test_fetcher.py` 的 6 个用例一个都没改
- [ ] 4xx 请求只发 1 次（用 `-v` 或打日志能看出来）
- [ ] 连续两次请求间隔 ≥ delay
- [ ] `seen.txt` 跑两次第二次能跳过
- [ ] 能口头解释：为什么 4xx 不重试、为什么用 `monotonic`、为什么限速要放在发请求之前

提交：

```powershell
git add -A
git commit -m "feat: fetch_url 支持重试、限速与断点续跑"
git push
```

---

## 5. 卡住了按这个顺序排查

1. `ImportError` → 你还没定义 `FetchError`，先加类。
2. `assert len(calls) == 3` 但实际是 1 → 你在 4xx 分支里也跳过了重试，检查 `if 400 <= code < 500` 的位置。
3. `test_rate_limit` 不稳定 → 你用了 `time.time()`，换成 `time.monotonic()`。
4. 测试互相影响 → 模块级 `_last_request_at` 是全局状态，测试里记得每次都传 `delay=0`。
5. `mock.patch("urllib.request.urlopen")` 没生效 → `fetcher.py` 里必须写 `urllib.request.urlopen(...)`，不能 `from urllib.request import urlopen`。

---

## 6. 做完之后你会得到什么

- 面试可讲：「我做的采集层有指数退避重试、全局限速、断点续跑，4xx 不重试避免打爆对方」
- 代码可复用：这套模式在 RAG 项目里抓文档、调 API 时一模一样
- 一个真问题：**限速是进程内的，多进程会失效**——这就是你以后要学「分布式限流」的入口

---

记录人：DSH ｜ 2026-11-11