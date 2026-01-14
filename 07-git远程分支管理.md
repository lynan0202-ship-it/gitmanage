# 07. Git远程分支管理

## 7.1 远程仓库概述

### 7.1.1 什么是远程仓库

远程仓库（Remote Repository）是托管在网络上的项目版本库，可以是 GitHub、GitLab、Gitee 等平台，也可以是自己搭建的 Git 服务器。

### 7.1.2 为什么需要远程仓库

- **代码备份**：防止本地代码丢失
- **团队协作**：多人共享代码
- **代码审查**：通过 Pull Request 进行代码审查
- **持续集成**：触发自动化构建和测试
- **版本发布**：管理和发布项目版本

### 7.1.3 常见的远程仓库平台

- **GitHub**：https://github.com （全球最大的代码托管平台）
- **GitLab**：https://gitlab.com （开源，可自托管）
- **Gitee**：https://gitee.com （国内平台，访问快）
- **Bitbucket**：https://bitbucket.org
- **Azure DevOps**：https://dev.azure.com

## 7.2 查看远程仓库

### 7.2.1 查看远程仓库列表

```bash
# 查看远程仓库
git remote

# 查看远程仓库及其 URL
git remote -v

# 输出示例：
# origin  git@github.com:username/repo.git (fetch)
# origin  git@github.com:username/repo.git (push)
```

**说明**：
- `origin`：默认的远程仓库名称
- `fetch`：拉取地址
- `push`：推送地址

### 7.2.2 查看远程仓库详细信息

```bash
# 查看指定远程仓库的详细信息
git remote show origin

# 输出包括：
# - 远程仓库 URL
# - 跟踪的分支
# - 本地分支与远程分支的关系
# - push 和 pull 的配置
```

## 7.3 添加远程仓库

### 7.3.1 添加远程仓库

```bash
# 添加远程仓库
git remote add <remote-name> <url>

# 示例：添加 origin
git remote add origin git@github.com:username/repo.git

# 示例：添加第二个远程仓库
git remote add upstream git@github.com:original/repo.git
```

### 7.3.2 常见的远程仓库名称

- `origin`：你自己的远程仓库（fork 的仓库）
- `upstream`：原始仓库（被 fork 的仓库）
- `github`、`gitlab`、`gitee`：不同平台的仓库

### 7.3.3 克隆时自动添加

```bash
# 克隆仓库会自动添加 origin
git clone git@github.com:username/repo.git

# 等同于：
git init
git remote add origin git@github.com:username/repo.git
git fetch origin
git checkout main
```

## 7.4 修改远程仓库

### 7.4.1 修改远程仓库 URL

```bash
# 修改远程仓库地址
git remote set-url <remote-name> <new-url>

# 示例：从 HTTPS 改为 SSH
git remote set-url origin git@github.com:username/repo.git

# 示例：更改仓库地址
git remote set-url origin git@github.com:username/new-repo.git
```

### 7.4.2 重命名远程仓库

```bash
# 重命名远程仓库
git remote rename <old-name> <new-name>

# 示例
git remote rename origin github
```

### 7.4.3 删除远程仓库

```bash
# 删除远程仓库引用
git remote remove <remote-name>
# 或
git remote rm <remote-name>

# 示例
git remote remove upstream
```

## 7.5 查看远程分支

### 7.5.1 查看远程分支列表

```bash
# 查看远程分支
git branch -r

# 输出示例：
# origin/main
# origin/develop
# origin/feature/login

# 查看所有分支（本地+远程）
git branch -a

# 输出示例：
# * main
#   develop
#   remotes/origin/main
#   remotes/origin/develop
```

### 7.5.2 查看远程分支详情

```bash
# 查看远程分支及最后一次提交
git branch -r -v

# 查看远程分支的跟踪关系
git branch -vv
```

## 7.6 拉取远程分支

### 7.6.1 git fetch - 获取远程更新

```bash
# 获取所有远程仓库的更新
git fetch

# 获取指定远程仓库的更新
git fetch origin

# 获取指定远程仓库的指定分支
git fetch origin main

# 获取所有远程仓库的所有分支
git fetch --all
```

**注意**：`fetch` 只下载数据，不会自动合并。

### 7.6.2 git pull - 拉取并合并

```bash
# 拉取当前分支对应的远程分支
git pull

# 拉取指定远程仓库的指定分支
git pull origin main

# 拉取并使用 rebase
git pull --rebase

# 拉取所有远程分支
git pull --all
```

**注意**：`git pull = git fetch + git merge`

### 7.6.3 fetch vs pull

| 操作 | git fetch | git pull |
|------|-----------|----------|
| 下载数据 | ✅ | ✅ |
| 自动合并 | ❌ | ✅ |
| 安全性 | 高（可以先查看再决定） | 低（直接合并） |
| 使用场景 | 查看更新，手动合并 | 快速同步 |

**推荐流程**：
```bash
# 1. 先 fetch 查看更新
git fetch origin

# 2. 查看差异
git diff main origin/main

# 3. 决定是否合并
git merge origin/main
```

## 7.7 推送到远程分支

### 7.7.1 基本推送

```bash
# 推送当前分支到远程
git push

# 推送指定分支到远程
git push origin main

# 首次推送并建立跟踪关系
git push -u origin main
# 或
git push --set-upstream origin main
```

### 7.7.2 推送所有分支

```bash
# 推送所有本地分支
git push --all origin

# 推送所有标签
git push --tags origin

# 推送分支和标签
git push --all origin && git push --tags origin
```

### 7.7.3 推送到不同名称的远程分支

```bash
# 将本地分支推送到不同名称的远程分支
git push origin <local-branch>:<remote-branch>

# 示例：将本地 dev 分支推送到远程 develop 分支
git push origin dev:develop
```

### 7.7.4 强制推送

```bash
# 强制推送（危险！）
git push -f origin main
# 或
git push --force origin main

# 更安全的强制推送
git push --force-with-lease origin main
```

**⚠️ 警告**：
- 强制推送会覆盖远程历史
- 可能导致其他人的代码丢失
- 团队协作时避免使用
- 如果必须使用，用 `--force-with-lease` 更安全

### 7.7.5 推送失败的处理

```bash
# 推送失败：远程有新提交
# 错误信息：
# ! [rejected]        main -> main (fetch first)

# 解决方案1：先拉取再推送
git pull origin main
git push origin main

# 解决方案2：使用 rebase
git pull --rebase origin main
git push origin main

# 解决方案3：强制推送（慎用）
git push --force-with-lease origin main
```

## 7.8 删除远程分支

### 7.8.1 删除远程分支

```bash
# 删除远程分支（新语法）
git push origin --delete <branch-name>

# 删除远程分支（旧语法）
git push origin :<branch-name>

# 示例
git push origin --delete feature/old-feature
```

### 7.8.2 删除远程标签

```bash
# 删除远程标签
git push origin --delete tag <tag-name>

# 或
git push origin :refs/tags/<tag-name>

# 示例
git push origin --delete tag v1.0.0
```

### 7.8.3 清理本地的远程分支引用

```bash
# 清理已删除的远程分支的本地引用
git remote prune origin

# 或在 fetch 时自动清理
git fetch --prune origin

# 查看将被清理的分支
git remote prune origin --dry-run
```

## 7.9 跟踪远程分支

### 7.9.1 什么是跟踪分支

跟踪分支（Tracking Branch）是与远程分支有直接关系的本地分支。在跟踪分支上执行 `git pull` 和 `git push` 时，Git 知道要操作哪个远程分支。

### 7.9.2 创建跟踪分支

```bash
# 方法1：克隆时自动创建
git clone <url>

# 方法2：从远程分支创建跟踪分支
git checkout -b <local-branch> origin/<remote-branch>

# 方法3：自动创建同名跟踪分支（Git 会自动跟踪）
git checkout <remote-branch>

# 方法4：使用 --track 参数
git checkout --track origin/<remote-branch>
```

### 7.9.3 设置跟踪关系

```bash
# 为当前分支设置跟踪的远程分支
git branch -u origin/<remote-branch>
# 或
git branch --set-upstream-to=origin/<remote-branch>

# 为指定分支设置跟踪关系
git branch -u origin/<remote-branch> <local-branch>

# 示例
git branch -u origin/main main
```

### 7.9.4 查看跟踪关系

```bash
# 查看本地分支及其跟踪的远程分支
git branch -vv

# 输出示例：
# * main    abc1234 [origin/main] Latest commit
#   develop def5678 [origin/develop: ahead 2] Another commit
```

**说明**：
- `[origin/main]`：跟踪 origin/main
- `[origin/develop: ahead 2]`：本地比远程领先 2 个提交
- `[origin/feature: behind 1]`：本地比远程落后 1 个提交

### 7.9.5 取消跟踪关系

```bash
# 取消当前分支的跟踪关系
git branch --unset-upstream

# 取消指定分支的跟踪关系
git branch --unset-upstream <branch-name>
```

## 7.10 同步 Fork 的仓库

### 7.10.1 Fork 工作流

在开源项目中，常见的流程是：

```
原始仓库（upstream）
        ↓ fork
你的仓库（origin）
        ↓ clone
本地仓库（local）
```

### 7.10.2 配置上游仓库

```bash
# 1. 克隆你 fork 的仓库
git clone git@github.com:your-username/repo.git
cd repo

# 2. 添加上游仓库
git remote add upstream git@github.com:original-owner/repo.git

# 3. 查看远程仓库
git remote -v
# origin    git@github.com:your-username/repo.git (fetch)
# origin    git@github.com:your-username/repo.git (push)
# upstream  git@github.com:original-owner/repo.git (fetch)
# upstream  git@github.com:original-owner/repo.git (push)
```

### 7.10.3 同步上游仓库

```bash
# 1. 获取上游仓库的更新
git fetch upstream

# 2. 切换到主分支
git checkout main

# 3. 合并上游的更新
git merge upstream/main

# 或使用 rebase
git rebase upstream/main

# 4. 推送到你的远程仓库
git push origin main
```

### 7.10.4 完整的贡献流程

```bash
# 1. 同步上游仓库
git fetch upstream
git checkout main
git merge upstream/main

# 2. 创建功能分支
git checkout -b feature/new-feature

# 3. 开发并提交
git add .
git commit -m "feat: 添加新功能"

# 4. 推送到你的远程仓库
git push origin feature/new-feature

# 5. 在 GitHub 上创建 Pull Request

# 6. PR 合并后，同步并删除分支
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

## 7.11 远程分支的高级操作

### 7.11.1 比较本地和远程分支

```bash
# 比较本地分支和远程分支的差异
git diff main origin/main

# 查看本地分支比远程分支多了哪些提交
git log origin/main..main

# 查看远程分支比本地分支多了哪些提交
git log main..origin/main

# 查看双向差异
git log --left-right main...origin/main
```

### 7.11.2 检出远程分支

```bash
# 检出远程分支的某个文件
git checkout origin/main -- path/to/file

# 临时切换到远程分支（detached HEAD）
git checkout origin/main

# 从远程分支创建本地分支
git checkout -b local-branch origin/remote-branch
```

### 7.11.3 基于远程分支创建新分支

```bash
# 基于远程分支创建新分支
git checkout -b new-branch origin/existing-branch

# 示例：基于远程的 develop 创建功能分支
git checkout -b feature/new-feature origin/develop
```

## 7.12 多个远程仓库管理

### 7.12.1 同时维护多个远程仓库

```bash
# 添加多个远程仓库
git remote add github git@github.com:username/repo.git
git remote add gitlab git@gitlab.com:username/repo.git
git remote add gitee git@gitee.com:username/repo.git

# 查看所有远程仓库
git remote -v

# 推送到多个远程仓库
git push github main
git push gitlab main
git push gitee main
```

### 7.12.2 配置推送到多个远程仓库

```bash
# 方法1：添加多个 push URL
git remote set-url --add --push origin git@github.com:username/repo.git
git remote set-url --add --push origin git@gitlab.com:username/repo.git

# 一次 push 推送到所有配置的 URL
git push origin main

# 方法2：使用脚本
#!/bin/bash
git push github main
git push gitlab main
git push gitee main
```

## 7.13 常见问题

### 7.13.1 推送被拒绝

```bash
# 问题：
! [rejected]        main -> main (fetch first)

# 原因：远程有新提交，本地落后于远程

# 解决：
git pull origin main
git push origin main
```

### 7.13.2 本地分支与远程分支不同步

```bash
# 查看状态
git status

# 输出可能显示：
# Your branch is ahead of 'origin/main' by 2 commits.
# Your branch is behind 'origin/main' by 1 commit.

# 领先：推送本地提交
git push

# 落后：拉取远程提交
git pull

# 分叉：先拉取再推送
git pull
git push
```

### 7.13.3 远程分支已删除但本地仍显示

```bash
# 清理已删除的远程分支引用
git remote prune origin

# 或
git fetch --prune
```

### 7.13.4 切换远程仓库地址

```bash
# 从 HTTPS 切换到 SSH
git remote set-url origin git@github.com:username/repo.git

# 从 SSH 切换到 HTTPS
git remote set-url origin https://github.com/username/repo.git
```

## 7.14 远程分支管理速查表

| 操作 | 命令 |
|------|------|
| 查看远程仓库 | `git remote -v` |
| 添加远程仓库 | `git remote add <name> <url>` |
| 修改远程地址 | `git remote set-url <name> <url>` |
| 删除远程仓库 | `git remote remove <name>` |
| 查看远程分支 | `git branch -r` |
| 获取远程更新 | `git fetch` |
| 拉取并合并 | `git pull` |
| 推送到远程 | `git push` |
| 推送并建立跟踪 | `git push -u origin <branch>` |
| 删除远程分支 | `git push origin --delete <branch>` |
| 设置跟踪分支 | `git branch -u origin/<branch>` |
| 清理远程分支引用 | `git remote prune origin` |

## 7.15 总结

- **远程仓库**：托管在网络上的代码仓库，支持团队协作
- **origin**：默认的远程仓库名称
- **fetch vs pull**：fetch 只下载，pull 下载并合并
- **跟踪分支**：与远程分支关联的本地分支
- **Fork 工作流**：通过 upstream 同步原始仓库
- **多远程仓库**：可以同时配置多个远程仓库
- **定期同步**：频繁 fetch/pull 保持代码最新

---

**上一章节**：[06. Git本地分支管理](./06-git本地分支管理.md)

**下一章节**：[08. Git工作流实践01](./08-git工作流实践01.md)
