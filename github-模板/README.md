# 这个文件夹怎么用

这是给你上传到 GitHub 的**仓库骨架**，不是让你在这里开发。

## 三步

1. 把 `profile-README.md` 复制出来，重命名为 `README.md`，
   放进一个**名字等于你 GitHub 用户名**的仓库的根目录（必须 Public，否则主页不显示）。
2. 把 `learning-log/` 整个文件夹复制成一个独立仓库 `learning-log`，
   按 `YYYY-MM.md` 每月一个文件，每天写 3 行。
3. `.gitignore` 复制到每个代码项目的根目录（Python 项目通用）。

## 注意

- 所有 `你的用户名` / `你的名字` 都要替换成真实内容
- 别把 API key、密码、身份证写进任何文件
- 这个文件夹本身不用 `git init`；复制出去之后再初始化

## 命令

```bash
# 以 learning-log 为例，复制到工作区外再初始化
cp -r learning-log D:/xueru/Documents/learning-log
cd D:/xueru/Documents/learning-log
git init -b main
git add .
git commit -m "chore: 初始化学习日志"
git remote add origin git@github.com:pythonlearnerX/learning-log.git
git push -u origin main
```