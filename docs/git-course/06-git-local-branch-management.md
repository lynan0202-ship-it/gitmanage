# 06. Git本地分支管理

## 目录
- [什么是分支](#什么是分支)
- [为什么使用分支](#为什么使用分支)
- [分支的基本操作](#分支的基本操作)
- [分支合并](#分支合并)
- [分支管理策略](#分支管理策略)
- [高级分支操作](#高级分支操作)
- [实战场景](#实战场景)
- [最佳实践](#最佳实践)

## 什么是分支

分支（Branch）是Git中非常强大的功能，它允许你从主开发线上分离出来，在不影响主线的情况下继续工作。

### 分支的本质

```
分支只是一个指向某个提交的指针

main:    A -- B -- C (main)
                 \
feature:          D -- E (feature)

每个分支都是独立的开发线
```

### Git的分支特点

- **轻量级**：创建分支几乎是瞬间完成
- **快速切换**：在分支间切换非常快
- **独立开发**：各分支互不影响
- **易于合并**：Git提供强大的合并功能

## 为什么使用分支

### 1. 并行开发

```
main:     A -- B -- C -------------- H (稳定版本)
           \        \               /
feature-1:  \        D -- E -- F --/  (功能1)
             \                      \
feature-2:    G -- I -- J ----------- K (功能2)
```

### 2. 功能隔离

```
开发新功能时不会影响主分支
如果功能失败，直接删除分支即可
```

### 3. 实验性开发

```
可以创建临时分支进行实验
成功就合并，失败就删除
```

### 4. Bug修复

```
发现bug时，从稳定版本创建修复分支
修复完成后合并回主分支和其他需要的分支
```

## 分支的基本操作

### 查看分支

```bash
# 查看本地分支
git branch

# 查看所有分支（包括远程）
git branch -a

# 查看分支详细信息
git branch -v

# 查看分支及其追踪关系
git branch -vv

# 查看已合并的分支
git branch --merged

# 查看未合并的分支
git branch --no-merged
```

**示例输出**：

```bash
$ git branch
  feature-login
* main
  develop

# * 表示当前所在分支
```

### 创建分支

```bash
# 创建新分支（不切换）
git branch <branch-name>

# 创建并切换到新分支
git checkout -b <branch-name>

# 或使用switch命令（Git 2.23+）
git switch -c <branch-name>

# 从指定提交创建分支
git branch <branch-name> <commit-id>

# 从远程分支创建本地分支
git checkout -b <local-branch> origin/<remote-branch>
```

**示例**：

```bash
# 创建feature分支
git branch feature-login

# 创建并切换到feature分支
git checkout -b feature-payment

# 从历史提交创建分支
git log --oneline  # 查看提交历史
git branch hotfix a1b2c3d
```

### 切换分支

```bash
# 切换分支（checkout）
git checkout <branch-name>

# 切换分支（switch，Git 2.23+，推荐）
git switch <branch-name>

# 切换到上一个分支
git checkout -
# 或
git switch -

# 切换并创建分支
git checkout -b <new-branch>
git switch -c <new-branch>
```

**示例**：

```bash
# 在main分支
git branch
# * main

# 切换到feature分支
git switch feature-login

git branch
#   main
# * feature-login

# 切回main
git switch -
```

### 重命名分支

```bash
# 重命名当前分支
git branch -m <new-name>

# 重命名指定分支
git branch -m <old-name> <new-name>
```

**示例**：

```bash
# 当前在feature分支，重命名为feature-v2
git branch -m feature-v2

# 重命名其他分支
git branch -m old-feature new-feature
```

### 删除分支

```bash
# 删除已合并的分支
git branch -d <branch-name>

# 强制删除分支（未合并也删除）
git branch -D <branch-name>

# 删除多个分支
git branch -d branch1 branch2 branch3
```

**示例**：

```bash
# 合并后删除feature分支
git checkout main
git merge feature-login
git branch -d feature-login

# 放弃某个功能，强制删除
git branch -D abandoned-feature
```

## 分支合并

### Fast-Forward合并

当目标分支没有新提交时，Git会执行快进合并：

```bash
# 情况：
main:    A -- B -- C
              \
feature:       D -- E

# 合并后：
main:    A -- B -- C -- D -- E
```

**操作**：

```bash
git checkout main
git merge feature

# 输出：Fast-forward
```

### 三方合并

当两个分支都有新提交时，Git会创建合并提交：

```bash
# 情况：
main:    A -- B -- C ------- M
              \             /
feature:       D -- E -- F

# M是合并提交，有两个父提交
```

**操作**：

```bash
git checkout main
git merge feature

# 会打开编辑器让你输入合并信息
# 或者使用 -m 参数
git merge feature -m "合并feature分支"
```

### 禁止Fast-Forward

```bash
# 始终创建合并提交
git merge --no-ff feature

# 这样可以保留分支历史信息
```

**对比**：

```bash
# 使用 --ff（默认）
main: A -- B -- C -- D -- E

# 使用 --no-ff
main: A -- B -- C --------- M
            \             /
             D -- E -----
```

### 取消合并

```bash
# 合并前
git merge --abort

# 合并后想撤销
git reset --hard HEAD~1
```

## 分支管理策略

### 1. Git Flow

经典的分支管理模型：

```
主要分支：
- main (master)：生产环境代码
- develop：开发环境代码

支持分支：
- feature/*：功能开发
- release/*：发布准备
- hotfix/*：紧急修复
```

**工作流程**：

```bash
# 1. 从develop创建功能分支
git checkout develop
git checkout -b feature/user-login

# 2. 开发功能
# ... 编码 ...
git add .
git commit -m "feat: implement user login"

# 3. 合并回develop
git checkout develop
git merge --no-ff feature/user-login
git branch -d feature/user-login

# 4. 准备发布
git checkout -b release/1.0.0 develop
# 进行测试和bug修复
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# 5. 紧急修复
git checkout -b hotfix/critical-bug main
# 修复bug
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix v1.0.1"

git checkout develop
git merge --no-ff hotfix/critical-bug
git branch -d hotfix/critical-bug
```

### 2. GitHub Flow

简化的分支策略，适合持续部署：

```
- main分支始终可部署
- 从main创建描述性分支
- 定期推送到远程
- 通过Pull Request合并
- 合并后立即部署
```

**工作流程**：

```bash
# 1. 创建分支
git checkout -b add-user-profile

# 2. 提交修改
git add .
git commit -m "Add user profile page"
git push origin add-user-profile

# 3. 创建Pull Request（在GitHub上）

# 4. 代码审查和讨论

# 5. 合并到main
# 在GitHub上点击"Merge Pull Request"

# 6. 删除分支
git checkout main
git pull origin main
git branch -d add-user-profile
```

### 3. GitLab Flow

结合Git Flow和GitHub Flow的优点：

```
- 环境分支：main, staging, production
- 功能分支从main创建
- 通过Merge Request合并
- 按环境部署
```

## 高级分支操作

### Rebase（变基）

将一个分支的修改移动到另一个分支上：

```bash
# 场景：
main:    A -- B -- C
              \
feature:       D -- E

# rebase后：
main:    A -- B -- C
                   \
feature:            D' -- E'
```

**操作**：

```bash
# 在feature分支
git checkout feature
git rebase main

# 或者
git rebase main feature
```

**Rebase vs Merge**：

```bash
# Merge：保留完整历史
main:    A -- B -- C ------- M
              \             /
feature:       D -- E -----

# Rebase：线性历史
main:    A -- B -- C -- D -- E
```

### 交互式Rebase

```bash
# 修改最近3次提交
git rebase -i HEAD~3

# 会打开编辑器，显示：
# pick abc1234 commit message 1
# pick def5678 commit message 2
# pick ghi9012 commit message 3

# 可用命令：
# p, pick = 使用提交
# r, reword = 使用提交，但修改提交信息
# e, edit = 使用提交，但停下来修改
# s, squash = 使用提交，合并到前一个提交
# f, fixup = 类似squash，但丢弃提交信息
# d, drop = 删除提交
```

**示例**：合并多个提交

```bash
# 修改为：
pick abc1234 Add feature A
squash def5678 Fix typo in feature A
squash ghi9012 Update feature A documentation

# 保存后，三个提交会合并为一个
```

### Cherry-Pick

选择性地将某个提交应用到当前分支：

```bash
# 将特定提交应用到当前分支
git cherry-pick <commit-id>

# 应用多个提交
git cherry-pick <commit1> <commit2>

# 应用提交但不自动提交
git cherry-pick -n <commit-id>

# 应用一个范围的提交
git cherry-pick <start-commit>..<end-commit>
```

**示例场景**：

```bash
# feature分支有个bug修复，想应用到main
feature: A -- B -- C -- D (bug fix)
main:    A -- B -- E -- F

# 在main分支
git checkout main
git cherry-pick <commit-D-id>

# 结果：
main:    A -- B -- E -- F -- D'
```

### Stash（储藏）

临时保存未提交的修改：

```bash
# 保存当前修改
git stash

# 保存时添加描述
git stash save "WIP: feature implementation"

# 查看stash列表
git stash list

# 应用最近的stash
git stash apply

# 应用并删除stash
git stash pop

# 应用特定stash
git stash apply stash@{2}

# 删除stash
git stash drop stash@{0}

# 清空所有stash
git stash clear

# 从stash创建分支
git stash branch <branch-name>
```

**示例**：

```bash
# 正在开发功能A
vim feature-a.js
git status  # 有未提交的修改

# 突然需要修复紧急bug
git stash save "WIP: feature A development"

# 切换分支修复bug
git checkout main
git checkout -b hotfix
# 修复bug...
git commit -am "fix: critical bug"
git checkout main
git merge hotfix

# 回到功能开发
git checkout feature-a
git stash pop

# 继续开发
```

## 实战场景

### 场景1：开发新功能

```bash
# 1. 从main创建功能分支
git checkout main
git pull origin main
git checkout -b feature/shopping-cart

# 2. 开发功能（多次提交）
# 编写代码...
git add cart.js
git commit -m "feat: add shopping cart model"

# 编写测试...
git add cart.test.js
git commit -m "test: add cart tests"

# 3. 保持与main同步
git checkout main
git pull origin main
git checkout feature/shopping-cart
git merge main  # 或 git rebase main

# 4. 完成开发，合并到main
git checkout main
git merge --no-ff feature/shopping-cart
git push origin main

# 5. 删除功能分支
git branch -d feature/shopping-cart
```

### 场景2：修复Bug

```bash
# 1. 从main创建修复分支
git checkout main
git checkout -b bugfix/login-error

# 2. 修复bug
# 修改代码...
git add login.js
git commit -m "fix: resolve login authentication error"

# 3. 测试修复
npm test

# 4. 合并到main
git checkout main
git merge bugfix/login-error
git push origin main

# 5. 如果需要，合并到其他分支
git checkout develop
git merge bugfix/login-error

# 6. 删除修复分支
git branch -d bugfix/login-error
```

### 场景3：实验性功能

```bash
# 1. 创建实验分支
git checkout -b experiment/new-ui

# 2. 进行实验
# 编写代码...
git commit -am "试验新的UI设计"

# 3. 如果实验成功
git checkout main
git merge experiment/new-ui
git branch -d experiment/new-ui

# 或者实验失败
git checkout main
git branch -D experiment/new-ui  # 强制删除
```

### 场景4：版本发布

```bash
# 1. 从develop创建release分支
git checkout develop
git checkout -b release/2.0.0

# 2. 准备发布（更新版本号、文档等）
vim package.json  # 更新版本号
git commit -am "chore: bump version to 2.0.0"

# 3. 测试
npm test
npm run build

# 4. 合并到main并打标签
git checkout main
git merge --no-ff release/2.0.0
git tag -a v2.0.0 -m "Release version 2.0.0"
git push origin main --tags

# 5. 合并回develop
git checkout develop
git merge --no-ff release/2.0.0

# 6. 删除release分支
git branch -d release/2.0.0
```

## 最佳实践

### 1. 分支命名规范

```bash
# 功能分支
feature/user-authentication
feature/payment-integration

# 修复分支
bugfix/login-error
hotfix/critical-security-issue

# 发布分支
release/1.0.0
release/2.0.0

# 实验分支
experiment/new-algorithm
poc/microservices

# 避免使用：
# test, temp, wip（不够描述性）
```

### 2. 保持分支整洁

```bash
# 定期删除已合并的分支
git branch --merged | grep -v "\*" | xargs -n 1 git branch -d

# 定期同步远程分支
git fetch --prune
```

### 3. 提交前检查

```bash
# 切换分支前确保工作区干净
git status

# 如果有未提交的修改
git stash  # 或
git commit -am "WIP: save current work"
```

### 4. 分支保护

```bash
# 重要分支（如main）应该：
- 通过Pull Request合并
- 要求代码审查
- 运行CI/CD测试
- 限制直接推送权限
```

### 5. 使用描述性分支名

```bash
# ✅ 好的分支名
feature/user-profile-page
bugfix/memory-leak-in-parser
hotfix/payment-validation

# ❌ 不好的分支名
test
fix
branch1
```

## 总结

本节学习了Git本地分支管理：

✅ 理解分支的概念和优势  
✅ 掌握分支的基本操作  
✅ 学会合并分支的不同方式  
✅ 了解主流分支管理策略  
✅ 掌握高级分支操作技巧  
✅ 实战常见分支使用场景  
✅ 遵循分支管理最佳实践

**关键点**：
- 分支是Git最强大的功能之一
- 频繁创建和使用分支
- 选择适合团队的分支策略
- 保持分支整洁和有序

## 下一步

掌握了本地分支管理后，下一节我们将学习如何管理远程分支和团队协作。

---

[← 上一节：Git冲突解决](05-git-conflict-resolution.md) | [返回目录](../../README.md) | [下一节：Git远程分支管理 →](07-git-remote-branch-management.md)
