# 07. Git远程分支管理

## 目录
- [远程仓库概述](#远程仓库概述)
- [远程仓库配置](#远程仓库配置)
- [远程分支操作](#远程分支操作)
- [跟踪分支](#跟踪分支)
- [远程协作工作流](#远程协作工作流)
- [Pull Request工作流](#pull-request工作流)
- [常见问题和解决方案](#常见问题和解决方案)
- [最佳实践](#最佳实践)

## 远程仓库概述

### 什么是远程仓库

远程仓库是托管在网络上的项目版本库，用于团队成员之间的代码共享和协作。

```
本地仓库               远程仓库
┌─────────┐          ┌─────────┐
│  main   │ ←push→   │  main   │
│ develop │ ←fetch→  │ develop │
│ feature │ ←pull→   │ feature │
└─────────┘          └─────────┘
```

### 常见远程仓库服务

- **GitHub**：最流行的代码托管平台
- **GitLab**：开源，可自建
- **Bitbucket**：Atlassian产品
- **Gitee**：国内代码托管平台
- **Azure DevOps**：微软的DevOps平台

## 远程仓库配置

### 查看远程仓库

```bash
# 查看远程仓库列表
git remote

# 查看远程仓库URL
git remote -v

# 查看远程仓库详细信息
git remote show origin
```

**示例输出**：

```bash
$ git remote -v
origin  https://github.com/username/repo.git (fetch)
origin  https://github.com/username/repo.git (push)
```

### 添加远程仓库

```bash
# 添加远程仓库
git remote add <name> <url>

# 通常命名为origin
git remote add origin https://github.com/username/repo.git

# 添加多个远程仓库
git remote add upstream https://github.com/original/repo.git
```

**示例**：

```bash
# 克隆后自动有origin
git clone https://github.com/username/repo.git

# 添加上游仓库
git remote add upstream https://github.com/original/repo.git

# 查看
git remote -v
# origin    https://github.com/username/repo.git (fetch)
# origin    https://github.com/username/repo.git (push)
# upstream  https://github.com/original/repo.git (fetch)
# upstream  https://github.com/original/repo.git (push)
```

### 修改远程仓库

```bash
# 修改远程仓库URL
git remote set-url <name> <new-url>

# 示例：从HTTPS切换到SSH
git remote set-url origin git@github.com:username/repo.git

# 重命名远程仓库
git remote rename <old-name> <new-name>

# 删除远程仓库
git remote remove <name>
# 或
git remote rm <name>
```

**示例**：

```bash
# 将origin从HTTPS改为SSH
git remote set-url origin git@github.com:username/repo.git

# 重命名
git remote rename origin github

# 删除
git remote remove old-remote
```

## 远程分支操作

### 查看远程分支

```bash
# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 查看远程分支详细信息
git remote show origin

# 查看远程分支和本地分支的跟踪关系
git branch -vv
```

**示例输出**：

```bash
$ git branch -a
* main
  develop
  feature/login
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
  remotes/origin/develop
  remotes/origin/feature/payment
```

### 获取远程分支

```bash
# 获取所有远程分支更新（不合并）
git fetch

# 获取特定远程仓库的更新
git fetch origin

# 获取特定分支的更新
git fetch origin main

# 获取所有远程仓库的更新
git fetch --all

# 获取并清理已删除的远程分支引用
git fetch --prune
# 或简写
git fetch -p
```

**fetch的作用**：

```bash
# 前：本地没有远程的新提交
本地 main:     A -- B
远程 main:     A -- B -- C -- D

# 执行 git fetch origin
本地 main:            A -- B
远程 origin/main:     A -- B -- C -- D

# 本地分支不变，但知道了远程的更新
```

### 拉取远程分支

```bash
# 拉取并合并当前分支
git pull

# 拉取特定远程分支
git pull origin main

# 使用rebase方式拉取
git pull --rebase origin main

# 拉取所有远程分支
git pull --all

# git pull = git fetch + git merge
# git pull --rebase = git fetch + git rebase
```

**示例**：

```bash
# 方式1：pull（自动合并）
git pull origin main

# 方式2：fetch + merge（手动控制）
git fetch origin
git diff main origin/main  # 查看差异
git merge origin/main

# 方式3：fetch + rebase（线性历史）
git fetch origin
git rebase origin/main
```

### 推送到远程分支

```bash
# 推送当前分支到远程
git push

# 推送指定分支
git push origin main

# 首次推送并设置上游分支
git push -u origin main
# 或
git push --set-upstream origin main

# 推送所有本地分支
git push --all origin

# 推送标签
git push --tags

# 强制推送（危险！）
git push -f origin main
# 或
git push --force origin main

# 更安全的强制推送
git push --force-with-lease origin main
```

**示例**：

```bash
# 首次推送新分支
git checkout -b feature/new-feature
git push -u origin feature/new-feature
# 之后只需要
git push

# 推送到不同名称的远程分支
git push origin local-branch:remote-branch
```

### 删除远程分支

```bash
# 删除远程分支
git push origin --delete <branch-name>

# 或使用旧语法
git push origin :<branch-name>

# 删除多个远程分支
git push origin --delete branch1 branch2
```

**示例**：

```bash
# 删除远程的feature分支
git push origin --delete feature/old-feature

# 删除本地对应的跟踪分支
git branch -d feature/old-feature
```

### 创建远程分支

```bash
# 推送本地分支到远程（如果不存在则创建）
git push origin <local-branch>

# 推送并重命名
git push origin local-branch:remote-branch
```

## 跟踪分支

### 什么是跟踪分支

跟踪分支（Tracking Branch）是与远程分支有直接关系的本地分支。

```
本地分支（跟踪） → 远程分支
main             → origin/main
develop          → origin/develop
```

### 设置跟踪分支

```bash
# 创建并设置跟踪分支
git checkout -b <branch> origin/<branch>

# 或使用更简洁的方式（Git会自动匹配）
git checkout <branch>

# 为现有分支设置上游
git branch --set-upstream-to=origin/<branch> <local-branch>
# 或简写
git branch -u origin/<branch>

# 推送时设置上游
git push -u origin <branch>
```

**示例**：

```bash
# 方式1：从远程分支创建跟踪分支
git checkout -b feature origin/feature

# 方式2：简化方式（如果本地没有feature分支）
git checkout feature  # Git自动创建并跟踪origin/feature

# 方式3：为现有分支设置跟踪
git checkout feature
git branch -u origin/feature

# 方式4：推送时设置
git push -u origin feature
```

### 查看跟踪关系

```bash
# 查看分支跟踪关系
git branch -vv

# 输出示例：
# * main    abc1234 [origin/main] Latest commit
#   develop def5678 [origin/develop: ahead 2, behind 1] WIP
#   feature ghi9012 No upstream configured
```

**状态说明**：
- `ahead 2`：本地领先远程2个提交
- `behind 1`：本地落后远程1个提交
- `No upstream`：未设置跟踪关系

### 取消跟踪关系

```bash
# 取消当前分支的上游跟踪
git branch --unset-upstream

# 取消指定分支的上游跟踪
git branch --unset-upstream <branch-name>
```

## 远程协作工作流

### Fork + Pull Request工作流

适用于开源项目和大型团队：

```bash
# 1. Fork项目（在GitHub上操作）

# 2. 克隆你的fork
git clone https://github.com/your-username/repo.git
cd repo

# 3. 添加上游仓库
git remote add upstream https://github.com/original/repo.git

# 4. 创建功能分支
git checkout -b feature/awesome-feature

# 5. 开发功能
# 编写代码...
git add .
git commit -m "feat: add awesome feature"

# 6. 推送到你的fork
git push origin feature/awesome-feature

# 7. 在GitHub上创建Pull Request

# 8. 保持同步
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### 共享仓库工作流

适用于小型团队，所有成员都有推送权限：

```bash
# 1. 克隆共享仓库
git clone https://github.com/team/repo.git

# 2. 创建功能分支
git checkout -b feature/login

# 3. 开发并推送
git add .
git commit -m "feat: implement login"
git push -u origin feature/login

# 4. 在平台上创建Pull Request

# 5. 代码审查后合并

# 6. 更新本地main分支
git checkout main
git pull origin main

# 7. 删除功能分支
git branch -d feature/login
git push origin --delete feature/login
```

### 集中式工作流

所有人在同一个分支上工作：

```bash
# 1. 克隆仓库
git clone <repo-url>

# 2. 拉取最新代码
git pull origin main

# 3. 修改代码
# 编写代码...
git add .
git commit -m "feat: new feature"

# 4. 推送前再次拉取（避免冲突）
git pull origin main

# 5. 解决冲突（如果有）

# 6. 推送
git push origin main
```

## Pull Request工作流

### 创建Pull Request

```bash
# 1. 创建并推送分支
git checkout -b feature/user-profile
# 开发...
git push -u origin feature/user-profile

# 2. 在GitHub/GitLab上创建PR
#    - 选择源分支：feature/user-profile
#    - 选择目标分支：main
#    - 填写PR标题和描述
#    - 指定审查者

# 3. 等待代码审查
```

### 更新Pull Request

```bash
# 根据审查意见修改代码
# 编辑文件...
git add .
git commit -m "refactor: address review comments"

# 推送更新
git push origin feature/user-profile

# PR会自动更新
```

### 同步主分支更新

```bash
# 在PR期间，main分支可能有新提交
git checkout main
git pull origin main

git checkout feature/user-profile

# 方式1：合并
git merge main

# 方式2：变基（保持线性历史）
git rebase main

# 推送更新（如果用了rebase需要强制推送）
git push origin feature/user-profile
# 或
git push --force-with-lease origin feature/user-profile
```

### 合并Pull Request

在Web界面上：
1. **Merge Commit**：创建合并提交
2. **Squash and Merge**：压缩所有提交为一个
3. **Rebase and Merge**：变基合并

命令行方式：

```bash
# 切换到目标分支
git checkout main

# 拉取最新代码
git pull origin main

# 方式1：普通合并
git merge --no-ff feature/user-profile

# 方式2：压缩合并
git merge --squash feature/user-profile
git commit -m "feat: add user profile feature"

# 方式3：变基合并
git rebase feature/user-profile

# 推送
git push origin main

# 删除功能分支
git branch -d feature/user-profile
git push origin --delete feature/user-profile
```

## 常见问题和解决方案

### 问题1：推送被拒绝

```bash
$ git push origin main

To https://github.com/username/repo.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs
```

**解决方案**：

```bash
# 先拉取远程更新
git pull origin main

# 解决冲突（如果有）

# 再推送
git push origin main

# 或使用rebase方式
git pull --rebase origin main
git push origin main
```

### 问题2：本地分支落后远程

```bash
# 查看状态
git branch -vv
# main [origin/main: behind 3]

# 更新本地分支
git pull origin main
```

### 问题3：本地分支领先远程

```bash
# 查看状态
git branch -vv
# main [origin/main: ahead 2]

# 推送本地提交
git push origin main
```

### 问题4：远程分支已删除但本地仍有引用

```bash
# 清理已删除的远程分支引用
git fetch --prune

# 或配置自动清理
git config --global fetch.prune true
```

### 问题5：忘记设置上游分支

```bash
$ git push

fatal: The current branch feature has no upstream branch.

# 解决：设置上游
git push -u origin feature

# 或
git branch --set-upstream-to=origin/feature
git push
```

### 问题6：需要强制推送

```bash
# 本地rebase后历史不一致

# ⚠️ 危险：会覆盖远程
git push --force origin feature

# ✅ 更安全：如果远程有其他人的提交会失败
git push --force-with-lease origin feature
```

### 问题7：克隆时连接超时

```bash
# 问题：网络不稳定或仓库太大

# 解决1：浅克隆
git clone --depth 1 <repo-url>

# 解决2：部分克隆
git clone --filter=blob:none <repo-url>

# 解决3：使用镜像
# 中国用户可使用Gitee导入GitHub仓库
```

## 最佳实践

### 1. 合理使用fetch和pull

```bash
# ✅ 查看远程更新但不合并
git fetch origin
git diff main origin/main
git merge origin/main

# ❌ 盲目pull可能导致意外合并
git pull  # 不知道会拉取什么
```

### 2. 保护重要分支

```
在GitHub/GitLab上设置分支保护规则：
- 要求Pull Request审查
- 要求CI测试通过
- 禁止强制推送
- 禁止删除
- 要求最新代码才能合并
```

### 3. 使用有意义的远程名称

```bash
# ✅ 清晰的命名
git remote add upstream https://github.com/original/repo.git
git remote add github https://github.com/username/repo.git
git remote add gitlab https://gitlab.com/username/repo.git

# ❌ 模糊的命名
git remote add remote1 <url>
git remote add temp <url>
```

### 4. 定期同步

```bash
# 每天开始工作前
git fetch --all --prune
git checkout main
git pull origin main

# 在功能分支上
git checkout feature
git merge main  # 或 git rebase main
```

### 5. 清理本地分支

```bash
# 查看已合并的分支
git branch --merged

# 删除已合并的本地分支
git branch --merged | grep -v "\*" | grep -v "main" | grep -v "develop" | xargs -n 1 git branch -d

# 清理远程跟踪分支
git fetch --prune
```

### 6. 使用别名简化操作

```bash
# 配置别名
git config --global alias.sync '!git fetch --all --prune && git pull'
git config --global alias.pf 'push --force-with-lease'
git config --global alias.cleanup "!git branch --merged | grep -v '\\*' | xargs -n 1 git branch -d"

# 使用
git sync
git pf
git cleanup
```

### 7. 提交前检查

```bash
# 推送前检查
git status              # 检查状态
git log origin/main..HEAD  # 查看即将推送的提交
git diff origin/main    # 查看差异
git push origin main    # 推送
```

## 远程协作检查清单

```
□ 拉取最新代码
□ 创建功能分支
□ 频繁提交
□ 定期推送到远程
□ 保持与主分支同步
□ 编写清晰的提交信息
□ 创建Pull Request
□ 响应代码审查意见
□ 测试通过后合并
□ 删除已合并的分支
□ 更新本地仓库
```

## 总结

本节学习了Git远程分支管理：

✅ 理解远程仓库的概念  
✅ 掌握远程仓库的配置方法  
✅ 学会远程分支的各种操作  
✅ 理解和使用跟踪分支  
✅ 掌握不同的协作工作流  
✅ 熟悉Pull Request流程  
✅ 解决常见远程协作问题  
✅ 遵循最佳实践

**关键点**：
- fetch只下载不合并，pull会自动合并
- 使用--force-with-lease代替--force
- 保持与上游仓库同步
- 遵循团队的工作流程

## 下一步

掌握了远程分支管理后，下一节我们将学习Git工作流的实践方法。

---

[← 上一节：Git本地分支管理](06-git-local-branch-management.md) | [返回目录](../../README.md) | [下一节：Git工作流实践01 →](08-git-workflow-practice-01.md)
