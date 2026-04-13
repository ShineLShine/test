# Git 命令行（获取 / 推送 / 合并 / 冲突解决 / 指定版本）

## 0. 前置准备（一次性）
### 0.1 安装与确认
```bash
git --version
```
### 0.2 配置身份（首次使用建议配置）
```bash
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
```
### 0.3 推荐基础配置（可选）
```bash
默认主分支名称
git config --global init.defaultBranch main

Windows 建议
git config --global core.autocrlf true

macOS / Linux 建议
git config --global core.autocrlf input
```
---
## 1. 获取项目（clone / fetch / pull）
### 1.1 第一次获取项目：`git clone`
#### 把远程仓库完整拉到本地（包含提交历史、分支信息）。
```
git clone https://github.com/ShineLShine/test.git
如果使用 SSH：
git clone git@github.com:ShineLShine/test.git
```
#### 克隆到指定目录名 (如果在克隆时没有指定绝对路径目录，则会默认克隆到 C:\Users\用户名)
```
git clone https://github.com/ShineLShine/test.git my-folder
git clone https://github.com/ShineLShine/test.git C:\Users\用户名\my-folder
```
#### 克隆指定分支
```
git clone -b <分支名> <仓库地址>
git clone -b develop https://github.com/ShineLShine/test.git
```

#### 只克隆最近 N 次提交（浅克隆）
```
git clone --depth 1 https://github.com/ShineLShine/test.git
```

#### 先克隆整个仓库，再切换分支
```
git clone https://github.com/ShineLShine/test.git
cd test
git switch develop

或旧命令：
git checkout develop
```

#### 克隆时查看有哪些分支（可选）
```
git ls-remote --heads https://github.com/ShineLShine/test.git
```
![示例图片](/src/1.png)
---
### 1.2 命令行创建分支 `git branch`
#### 创建分支（不切换）
```
git branch 分支名
手动切换
git switch 分支名
```
#### 创建并切换到新分支
```
git switch -c 分支名
或
git checkout -b 分支名
```
#### 指定分支创建新分支
```
git switch main
git switch -c 分支名
或
git checkout -b 分支名 main
```
#### 指定 commit 创建分支（老版本紧急修复）
```
git checkout -b hotfix/uart a1b2c3d
```

#### 推送新分支到远程
```
#第一次推送必须这样(第一次推送新分支时，Git 不知道你要推到哪个远程、哪个分支)
git push -u origin feature/login
#之后就可以（因为 -u 已经帮你“记住”了这个关系）
git push
```

#### 常见错误 & 解决方法
```
#1：clone 后发现不是想要的分支
git switch <分支名>

#2：新分支推不上去（没有关联远程分支）
git push -u origin <分支名>

#3：不知道当前在哪个分支(当前分支前面会有 *)
git branch
```
![示例图片](/src/2.png)

---
### 1.3 查看分支 `git branch`
```
git branch
#查看远程分支
git branch -r
```
---
### 1.4 更新远程信息 `git fetch`
```bash
git fetch origin
git branch -r
```
> `fetch` 只下载更新，不会修改当前代码。
---
### 1.5 拉取并合并：`git pull`
```bash
拉取指定分支
git pull origin main

推荐团队常用（保持历史线性）：
git pull
--rebase origin main
```
---
## 2. 推送项目（add / commit / push）
### 2.1 基本流程
```bash
查看当前修改状态
git status

添加所有修改(注意：这里的add .中间必须有一个空格)
git add .
# 或指定文件
git add src/app.py

提交本地修改（commit）
git commit -m "feat: add login endpoint"

推送到远程仓库（push）
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
### 3.1 确认当前是否有未提交的代码（很重要）
```
git status
# 如果看到
working tree clean
# 说明可以继续;否则请先
git add .
git commit -m "save work"
```

### 3.1 使用 `merge` 方法
```bash
# 切换到main主分支
git switch main
# 拉取 main 最新代码（避免合并老代码）
git pull origin main
# 执行合并（shine → main）
git merge shine
# 如有冲突，标记解决
git add <冲突文件
# 完成合并提交
git commit
推送合并后的 main 到远程
git push origin main
```
---
### 3.2 使用 `rebase` 方法
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
### 4.0 没有冲突，那就制造冲突
#### ⚠️制造merge冲突
```
1️⃣ 当前在 main 分支
git switch main

2️⃣ 新建一个测试文件
echo "Hello Git" > conflict.txt

3️⃣ 提交到 main
git add conflict.txt
git commit -m "init: add conflict test file"
此时 conflict.txt 内容是：conflict.txt

4️⃣ 从 main 创建 shine 分支
git switch -c shine

5️⃣ 在 shine 分支修改同一行内容，把 conflict.txt 改成：Hello from shine 提交
git add conflict.txtgit
git commit -m "shine: modify conflict.txt"

6️⃣ 切回 main 分支
git switch main

7️⃣ 在 main 分支也修改同一行内容，把 conflict.txt 改成：Hello from main 提交
git add conflict.txt
git commit -m "main: modify conflict.txt"

8️⃣ 执行合并
git merge shine

💥 出现冲突
CONFLICT (content): Merge conflict in conflict.txt
Automatic merge failed; fix conflicts and then commit the result.

9️⃣ 查看冲突内容
打开文件你会看到 Git 的冲突标记
<<<<<<< HEAD
Hello from main
=======
Hello from shine
>>>>>>> shine
![示例图片](/src/4.png)

🔟 解决冲突
✅ 方法 1：只保留 main
✅ 方法 2：只保留 shine
✅ 方法 3：自己写最终版本
⚠️ 删除所有冲突标记 <<<<<<< ======= >>>>>>>

告诉 Git：冲突解决完成(会打开编辑器，直接保存即可)
git add conflict.txt
git commit
![示例图片](/src/5.png)

```

### 4.1 冲突是怎么产生的?
满足这 3 个条件之一就会产生冲突
* ✅ 同一个文件
* ✅ 同一段代码
* ✅ 不同分支都改过
  
### 4.2 冲突提示
```text
CONFLICT (content): Merge conflict in .....

git status
```
---
### 4.3 冲突文件示例
```text
<<<<<<< HEAD
console.log("main branch");
=======
console.log("feature branch");
>>>>>>> feature/login
```
---
### 4.4 解决冲突步骤
#### 放弃本次合并（不想合了）
```
git merge --abort
```

#### ✅ 方法一
```
1. 手动修改代码
你决定最终代码如下：
Cprintf("UART initialization failed\n");

2. 并删除所有冲突标志：
Plain Text<<<<<<<=======>>>>>>>显示更多行

3. 标记冲突已解决
git add src/main.c

4. 完成合并提交（Git 会自动生成 merge commit 信息）
git commit
```

#### ✅ 方法二
```
1. 手动修改代码
你决定最终代码如下：
Cprintf("UART initialization failed\n");

2. 并删除所有冲突标志：
Plain Text<<<<<<<=======>>>>>>>显示更多行

3. merge 冲突
git commit

4. rebase 冲突
git rebase --continue

5. 推送
git push
```
---
### 4.5 常用“救命”命令
```bash
git merge --abort
git rebase --abort

git diff
git diff --merge
```

### 4.6 合并提交（merge commit）的提交说明界面
![示例图片](/src/3.png)
#### ✅ 方案一（最常用）：直接接受默认提交说明
* 1️⃣ 按 Esc（确保不在编辑模式）
* 2️⃣ 输入：:wq
* 3️⃣ 按 Enter

#### ✅ 方案二：修改提交说明再保存
* 1️⃣ 按 i（进入编辑模式）
* 2️⃣ 把第一行改成你想要的，例如：merge: 合并 shine 分支到 main
* 3️⃣ 按 Esc
* 4️⃣ 输入：:wq
* 5️⃣ 按 Enter

#### ❌ 方案三：放弃这次合并（不建议用）
* ⚠️ 如果你不想完成这次合并：:q!
* ⚠️ 这样会终止 merge
* ⚠️ 一般只在“合并错分支”的情况下用

#### ✅怎么确认合并完成了
退出 Vim 之后，看到命令行没有报错，再执行：
```
git status
```
如果看到：
```
On branch main
nothing to commit, working tree clean
```
说明：shine → main 合并已完成,本地 main 是最新的

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
---
## 6. 常用排错
### 6.1 撤销修改（未 add）
```
git checkout -- src/main.c
```
### 6.2 撤销 add（回到工作区）
```
git reset HEAD src/main.c
```
### 6.3 查看文件是谁改的
```
git blame src/main.c
```
### 6.4 临时保存代码（stash）
```
git stash
git pull
git stash pop
```
## 7. 删除本地与远程分支
### 7.1 删除本地分支
```
git branch -d shine
```
### 7.2 删除远程分支
```
git push origin --delete shine
```

## 8. 日常最常用命令速查表
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
解决冲突（merge）   git add ... && git commit
解决冲突（rebase）  git add ... && git rebase --continue
放弃合并/变基       git merge --abort / git rebase --abort
切 tag/commit       git checkout <tag-or-hash>
```
![示例图片](/src/dp.JPG)
![示例图片](/src/hs.jpg)
![示例图片](/src/image.jpg)

---
