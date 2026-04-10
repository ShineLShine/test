# Git 命令行（获取 / 推送 / 合并 / 冲突解决 / 指定版本）

## 0. 前置准备（一次性）
### 0.1 安装与确认
```bash
git --version
```
### 0.2 配置身份（首次使用建议配置）
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
### 0.3 推荐基础配置（可选）
```bash
# 默认主分支名称
git config --global init.defaultBranch main

# Windows 建议
git config --global core.autocrlf true

# macOS / Linux 建议
git config --global core.autocrlf input
```
---
## 1. 获取项目（clone / fetch / pull）
### 1.1 第一次获取项目：`git clone`
#把远程仓库完整拉到本地（包含提交历史、分支信息）。
```bash
git clone https://github.com/ShineLShine/test.git

#如果使用 SSH：
git clone git@github.com:ShineLShine/test.git

# 克隆到指定目录名 (如果在克隆时没有指定绝对路径目录，则会默认克隆到 C:\Users\用户名)
git clone https://github.com/ShineLShine/test.git my-folder
git clone https://github.com/ShineLShine/test.git C:\Users\用户名\my-folder

# 克隆指定分支
git clone -b <分支名> <仓库地址>
git clone -b develop https://github.com/ShineLShine/test.git
# 只克隆最近 N 次提交（浅克隆）
git clone --depth 1 https://github.com/ShineLShine/test.git
```

#先克隆整个仓库，再切换分支
```
git clone https://github.com/ShineLShine/test.git
cd demo
git switch develop
或旧命令：
git checkout develop

#克隆时查看有哪些分支（可选）
git ls-remote --heads https://github.com/ShineLShine/test.git
```
---
### 1.2 命令行创建分支
#创建分支（不切换）
```
git branch 分支名
#然后手动切换
git switch 分支名
```
#创建并切换到新分支
```
git switch -c 分支名
#或
git checkout -b 分支名
```
#指定分支创建新分支
```
git switch main
git switch -c 分支名
或
git checkout -b 分支名 main
```
#指定 commit 创建分支（老版本紧急修复）
```
git checkout -b hotfix/uart a1b2c3d
```

#推送新分支到远程
```
#第一次推送必须这样
git push -u origin feature/login
#之后就可以
git push
```

#常见错误 & 解决方法
```
#1：clone 后发现不是想要的分支
git switch <分支名>

#2：新分支推不上去（没有关联远程分支）
git push -u origin <分支名>

#3：不知道当前在哪个分支(当前分支前面会有 *)
git branch
```

#示例
```
# 1. 克隆 develop 分支
git clone -b main https://github.com/ShineLShine/test.git
cd test

# 2. 创建功能分支
git switch -c 新分支名

# 3. 开发并提交
git add .
git commit -m "feat: 增加串口自动监听"

# 4. 推送分支
git push -u origin 新分支名
``
```
---
### 1.3 更新远程信息：`git fetch`
```bash
git fetch origin
git branch -r
```
> `fetch` 只下载更新，不会修改当前代码。
---
### 1.4 拉取并合并：`git pull`
```bash
git pull origin main
```
推荐团队常用（保持历史线性）：
```bash
git pull --rebase origin main
```
---
## 2. 推送项目（add / commit / push）
### 2.1 基本流程
```bash
git status

git add .
# 或单个文件
git add src/app.py

git commit -m "feat: add login endpoint"

git push origin main
```
---
### 2.2 新分支首次推送（设置 upstream）
```bash
git checkout -b feature/login
# ... 开发、add、commit
git push -u origin feature/login
```
之后只需要：
```bash
git push
```
---
### 2.3 常见推送失败处理
远程比你新（non-fast-forward）：
```bash
git pull --rebase origin main
git push
```
权限问题排查：
```bash
git remote -v
```
确认 SSH Key / Token / 仓库权限。
---
## 3. 分支合并（merge / rebase）
### 3.1 使用 `merge`
```bash
git checkout main
git pull origin main
git merge feature/login
git push origin main
```
---
### 3.2 使用 `rebase`
```bash
git checkout feature/login
git fetch origin
git rebase origin/main

# 合并回 main
git checkout main
git merge feature/login
git push origin main
```
> ⚠ 不要对已经共享给他人的公共分支随意 rebase。
---
## 4. 合并冲突解决（详细流程）
### 4.1 冲突提示
```text
CONFLICT (content): Merge conflict in src/app.py
```
```bash
git status
```
---
### 4.2 冲突文件示例
```text
<<<<<<< HEAD
console.log("main branch");
=======
console.log("feature branch");
>>>>>>> feature/login
```
---
### 4.3 解决步骤（推荐顺序）
打开冲突文件
删除 `<<<<<<< ======= >>>>>>>` 标记
编写最终代码
标记为已解决
```bash
git add src/app.js
```
继续操作：
```bash
# merge 冲突
git commit

# rebase 冲突
git rebase --continue
```
推送：
```bash
git push
```
---
### 4.4 常用“救命”命令
```bash
git merge --abort
git rebase --abort

git diff
git diff --merge
```
---
## 5. 获取指定版本代码（tag / commit / branch）
### 5.1 查看 tag
```bash
git tag
git tag -l "v1.*"
```
---
### 5.2 切换到 tag（只读，detached HEAD）
```bash
git checkout v1.2.3
```
建议新建分支：
```bash
git checkout -b hotfix/v1.2.3 v1.2.3
```
---
### 5.3 切换到指定 commit
```bash
git log --oneline --decorate --graph --all
git checkout 1a2b3c4
git checkout -b debug/issue-123 1a2b3c4
```
---
### 5.4 切换远程分支
```bash
git fetch origin
git checkout -b release/2026Q2 origin/release/2026Q2
```
---
### 5.5 仅下载某版本源码（不保留历史）
推荐：`git clone` + `git checkout`
或使用 GitHub / GitLab 提供的 Source archive
---
## 6. 日常最常用命令速查表
```text
克隆仓库            git clone <url>
查看状态            git status
查看差异            git diff
添加暂存            git add .
提交                git commit -m "msg"
拉取更新            git pull --rebase
推送                git push
新建分支            git checkout -b <branch>
合并分支            git merge <branch>
解决冲突（merge）    git add ... && git commit
解决冲突（rebase）   git add ... && git rebase --continue
放弃合并/变基        git merge --abort / git rebase --abort
切 tag/commit       git checkout <tag-or-hash>
```
---
