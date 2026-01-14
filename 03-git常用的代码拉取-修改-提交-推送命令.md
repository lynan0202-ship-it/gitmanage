# 03. Git常用的代码拉取-修改-提交-推送命令

## 3.1 Git 基本工作流程

Git 的基本工作流程包括以下几个步骤：

```
1. 克隆/初始化仓库
2. 拉取最新代码
3. 修改代码
4. 查看状态
5. 添加到暂存区
6. 提交到本地仓库
7. 推送到远程仓库
```

## 3.2 初始化和克隆仓库

### 3.2.1 初始化本地仓库

```bash
# 在当前目录初始化 Git 仓库
git init

# 在指定目录初始化 Git 仓库
git init <project-name>
```

执行后会创建 `.git` 目录，这就是 Git 的版本库。

### 3.2.2 克隆远程仓库

```bash
# 克隆远程仓库（HTTPS）
git clone https://github.com/username/repository.git

# 克隆远程仓库（SSH）
git clone git@github.com:username/repository.git

# 克隆到指定目录
git clone https://github.com/username/repository.git my-project

# 克隆指定分支
git clone -b <branch-name> https://github.com/username/repository.git
```

**HTTPS vs SSH**：
- HTTPS：每次推送需要输入用户名密码（可以配置凭证缓存）
- SSH：配置密钥后无需输入密码（推荐）

## 3.3 拉取代码

### 3.3.1 git pull - 拉取并合并

```bash
# 拉取当前分支的最新代码
git pull

# 拉取指定远程仓库的指定分支
git pull origin main

# 拉取并使用 rebase 方式合并
git pull --rebase
```

**git pull 做了什么？**
```bash
git pull = git fetch + git merge
```

### 3.3.2 git fetch - 仅拉取不合并

```bash
# 拉取所有远程仓库的更新
git fetch

# 拉取指定远程仓库的更新
git fetch origin

# 拉取指定远程仓库的指定分支
git fetch origin main
```

**fetch vs pull**：
- `git fetch`：只下载远程的更新，不会自动合并
- `git pull`：下载并自动合并到当前分支

### 3.3.3 查看远程仓库

```bash
# 查看远程仓库
git remote -v

# 查看远程仓库详细信息
git remote show origin

# 添加远程仓库
git remote add origin https://github.com/username/repository.git

# 修改远程仓库地址
git remote set-url origin https://github.com/username/new-repository.git

# 删除远程仓库
git remote remove origin
```

## 3.4 查看状态和差异

### 3.4.1 git status - 查看状态

```bash
# 查看工作区状态
git status

# 查看简洁状态
git status -s
```

**状态说明**：
- `Untracked files`：未跟踪的文件（新文件）
- `Changes not staged for commit`：已修改但未暂存
- `Changes to be committed`：已暂存待提交

### 3.4.2 git diff - 查看差异

```bash
# 查看工作区与暂存区的差异
git diff

# 查看暂存区与最新提交的差异
git diff --staged
# 或
git diff --cached

# 查看工作区与最新提交的差异
git diff HEAD

# 查看指定文件的差异
git diff README.md

# 查看两个提交之间的差异
git diff commit1 commit2

# 查看两个分支之间的差异
git diff branch1 branch2
```

## 3.5 添加文件到暂存区

### 3.5.1 git add - 添加到暂存区

```bash
# 添加指定文件
git add <file>

# 添加指定目录
git add <directory>

# 添加所有修改和新文件
git add .

# 添加所有修改（包括删除）
git add -A
# 或
git add --all

# 添加所有已跟踪文件的修改
git add -u

# 交互式添加
git add -i

# 部分添加文件内容
git add -p <file>
```

**常用选项说明**：
- `git add .`：添加当前目录下所有修改
- `git add -A`：添加所有修改（包括删除的文件）
- `git add -u`：只添加已跟踪文件的修改
- `git add -p`：交互式选择要添加的内容

### 3.5.2 示例

```bash
# 场景1：添加单个文件
git add index.html

# 场景2：添加多个文件
git add index.html style.css script.js

# 场景3：添加整个目录
git add src/

# 场景4：添加所有修改
git add .

# 场景5：部分添加
git add -p main.js
# 然后根据提示选择 y(yes) 或 n(no)
```

## 3.6 提交代码

### 3.6.1 git commit - 提交到本地仓库

```bash
# 提交暂存区的修改
git commit -m "提交说明"

# 提交所有已跟踪文件的修改（跳过 git add）
git commit -a -m "提交说明"
# 或
git commit -am "提交说明"

# 打开编辑器编写详细的提交信息
git commit

# 修改最后一次提交
git commit --amend

# 修改最后一次提交的信息
git commit --amend -m "新的提交说明"

# 提交时显示差异信息
git commit -v
```

### 3.6.2 编写好的提交信息

**基本格式**：

```
<type>: <subject>

<body>

<footer>
```

**提交类型（type）**：
- `feat`：新功能
- `fix`：修复bug
- `docs`：文档修改
- `style`：代码格式修改（不影响代码运行）
- `refactor`：重构
- `test`：测试用例修改
- `chore`：构建过程或辅助工具的变动

**示例**：

```bash
# 简单提交
git commit -m "feat: 添加用户登录功能"

# 详细提交
git commit -m "fix: 修复用户登录时的空指针异常

- 添加了空值检查
- 优化了错误提示信息
- 更新了相关文档

Closes #123"
```

### 3.6.3 提交最佳实践

1. **提交信息清晰明确**：让别人（和未来的自己）能快速理解做了什么
2. **提交粒度适中**：一个提交只做一件事
3. **频繁提交**：完成一个小功能就提交
4. **避免提交不完整的代码**：确保提交的代码是可运行的

## 3.7 推送代码

### 3.7.1 git push - 推送到远程仓库

```bash
# 推送当前分支到远程仓库
git push

# 推送到指定远程仓库的指定分支
git push origin main

# 首次推送时建立跟踪关系
git push -u origin main
# 或
git push --set-upstream origin main

# 推送所有分支
git push --all

# 推送标签
git push --tags

# 强制推送（危险操作！）
git push -f
# 或
git push --force

# 更安全的强制推送
git push --force-with-lease
```

### 3.7.2 推送注意事项

**⚠️ 注意**：
1. 推送前先拉取：`git pull` 确保本地是最新的
2. 避免强制推送：`git push -f` 会覆盖远程历史，可能导致其他人的代码丢失
3. 首次推送使用 `-u` 参数：建立跟踪关系，以后可以直接 `git push`

### 3.7.3 推送失败的解决

```bash
# 推送失败提示：Updates were rejected
# 原因：远程仓库有新的提交

# 解决方案1：先拉取再推送
git pull
git push

# 解决方案2：使用 rebase
git pull --rebase
git push

# 解决方案3：如果确定要覆盖（慎用）
git push --force-with-lease
```

## 3.8 完整工作流程示例

### 3.8.1 场景1：新功能开发

```bash
# 1. 克隆仓库（首次）
git clone git@github.com:username/project.git
cd project

# 2. 拉取最新代码
git pull

# 3. 创建并切换到功能分支
git checkout -b feature/user-login

# 4. 修改代码
# ... 编辑文件 ...

# 5. 查看修改
git status
git diff

# 6. 添加到暂存区
git add .

# 7. 提交
git commit -m "feat: 实现用户登录功能"

# 8. 推送到远程
git push -u origin feature/user-login
```

### 3.8.2 场景2：修复bug

```bash
# 1. 拉取最新代码
git pull

# 2. 创建修复分支
git checkout -b fix/login-error

# 3. 修改代码
# ... 修复bug ...

# 4. 查看修改
git status
git diff

# 5. 添加并提交
git add .
git commit -m "fix: 修复登录时的空指针异常"

# 6. 推送
git push -u origin fix/login-error
```

### 3.8.3 场景3：日常开发

```bash
# 早上开始工作
git pull

# 修改文件
# ... 编辑 file1.js ...
git add file1.js
git commit -m "refactor: 重构登录逻辑"

# 继续修改
# ... 编辑 file2.js ...
git add file2.js
git commit -m "feat: 添加记住密码功能"

# 修改更多文件
# ... 编辑多个文件 ...
git add .
git commit -m "docs: 更新README文档"

# 一天结束，推送所有提交
git push
```

## 3.9 撤销和回退操作

### 3.9.1 撤销工作区的修改

```bash
# 撤销单个文件的修改
git checkout -- <file>

# 撤销所有文件的修改
git checkout -- .

# Git 2.23+ 新命令
git restore <file>
git restore .
```

### 3.9.2 撤销暂存区的文件

```bash
# 从暂存区移除文件（但保留工作区的修改）
git reset HEAD <file>

# Git 2.23+ 新命令
git restore --staged <file>
```

### 3.9.3 撤销提交

```bash
# 撤销最后一次提交，保留修改在工作区
git reset HEAD~1

# 撤销最后一次提交，保留修改在暂存区
git reset --soft HEAD~1

# 撤销最后一次提交，丢弃所有修改
git reset --hard HEAD~1
```

## 3.10 查看提交历史

```bash
# 查看提交历史
git log

# 查看简洁的提交历史
git log --oneline

# 查看最近 n 条提交
git log -n 5

# 查看图形化的提交历史
git log --graph --oneline --all

# 查看指定文件的提交历史
git log README.md

# 查看指定作者的提交
git log --author="张三"

# 查看指定日期范围的提交
git log --since="2024-01-01" --until="2024-12-31"

# 查看提交的详细信息
git show <commit-id>
```

## 3.11 常用命令速查表

| 命令 | 说明 |
|------|------|
| `git clone <url>` | 克隆远程仓库 |
| `git pull` | 拉取并合并远程代码 |
| `git fetch` | 拉取远程代码（不合并） |
| `git status` | 查看工作区状态 |
| `git diff` | 查看工作区修改 |
| `git add <file>` | 添加文件到暂存区 |
| `git add .` | 添加所有修改到暂存区 |
| `git commit -m "msg"` | 提交到本地仓库 |
| `git push` | 推送到远程仓库 |
| `git log` | 查看提交历史 |

## 3.12 总结

- **克隆仓库**：`git clone` 从远程获取代码
- **拉取代码**：`git pull` 获取最新更新
- **查看状态**：`git status` 和 `git diff`
- **添加文件**：`git add` 将修改加入暂存区
- **提交代码**：`git commit` 提交到本地仓库
- **推送代码**：`git push` 推送到远程仓库
- **完整流程**：pull → edit → add → commit → push

---

**上一章节**：[02. Git客户端环境搭建](./02-git客户端环境搭建.md)

**下一章节**：[04. Git各阶段代码修改回退撤销操作](./04-git各阶段代码修改回退撤销操作.md)
