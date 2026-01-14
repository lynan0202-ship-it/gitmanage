# 06. Git本地分支管理

## 6.1 什么是分支

### 6.1.1 分支的概念

分支（Branch）是 Git 中最强大的功能之一。它允许你从主线开发中分离出来，进行独立的开发工作，而不影响主线。

```
main:    A---B---C---F---G
              \       /
feature:       D-----E
```

### 6.1.2 为什么需要分支

- **并行开发**：多个功能同时开发互不干扰
- **功能隔离**：新功能在独立分支上开发，不影响稳定版本
- **实验性尝试**：可以随时创建分支尝试新想法
- **Bug 修复**：在独立分支上修复 bug，测试通过后再合并
- **版本管理**：不同版本可以使用不同的分支

### 6.1.3 Git 分支的特点

- **轻量级**：创建分支几乎是瞬间完成
- **快速切换**：分支切换非常快速
- **本质是指针**：Git 的分支只是指向提交对象的可变指针

## 6.2 查看分支

### 6.2.1 查看本地分支

```bash
# 查看所有本地分支
git branch

# 查看所有本地分支及最后一次提交
git branch -v

# 查看所有本地分支及其跟踪的远程分支
git branch -vv
```

**输出示例**：
```
* main
  feature/login
  hotfix/bug-123
```

`*` 表示当前所在的分支。

### 6.2.2 查看远程分支

```bash
# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a
```

### 6.2.3 查看分支详细信息

```bash
# 查看分支及其最后一次提交
git branch -v

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged
```

## 6.3 创建分支

### 6.3.1 创建新分支

```bash
# 创建新分支（但不切换）
git branch <branch-name>

# 示例
git branch feature/user-login
```

**注意**：这个命令只创建分支，不会切换到新分支。

### 6.3.2 从指定提交创建分支

```bash
# 从指定提交创建分支
git branch <branch-name> <commit-id>

# 从指定分支创建新分支
git branch <new-branch> <existing-branch>

# 示例
git branch hotfix/bug-fix abc1234
git branch develop main
```

### 6.3.3 创建并切换到新分支

```bash
# 创建并切换（旧方式）
git checkout -b <branch-name>

# 创建并切换（新方式，Git 2.23+）
git switch -c <branch-name>

# 示例
git checkout -b feature/user-login
# 或
git switch -c feature/user-login
```

### 6.3.4 从远程分支创建本地分支

```bash
# 创建跟踪远程分支的本地分支
git checkout -b <local-branch> origin/<remote-branch>

# 简化写法（自动创建同名分支）
git checkout <remote-branch>

# Git 2.23+ 新命令
git switch <remote-branch>

# 示例
git checkout -b feature origin/feature
```

## 6.4 切换分支

### 6.4.1 切换到已存在的分支

```bash
# 切换分支（旧方式）
git checkout <branch-name>

# 切换分支（新方式，Git 2.23+，推荐）
git switch <branch-name>

# 示例
git checkout main
# 或
git switch main
```

### 6.4.2 切换分支的注意事项

**切换前的要求**：
- 工作区没有未提交的修改（或已暂存/已 stash）
- 没有未解决的冲突

**如果有未提交的修改**：

```bash
# 方案1：提交修改
git add .
git commit -m "保存当前工作"
git switch other-branch

# 方案2：暂存修改
git stash
git switch other-branch
# 之后恢复
git stash pop

# 方案3：强制切换（会丢失修改！）
git checkout -f other-branch
```

### 6.4.3 切换到上一个分支

```bash
# 切换到上一个分支
git checkout -
# 或
git switch -
```

### 6.4.4 detached HEAD 状态

```bash
# 切换到特定提交（detached HEAD 状态）
git checkout <commit-id>

# 这时可以查看代码，但不建议直接修改
# 如果要基于这个提交开发，创建新分支
git checkout -b new-branch
```

## 6.5 重命名分支

### 6.5.1 重命名当前分支

```bash
# 重命名当前分支
git branch -m <new-name>

# 示例
git branch -m feature/login
```

### 6.5.2 重命名其他分支

```bash
# 重命名指定分支
git branch -m <old-name> <new-name>

# 示例
git branch -m old-feature new-feature
```

### 6.5.3 强制重命名（覆盖已存在的分支名）

```bash
# 强制重命名
git branch -M <new-name>

# 示例
git branch -M main
```

## 6.6 删除分支

### 6.6.1 删除已合并的分支

```bash
# 删除分支（安全删除，只能删除已合并的分支）
git branch -d <branch-name>

# 示例
git branch -d feature/login
```

如果分支未合并，会提示错误：
```
error: The branch 'feature/login' is not fully merged.
```

### 6.6.2 强制删除分支

```bash
# 强制删除分支（即使未合并）
git branch -D <branch-name>

# 示例
git branch -D feature/abandoned
```

**⚠️ 警告**：强制删除会丢失未合并的修改！

### 6.6.3 删除多个分支

```bash
# 删除多个分支
git branch -d branch1 branch2 branch3

# 使用通配符删除（需要 shell 支持）
git branch | grep 'feature/' | xargs git branch -d
```

### 6.6.4 清理已合并的分支

```bash
# 查看已合并的分支
git branch --merged

# 删除所有已合并的分支（保留 main 和 develop）
git branch --merged | grep -v "\*" | grep -v "main" | grep -v "develop" | xargs git branch -d
```

## 6.7 合并分支

### 6.7.1 快进合并（Fast-forward）

当目标分支是源分支的直接后继时，Git 会进行快进合并：

```bash
# 场景：main 没有新提交
main:    A---B---C
              \
feature:       D---E

# 合并
git checkout main
git merge feature

# 结果：
main:    A---B---C---D---E
```

命令：

```bash
git checkout main
git merge feature
```

### 6.7.2 三方合并（3-way merge）

当两个分支都有新提交时，Git 会进行三方合并并创建新的合并提交：

```bash
# 场景：main 有新提交
main:    A---B---C---F
              \
feature:       D---E

# 合并
git checkout main
git merge feature

# 结果：
main:    A---B---C---F---M
              \         /
feature:       D-------E
```

命令：

```bash
git checkout main
git merge feature
```

### 6.7.3 禁止快进合并

```bash
# 即使可以快进，也创建合并提交
git merge --no-ff <branch-name>

# 示例
git checkout main
git merge --no-ff feature/login -m "Merge feature/login"
```

**使用场景**：想保留分支的存在历史记录。

### 6.7.4 压缩合并（Squash merge）

```bash
# 将分支的所有提交压缩成一个提交
git merge --squash <branch-name>

# 需要手动提交
git commit -m "Squashed commit from feature/login"
```

**特点**：
- 将多个提交合并为一个
- 保持主分支历史干净
- 丢失了详细的提交历史

## 6.8 变基（Rebase）

### 6.8.1 什么是 Rebase

Rebase 会将一个分支的提交"移动"到另一个分支的顶端：

```bash
# 原始状态
main:    A---B---C---F
              \
feature:       D---E

# rebase 后
main:    A---B---C---F
                    \
feature:             D'---E'
```

### 6.8.2 使用 Rebase

```bash
# 切换到功能分支
git checkout feature

# 变基到 main
git rebase main

# 或一步到位
git rebase main feature
```

### 6.8.3 Rebase vs Merge

**Merge**：
- 保留完整的历史记录
- 产生合并提交
- 历史呈分叉状

**Rebase**：
- 产生线性的历史记录
- 不产生合并提交
- 改写提交历史

### 6.8.4 交互式 Rebase

```bash
# 交互式变基最近 3 个提交
git rebase -i HEAD~3

# 或变基到指定提交
git rebase -i <commit-id>
```

**交互式操作**：
- `pick`：保留提交
- `reword`：修改提交信息
- `edit`：编辑提交
- `squash`：合并到前一个提交
- `fixup`：合并到前一个提交，丢弃提交信息
- `drop`：删除提交

**示例**：
```
pick abc1234 第一个提交
squash def5678 第二个提交
squash ghi9012 第三个提交
```

### 6.8.5 Rebase 的黄金法则

**⚠️ 警告**：不要对已推送到公共仓库的提交使用 rebase！

**原因**：Rebase 会改写历史，可能导致其他人的代码出现问题。

**适用场景**：
- ✅ 个人的功能分支
- ✅ 未推送的本地提交
- ❌ 公共分支的已推送提交

## 6.9 分支管理实践

### 6.9.1 功能开发流程

```bash
# 1. 创建功能分支
git checkout -b feature/user-profile

# 2. 开发并提交
git add .
git commit -m "feat: 实现用户资料页面"

# 3. 完成开发，切换到主分支
git checkout main

# 4. 拉取最新代码
git pull

# 5. 合并功能分支
git merge feature/user-profile

# 6. 推送到远程
git push

# 7. 删除功能分支
git branch -d feature/user-profile
```

### 6.9.2 Bug 修复流程

```bash
# 1. 从主分支创建修复分支
git checkout main
git checkout -b hotfix/critical-bug

# 2. 修复 bug
git add .
git commit -m "fix: 修复关键bug"

# 3. 合并到主分支
git checkout main
git merge hotfix/critical-bug

# 4. 推送
git push

# 5. 如果有开发分支，也要合并到开发分支
git checkout develop
git merge hotfix/critical-bug
git push

# 6. 删除修复分支
git branch -d hotfix/critical-bug
```

### 6.9.3 长期分支策略

**主要分支**：
- `main`（或 `master`）：生产环境代码，最稳定
- `develop`：开发分支，包含最新的开发代码

**临时分支**：
- `feature/*`：功能分支，开发新功能
- `hotfix/*`：修复分支，修复生产环境的 bug
- `release/*`：发布分支，准备新版本发布

## 6.10 分支命名规范

### 6.10.1 常见命名约定

**功能分支**：
```
feature/user-authentication
feature/shopping-cart
feature/payment-integration
```

**修复分支**：
```
hotfix/login-error
hotfix/payment-bug
fix/typo-in-header
```

**发布分支**：
```
release/v1.0.0
release/v2.1.0
```

**其他分支**：
```
docs/update-readme
refactor/optimize-database
test/add-unit-tests
```

### 6.10.2 分支命名最佳实践

- 使用小写字母
- 使用连字符（`-`）分隔单词
- 使用描述性名称
- 包含相关的 issue 号（如果有）：`feature/123-user-login`
- 避免使用空格和特殊字符

## 6.11 查看分支图

### 6.11.1 命令行查看

```bash
# 简单图形
git log --graph --oneline

# 详细图形
git log --graph --oneline --all --decorate

# 带日期的图形
git log --graph --oneline --all --date=short

# 自定义格式
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all
```

### 6.11.2 配置别名

```bash
# 配置一个好看的分支图别名
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# 使用
git lg
```

## 6.12 常见问题

### 6.12.1 分支切换失败

```bash
# 错误：工作区有未提交的修改
error: Your local changes to the following files would be overwritten by checkout:
    file.txt
Please commit your changes or stash them before you switch branches.

# 解决方案1：提交修改
git add .
git commit -m "临时保存"

# 解决方案2：暂存修改
git stash
git switch other-branch
git stash pop

# 解决方案3：放弃修改
git restore .
git switch other-branch
```

### 6.12.2 误删分支怎么办

```bash
# 使用 reflog 找回
git reflog

# 找到被删除分支的最后一次提交
# 重新创建分支
git branch <branch-name> <commit-id>
```

### 6.12.3 分支合并冲突

```bash
# 合并时产生冲突
git merge feature
# 解决冲突
# ... 编辑冲突文件 ...
git add .
git commit

# 如果想放弃合并
git merge --abort
```

## 6.13 本地分支管理速查表

| 操作 | 命令 |
|------|------|
| 查看分支 | `git branch` |
| 查看所有分支 | `git branch -a` |
| 创建分支 | `git branch <name>` |
| 创建并切换 | `git switch -c <name>` |
| 切换分支 | `git switch <name>` |
| 重命名分支 | `git branch -m <new-name>` |
| 删除分支 | `git branch -d <name>` |
| 强制删除 | `git branch -D <name>` |
| 合并分支 | `git merge <branch>` |
| 变基分支 | `git rebase <branch>` |
| 查看分支图 | `git log --graph --oneline` |

## 6.14 总结

- **分支是指针**：Git 的分支非常轻量，本质是指向提交的指针
- **频繁使用分支**：为每个功能或修复创建独立分支
- **合理命名**：使用描述性的分支名称
- **merge vs rebase**：团队分支用 merge，个人分支用 rebase
- **定期清理**：删除已合并的分支，保持仓库整洁
- **遵循规范**：采用统一的分支管理策略

---

**上一章节**：[05. Git推送代码冲突解决实践方案](./05-git推送代码冲突解决实践方案.md)

**下一章节**：[07. Git远程分支管理](./07-git远程分支管理.md)
