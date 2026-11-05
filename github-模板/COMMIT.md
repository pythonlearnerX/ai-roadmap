# Commit 规范速查

格式：`<类型>: <做了什么>`

| 类型 | 用于 | 例子 |
| --- | --- | --- |
| `feat` | 新功能 | `feat: 支持按城市筛选岗位` |
| `fix` | 修 bug | `fix: 修复薪资解析 14薪 报错` |
| `docs` | 文档 | `docs: 补充 RAG 架构说明` |
| `refactor` | 重构 | `refactor: 抽出检索模块` |
| `test` | 测试 | `test: 补充清洗函数单测` |
| `chore` | 杂务 | `chore: 升级依赖版本` |
| `perf` | 性能 | `perf: 批量推理提速 3 倍` |
| `ci` | CI 配置 | `ci: 添加 pytest 工作流` |

## 不要这么写

`update` ｜ `修改` ｜ `1` ｜ `提交` ｜ `final` ｜ `fix bug`

## 一天至少一次

commit 历史就是你的工作日志。面试官真的会翻。
一天没有任何 commit，就等于那天没发生过。