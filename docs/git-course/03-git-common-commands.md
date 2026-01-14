# 03. Git常用的代码拉取-修改-提交-推送命令

## 目录
- [Git工作流程概述](#git工作流程概述)
- [仓库初始化](#仓库初始化)
- [克隆远程仓库](#克隆远程仓库)
- [查看仓库状态](#查看仓库状态)
- [添加和提交更改](#添加和提交更改)
- [推送到远程仓库](#推送到远程仓库)
- [拉取远程更新](#拉取远程更新)
- [常用命令速查表](#常用命令速查表)

## Git工作流程概述

Git有三个主要的工作区域：

```
工作区              暂存区              本地仓库            远程仓库
(Working Directory) (Staging Area)      (Local Repository)  (Remote Repository)
    |                   |                    |                   |
    |-- git add ------->|                    |                   |
    |                   |-- git commit ----->|                   |
    |                   |                    |-- git push ------>|
    |<--------------- git pull/fetch ----------------------------|
```

### 三个区域详解

1. **工作区（Working Directory）**
   - 实际操作的目录
   - 包含项目的实际文件

2. **暂存区（Staging Area/Index）**
   - 临时存储区域
   - 存放下次提交的文件列表

3. **本地仓库（Local Repository）**
   - 存储项目的完整历史
   - 位于`.git`目录

4. **远程仓库（Remote Repository）**
   - 托管在服务器上的仓库
   - 用于团队协作

## 仓库初始化

### 创建新仓库

```bash
# 在当前目录创建新的Git仓库
git init

# 在指定目录创建新仓库
git init <project-name>

# 创建裸仓库（通常用于服务器）
git init --bare
```

**示例**：

```bash
# 创建项目目录并初始化
mkdir my-project
cd my-project
git init

# 输出：Initialized empty Git repository in /path/to/my-project/.git/
```

### 查看初始化后的目录结构

```bash
ls -la

# 会看到.git目录
# .git/目录包含所有Git需要的元数据和对象数据库
```

## 克隆远程仓库

### 基本克隆

```bash
# 使用HTTPS克隆
git clone https://github.com/username/repository.git

# 使用SSH克隆（需要配置SSH密钥）
git clone git@github.com:username/repository.git

# 克隆到指定目录
git clone <repository-url> <directory-name>

# 克隆指定分支
git clone -b <branch-name> <repository-url>

# 浅克隆（只克隆最近的历史，节省空间和时间）
git clone --depth 1 <repository-url>
```

**示例**：

```bash
# 克隆一个GitHub仓库
git clone https://github.com/torvalds/linux.git

# 克隆并重命名目录
git clone https://github.com/torvalds/linux.git linux-kernel

# 只克隆main分支
git clone -b main --single-branch https://github.com/username/repo.git
```

## 查看仓库状态

### git status - 查看状态

```bash
# 查看详细状态
git status

# 查看简短状态
git status -s
# 或
git status --short
```

**状态输出解释**：

```bash
# 简短状态标记说明：
# ?? - 未跟踪的文件
# A  - 新添加到暂存区的文件
# M  - 修改过的文件
# D  - 删除的文件
# R  - 重命名的文件
# C  - 复制的文件
# U  - 更新但未合并的文件
```

**示例输出**：

```bash
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        new-file.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

### git diff - 查看差异

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与最后一次提交的差异
git diff --staged
# 或
git diff --cached

# 查看工作区与最后一次提交的差异
git diff HEAD

# 比较两个提交之间的差异
git diff <commit1> <commit2>

# 查看特定文件的差异
git diff <file>

# 查看统计信息
git diff --stat
```

## 添加和提交更改

### git add - 添加到暂存区

```bash
# 添加特定文件
git add <file>

# 添加多个文件
git add <file1> <file2> <file3>

# 添加当前目录下所有变更
git add .

# 添加所有变更（包括删除）
git add -A
# 或
git add --all

# 添加所有已跟踪文件的修改
git add -u
# 或
git add --update

# 交互式添加
git add -i

# 部分添加文件内容
git add -p <file>
```

**示例**：

```bash
# 场景：添加新功能
echo "new feature" > feature.txt
git add feature.txt

# 场景：添加所有修改
git add -A

# 场景：只添加特定类型文件
git add *.js

# 场景：交互式选择要添加的改动
git add -p
```

### git commit - 提交更改

```bash
# 提交暂存区的内容
git commit -m "提交信息"

# 提交时打开编辑器写详细信息
git commit

# 跳过暂存区直接提交所有已跟踪文件的修改
git commit -a -m "提交信息"
# 或
git commit -am "提交信息"

# 修改最后一次提交
git commit --amend -m "新的提交信息"

# 修改最后一次提交，不改变提交信息
git commit --amend --no-edit

# 提交时显示差异
git commit -v
```

**良好的提交信息格式**：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**示例**：

```
feat(user): 添加用户登录功能

- 实现用户名密码验证
- 添加JWT token生成
- 添加登录状态缓存

Closes #123
```

**提交类型（type）**：
- `feat`: 新功能
- `fix`: 修复bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建/工具变更

**示例提交**：

```bash
# 简单提交
git commit -m "fix: 修复登录页面样式问题"

# 详细提交
git commit -m "feat: 添加用户注册功能" -m "实现邮箱验证和密码加密"

# 修改上一次提交
git commit --amend -m "fix: 修复登录和注册页面样式问题"
```

## 推送到远程仓库

### git remote - 管理远程仓库

```bash
# 查看远程仓库
git remote

# 查看远程仓库详细信息
git remote -v

# 添加远程仓库
git remote add <name> <url>

# 删除远程仓库
git remote remove <name>
# 或
git remote rm <name>

# 重命名远程仓库
git remote rename <old-name> <new-name>

# 修改远程仓库URL
git remote set-url <name> <new-url>

# 查看远程仓库详细信息
git remote show <name>
```

**示例**：

```bash
# 添加origin远程仓库
git remote add origin https://github.com/username/repo.git

# 查看远程仓库
git remote -v
# 输出：
# origin  https://github.com/username/repo.git (fetch)
# origin  https://github.com/username/repo.git (push)

# 修改为SSH URL
git remote set-url origin git@github.com:username/repo.git
```

### git push - 推送到远程

```bash
# 推送到远程仓库
git push <remote> <branch>

# 推送当前分支到origin
git push origin main

# 推送所有分支
git push --all

# 推送标签
git push --tags

# 首次推送并设置上游分支
git push -u origin main
# 或
git push --set-upstream origin main

# 强制推送（危险操作，慎用）
git push -f
# 或
git push --force

# 更安全的强制推送
git push --force-with-lease

# 删除远程分支
git push origin --delete <branch-name>
```

**示例**：

```bash
# 首次推送新仓库
git push -u origin main

# 之后的推送只需要
git push

# 推送特定分支
git push origin feature-branch

# 删除远程分支
git push origin --delete old-feature
```

## 拉取远程更新

### git fetch - 获取远程更新

```bash
# 获取所有远程仓库的更新
git fetch

# 获取指定远程仓库的更新
git fetch <remote>

# 获取指定远程分支的更新
git fetch <remote> <branch>

# 获取所有远程仓库的所有分支
git fetch --all

# 删除本地已不存在于远程的分支引用
git fetch --prune
# 或
git fetch -p
```

### git pull - 拉取并合并

```bash
# 拉取并合并当前分支
git pull

# 拉取指定远程和分支
git pull <remote> <branch>

# 使用rebase方式拉取
git pull --rebase

# 只在可以快进时才拉取
git pull --ff-only

# 拉取所有远程分支
git pull --all
```

**fetch vs pull的区别**：

```bash
# git pull = git fetch + git merge
git pull origin main

# 等价于
git fetch origin main
git merge origin/main

# 使用rebase方式
git pull --rebase origin main

# 等价于
git fetch origin main
git rebase origin/main
```

**示例工作流**：

```bash
# 方式1：使用pull（自动合并）
git pull origin main
# 解决冲突（如果有）
git push origin main

# 方式2：使用fetch（手动控制）
git fetch origin main
git diff main origin/main  # 查看差异
git merge origin/main      # 决定是否合并
git push origin main
```

## 常用命令速查表

### 基本操作

| 命令 | 说明 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git status` | 查看状态 |
| `git add <file>` | 添加文件到暂存区 |
| `git add .` | 添加所有变更 |
| `git commit -m "msg"` | 提交更改 |
| `git push` | 推送到远程 |
| `git pull` | 拉取远程更新 |

### 查看信息

| 命令 | 说明 |
|------|------|
| `git diff` | 查看未暂存的改动 |
| `git diff --staged` | 查看已暂存的改动 |
| `git log` | 查看提交历史 |
| `git log --oneline` | 简洁的提交历史 |
| `git show <commit>` | 查看提交详情 |

### 远程操作

| 命令 | 说明 |
|------|------|
| `git remote -v` | 查看远程仓库 |
| `git remote add origin <url>` | 添加远程仓库 |
| `git fetch` | 获取远程更新 |
| `git pull` | 拉取并合并 |
| `git push -u origin main` | 推送并设置上游 |

## 完整工作流程示例

### 场景1：创建新项目

```bash
# 1. 创建项目目录
mkdir my-project
cd my-project

# 2. 初始化Git仓库
git init

# 3. 创建README文件
echo "# My Project" > README.md

# 4. 添加到暂存区
git add README.md

# 5. 首次提交
git commit -m "Initial commit"

# 6. 添加远程仓库
git remote add origin https://github.com/username/my-project.git

# 7. 推送到远程
git push -u origin main
```

### 场景2：参与现有项目

```bash
# 1. 克隆项目
git clone https://github.com/username/project.git
cd project

# 2. 查看状态
git status

# 3. 创建新文件
echo "console.log('Hello');" > app.js

# 4. 查看状态
git status

# 5. 添加文件
git add app.js

# 6. 提交
git commit -m "feat: 添加app.js入口文件"

# 7. 推送到远程
git push origin main
```

### 场景3：日常开发流程

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 修改文件
vim index.html

# 3. 查看修改
git diff

# 4. 添加修改
git add index.html

# 5. 提交
git commit -m "fix: 修复首页显示问题"

# 6. 推送
git push origin main
```

### 场景4：多文件修改

```bash
# 1. 修改多个文件
vim src/app.js
vim src/utils.js
vim README.md

# 2. 查看所有修改
git status

# 3. 查看具体改动
git diff

# 4. 分别提交不同类型的修改
git add src/
git commit -m "feat: 添加新功能模块"

git add README.md
git commit -m "docs: 更新README文档"

# 5. 推送所有提交
git push origin main
```

## 最佳实践

### 1. 频繁提交

```bash
# 好的习惯：小步提交
git add feature.js
git commit -m "feat: 添加用户验证"

git add tests/feature.test.js
git commit -m "test: 添加用户验证测试"

# 而不是一次性提交所有改动
```

### 2. 写好提交信息

```bash
# ✅ 好的提交信息
git commit -m "fix: 修复用户登录时的空指针异常"

# ❌ 不好的提交信息
git commit -m "fix bug"
git commit -m "update"
git commit -m "修改"
```

### 3. 推送前先拉取

```bash
# 先拉取远程更新
git pull origin main

# 解决可能的冲突

# 再推送
git push origin main
```

### 4. 使用.gitignore

```bash
# 创建.gitignore文件
cat > .gitignore << EOF
node_modules/
*.log
.env
.DS_Store
dist/
EOF

git add .gitignore
git commit -m "chore: 添加.gitignore文件"
```

## 总结

本节学习了Git的核心命令：

✅ 初始化和克隆仓库  
✅ 查看状态和差异  
✅ 添加和提交更改  
✅ 推送到远程仓库  
✅ 拉取远程更新  
✅ 完整的工作流程

## 下一步

掌握了基本的Git命令后，下一节我们将学习如何处理各种代码修改的回退和撤销操作。

---

[← 上一节：Git客户端环境搭建](02-git-client-setup.md) | [返回目录](../../README.md) | [下一节：Git代码回退撤销操作 →](04-git-code-rollback-operations.md)
