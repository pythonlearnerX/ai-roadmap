# 04 · GitHub 从零到作品集

> 目标：**今天 2 小时内**完成注册 + 配置 + 第一波仓库上传，之后一年只管往里放东西。
> 说明：注册和验证码必须你本人操作（需要你的邮箱和手机），我无法代注册；但仓库骨架、Profile、配置命令我都写好了。

---

## 0. 今天要完成的事（勾完就结束）

- [ ] 注册 GitHub 账号（邮箱验证通过）
- [ ] 开启两步验证（2FA），保存恢复码
- [ ] 本机配置 `user.name` / `user.email` / 默认分支
- [ ] 生成 SSH key 并添加到 GitHub，`ssh -T` 显示成功
- [ ] 建 3 个仓库：`<用户名>`（Profile）、`learning-log`、`ai-roadmap`
- [ ] 把 `github-模板/` 里的文件传上去
- [ ] 申请 GitHub 学生包（有 .edu 邮箱的话）

---

## 1. 注册账号

### 1.1 先想好用户名（这个会跟你一辈子）

规则：全小写、用 `-` 连接、别用一串数字/生日、能在简历上念出来。

| 推荐 | 不推荐 |
| --- | --- |
| `xueru-ai`、`rui-ml`、`zhangsan-dev` | `abc123456`、`xiaoming2004`、`qwerty` |

> 想改用户名可以改（Settings → Account），但旧链接会失效、贡献记录会断，**一次定好**。

### 1.2 注册步骤

1. 打开 `https://github.com/signup`
2. 填邮箱（Gmail / Outlook / QQ 邮箱都行，**用你每天都会看的那个**）
3. 设密码：至少 8 位且含数字和小写字母；建议直接上 15 位以上的密码短语
4. 填用户名（见上表）
5. 邮箱收 6 位验证码，填入
6. 完成人机验证（国内网络可能要多试几次或开代理）

### 1.3 国内访问不稳怎么办

| 方案 | 说明 | 推荐度 |
| --- | --- | --- |
| 代理/加速器 | 浏览器和 Git 都走同一代理，最稳 | ★★★★★ |
| 换网络 | 校园网卡就切手机热点，反之亦然 | ★★★ |
| SSH 走 443 端口 | 只解决 `git push` 的 22 端口被封问题（见 3.3） | ★★★★（必配） |
| 镜像站 | 只读、更新滞后、随时失效 | ★（不推荐长期依赖） |

> **网页打不开 ≠ 不能 push。** 先把 SSH 443 配好，很多"GitHub 用不了"其实是 22 端口被封。

### 1.4 立刻开启 2FA

`Settings → Password and authentication → Two-factor authentication → Enable`
用手机验证器（Microsoft Authenticator / Google Authenticator / 1Password）扫码，
**把恢复码（recovery codes）抄到纸上或密码管理器里**——手机丢了没这个就登不进去。

---

## 2. 本机配置 Git（一次性）

Git 已安装（本机 `git version 2.47.0`）。打开 **Git Bash** 或 PowerShell：

```bash
git config --global user.name "你的名字或拼音"
git config --global user.email "你注册 GitHub 用的邮箱"
git config --global init.defaultBranch main
git config --global core.quotepath false      # 中文文件名不乱码
git config --global pull.rebase false
git config --global --list                     # 检查
```

> **邮箱必须和 GitHub 账号一致**，否则提交记录不会算到你头上（绿格子不亮）。
> 如果不想公开真实邮箱：GitHub → Settings → Emails → 勾选 *Keep my email addresses private*，
> 用它给的 `xxxx@users.noreply.github.com` 作为 `user.email`。

---

## 3. 配置 SSH（配一次，以后不用输密码）

### 3.1 生成密钥

```bash
ssh-keygen -t ed25519 -C "你注册 GitHub 用的邮箱"
```

一路回车（密码可留空）。生成在 `C:\Users\<你>\.ssh\id_ed25519` 和 `.pub`。
> 这个目录属于系统配置，是"C 盘只读"规则里的明确例外，不算违规产出。

### 3.2 添加到 GitHub

```bash
cat ~/.ssh/id_ed25519.pub
```

复制整行输出（以 `ssh-ed25519` 开头），到 GitHub：
`Settings → SSH and GPG keys → New SSH key → 粘贴 → Add SSH key`

### 3.3 测试（Windows 上第一次会问 yes，输入 `yes` 回车）

```bash
ssh -T git@github.com
# 成功输出：Hi <用户名>! You've successfully authenticated...
```

**如果卡住或 `Connection timed out`**：22 端口被封。新建/编辑 `C:\Users\<你>\.ssh\config`：

```text
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```

再执行 `ssh -T git@github.com`。这条配置能救 80% 的"push 不上去"。

---

## 4. 建第一批仓库

### 4.1 在网页建（3 个，都选 Public，**都不要**勾 Add README / .gitignore / license）

| 仓库名 | 用途 |
| --- | --- |
| `YOUR_USERNAME` | 同名仓库 = Profile 主页展示，必须同名、必须 Public |
| `learning-log` | 每天 3 行的学习日志，绿格子来源 |
| `ai-roadmap` | 这份路线 + 你的月度复盘 |

### 4.2 本地初始化并推送（以 learning-log 为例）

```bash
mkdir learning-log && cd learning-log
git init -b main
# 把 github-模板/learning-log/ 里的文件复制进来
git add .
git commit -m "chore: 初始化学习日志仓库"
git remote add origin git@github.com:YOUR_USERNAME/learning-log.git
git push -u origin main
```

### 4.3 之后每天的正常流程

```bash
git status                 # 看改了什么
git add -A
git commit -m "docs: 记录 11-06 学习内容"
git push
```

> 记住这 4 条就够用一年。其余命令用到再查，不必背 `git rebase`。

---

## 5. 仓库规范（面试官会点进来看）

### 5.1 命名

- 全小写 `kebab-case`：`ai-job-radar`、`rag-knowledge-base`、`bert-text-classifier`
- 不要：`项目1`、`test`、`新建文件夹 (2)`、`my_project_final_final`

### 5.2 每个项目仓库的 README 首屏模板

````markdown
# 项目名

> 一句话：给谁解决什么问题。

![demo](docs/demo.gif)

## 效果
| 指标 | 数值 |
| --- | --- |
| 准确率 / F1 | 0.9x |
| P99 延迟 | xx ms |
| QPS | xx |

## 架构
（一张图，draw.io / Excalidraw / mermaid 都行）

## 快速开始
```bash
git clone ...
cd ...
pip install -r requirements.txt
cp .env.example .env   # 填你的 key
docker compose up
```

## 我踩的坑
1. ……
2. ……

## 局限与下一步
````

### 5.3 目录与杂项文件

```text
项目/
├─ README.md
├─ .gitignore          # 必须有
├─ .env.example        # 只放变量名，不放真值
├─ LICENSE             # MIT 即可
├─ pyproject.toml 或 requirements.txt
├─ Dockerfile
├─ src/  tests/  docs/  scripts/
└─ data/               # 只放 sample，大文件不进 git
```

### 5.4 `.gitignore`（Python 项目直接抄）

`github-模板/.gitignore` 已备好，核心几条：

```text
__pycache__/
*.py[cod]
.venv/
venv/
.env
*.log
data/raw/
outputs/
.ipynb_checkpoints/
.DS_Store
```

> 大文件（模型权重、数据集、视频）**不要进 git**：放网盘，README 里给链接；
> 实在要放用 Git LFS，但免费额度有限，能避就避。

---

## 6. Commit 规范（Conventional Commits 中文版）

格式：`<类型>: <做了什么>`

| 类型 | 用于 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 修 bug |
| `docs` | 文档 |
| `refactor` | 重构（不改行为） |
| `test` | 测试 |
| `chore` | 杂务（依赖、配置） |
| `perf` | 性能 |
| `ci` | CI 配置 |

好的：`feat: 支持按城市筛选岗位数据`、`fix: 修复薪资解析 14薪 报错`
差的：`update`、`修改`、`1`、`提交`

> 每天至少 1 次 commit。**commit 历史就是你的工作日志**，面试官真的会翻。

---

## 7. Profile README（你的门面）

仓库名 = 用户名，在里面放 `README.md`，内容已备好：`github-模板/profile-README.md`。

要点（别写成长篇简历）：

1. 一句话定位：`AI 大三学生 · 方向：大模型应用开发`
2. 3–4 个置顶项目，每个一行 + 链接
3. 技术栈徽章（可选，别堆 50 个）
4. 正在学什么（每月更新一行）
5. 联系方式

---

## 8. 练一次分支 + PR（20 分钟，值回票价）

```bash
git checkout -b docs/add-week1
# 改点东西
git add -A && git commit -m "docs: 补充第一周计划"
git push -u origin docs/add-week1
```

网页会提示 `Compare & pull request` → 创建 → 自己 review → Merge → 删分支。
目的不是学流程，是**以后参与开源时知道怎么提 PR**。

---

## 9. 安全红线（踩了会很难看）

| 红线 | 后果 | 正确做法 |
| --- | --- | --- |
| 把 API key 提交上去 | 机器人 5 分钟内扫到并盗刷，账单可能是几千块 | `.env` + `.gitignore`，仓库里只放 `.env.example` |
| 提交个人信息/身份证/成绩单 | 隐私泄露 | 不放，或打码 |
| 提交公司内部代码 | 违约、开除 | 实习期间绝不外传 |
| 用公司数据做个人项目 | 同上 | 用公开数据集 |
| fork 别人的项目当自己的 | 面试一问就穿 | 注明来源，写"复现/改进" |

**已经提交了密钥怎么办**：立刻去平台控制台**吊销并重新生成**（这是唯一有效的动作），
然后 `git filter-repo` 清理历史。只删文件不吊销 = 没做。

---

## 10. 绿格子策略（有用，但别表演）

- `learning-log` 每天提交一次，一条 3 行，全年 300+ 天有记录 → 面试时是很强的"持续学习"证据
- 但**不要**为了格子提交垃圾（改空格、`update`）——历史是可读的，反而扣分
- 真正的加分项是：项目仓库有清晰的 issue、PR、release、CI 徽章

---

## 11. GitHub 学生包（有学籍就申请）

`https://education.github.com/pack` → 用学校邮箱（`.edu.cn`）申请学生认证。
通常包含 Copilot 免费额度、云服务抵用券、域名等。审核几天到几周，**现在申请，正好在 M1 用上**。
> 学校邮箱不支持时，可用学信网/学生证截图走人工验证。

---

## 12. 常见错误速查

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| `Permission denied (publickey)` | key 没加上 / 加错账号 | 重跑 3.2，`ssh -T` 验证 |
| `Connection timed out` on port 22 | 22 被封 | 用 3.3 的 443 配置 |
| `rejected: fetch first` | 远程有本地没有的提交 | `git pull --rebase` 后再 push |
| `src refspec main does not match any` | 还没 commit | 先 `git add . && git commit` |
| 中文文件名显示 `\344\275\240` | quotepath | `git config --global core.quotepath false` |
| push 很慢/中断 | 大文件或网络 | 检查是否误提交了模型/数据；配代理 |
| 绿格子不亮 | 邮箱不一致 / 仓库是 fork | 改 `user.email` 为账号邮箱 |
| 忘记密码且无 2FA 恢复码 | 锁号 | 用恢复码；没有就联系 GitHub 支持（很慢） |

---

## 13. 今天之后

- 仓库建好就**不要再折腾配置**（不要花三天配 vim、配终端、配主题，这是典型的逃避行为）
- 把 `ai-roadmap` 的仓库地址填进简历和个人主页
- 每周日往 `learning-log` 写周报，每月往 `ai-roadmap` 提一次复盘

---

记录人：DSH ｜ 2026-11-05 ｜ 命令均在本机 Git 2.47 / Windows 环境下核对过语法