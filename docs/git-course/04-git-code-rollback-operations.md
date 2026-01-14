# 04. Git各阶段代码修改回退撤销操作

## 目录
- [Git三个区域回顾](#git三个区域回顾)
- [撤销工作区的修改](#撤销工作区的修改)
- [撤销暂存区的修改](#撤销暂存区的修改)
- [撤销提交](#撤销提交)
- [重置操作详解](#重置操作详解)
- [回退到历史版本](#回退到历史版本)
- [恢复已删除的提交](#恢复已删除的提交)
- [常见场景与解决方案](#常见场景与解决方案)

## Git三个区域回顾

在学习撤销操作之前，先回顾Git的三个工作区域：

```
工作区                暂存区               本地仓库
(Working Directory)   (Staging Area)       (Repository)
     |                     |                     |
     |--- git add -------->|                     |
     |                     |--- git commit ----->|
     |<-- git restore -----|                     |
     |<-- git reset ----------------------------|
     |<-- git checkout --------------------------|
```

## 撤销工作区的修改

### 场景：修改了文件但还未添加到暂存区

#### 使用 git restore（推荐，Git 2.23+）

```bash
# 撤销单个文件的修改
git restore <file>

# 撤销多个文件的修改
git restore <file1> <file2>

# 撤销当前目录下所有文件的修改
git restore .

# 撤销所有已跟踪文件的修改
git restore :/
```

**示例**：

```bash
# 修改了README.md但想撤销
echo "wrong content" >> README.md
git status
# 输出：modified: README.md

# 撤销修改
git restore README.md

# 再次查看状态
git status
# 输出：nothing to commit, working tree clean
```

#### 使用 git checkout（旧版本）

```bash
# 撤销单个文件的修改
git checkout -- <file>

# 撤销所有修改
git checkout -- .
```

### 场景：删除了文件但想恢复

```bash
# 情况1：仅在工作区删除（未git add）
rm file.txt
git restore file.txt

# 情况2：已经git add删除操作
git rm file.txt
git restore --staged file.txt  # 先从暂存区恢复
git restore file.txt            # 再从工作区恢复
```

### 场景：丢弃所有未跟踪的文件

```bash
# 查看会删除哪些文件（预览）
git clean -n

# 删除未跟踪的文件
git clean -f

# 删除未跟踪的文件和目录
git clean -fd

# 删除未跟踪的文件（包括.gitignore中的文件）
git clean -fx

# 交互式清理
git clean -i
```

**示例**：

```bash
# 创建了一些临时文件
touch temp1.txt temp2.txt
mkdir temp-dir
touch temp-dir/file.txt

# 预览将要删除的文件
git clean -n
# 输出：Would remove temp1.txt
#       Would remove temp2.txt

# 删除未跟踪的文件和目录
git clean -fd
```

## 撤销暂存区的修改

### 场景：已经git add但还未commit

#### 使用 git restore --staged（推荐，Git 2.23+）

```bash
# 将文件从暂存区移除，但保留工作区的修改
git restore --staged <file>

# 移除所有暂存的文件
git restore --staged .
```

**示例**：

```bash
# 修改文件并添加到暂存区
echo "new feature" > feature.txt
git add feature.txt

# 查看状态
git status
# 输出：Changes to be committed:
#       new file: feature.txt

# 从暂存区移除
git restore --staged feature.txt

# 再次查看状态
git status
# 输出：Untracked files:
#       feature.txt
```

#### 使用 git reset HEAD（旧版本）

```bash
# 将文件从暂存区移除
git reset HEAD <file>

# 移除所有暂存的文件
git reset HEAD .
```

### 完全撤销：从暂存区和工作区都删除

```bash
# 方法1：分步操作
git restore --staged <file>  # 先从暂存区移除
git restore <file>            # 再从工作区撤销

# 方法2：直接使用checkout（旧版本）
git checkout HEAD <file>
```

## 撤销提交

### 场景1：撤销最后一次提交，保留修改

```bash
# 撤销提交，保留暂存区和工作区的修改
git reset --soft HEAD~1

# 或者使用
git reset --soft HEAD^
```

**示例**：

```bash
# 进行了一次提交
git commit -m "Add feature"

# 发现提交信息写错了，撤销提交
git reset --soft HEAD~1

# 现在可以重新提交
git commit -m "feat: Add user authentication feature"
```

### 场景2：撤销最后一次提交，但保留工作区修改

```bash
# 撤销提交和暂存，但保留工作区的修改
git reset --mixed HEAD~1

# 或者简写（--mixed是默认选项）
git reset HEAD~1
```

**示例**：

```bash
# 提交了多个不相关的修改
git add file1.txt file2.txt
git commit -m "Update files"

# 想要分开提交，撤销这次提交
git reset HEAD~1

# 现在可以分别提交
git add file1.txt
git commit -m "feat: Update file1"

git add file2.txt
git commit -m "fix: Fix bug in file2"
```

### 场景3：完全撤销最后一次提交

```bash
# 彻底撤销提交，删除所有修改
git reset --hard HEAD~1

# ⚠️ 警告：此操作会永久删除修改，请谨慎使用！
```

### 场景4：修改最后一次提交

```bash
# 修改最后一次提交的信息
git commit --amend -m "新的提交信息"

# 添加遗漏的文件到最后一次提交
git add forgotten-file.txt
git commit --amend --no-edit

# 修改提交内容和信息
git add modified-file.txt
git commit --amend -m "更新后的提交信息"
```

**示例**：

```bash
# 提交后发现漏了一个文件
git commit -m "feat: Add login feature"

# 添加遗漏的文件
git add login.css
git commit --amend --no-edit

# 现在这个文件会包含在同一个提交中
```

## 重置操作详解

### git reset 的三种模式

```bash
# --soft：撤销提交，保留暂存区和工作区
git reset --soft <commit>

# --mixed（默认）：撤销提交和暂存，保留工作区
git reset --mixed <commit>
# 或
git reset <commit>

# --hard：撤销提交、暂存和工作区的所有修改
git reset --hard <commit>
```

### 三种模式对比

| 模式 | HEAD位置 | 暂存区 | 工作区 | 使用场景 |
|------|---------|--------|--------|---------|
| --soft | 移动 | 不变 | 不变 | 重新整理提交 |
| --mixed | 移动 | 重置 | 不变 | 撤销暂存，保留修改 |
| --hard | 移动 | 重置 | 重置 | 完全撤销，危险操作 |

**图解**：

```bash
# 假设有以下提交历史
A -- B -- C -- D (HEAD, main)

# git reset --soft B
# HEAD移到B，C和D的修改在暂存区
A -- B (HEAD, main)
     暂存区：C和D的修改

# git reset --mixed B
# HEAD移到B，C和D的修改在工作区
A -- B (HEAD, main)
     工作区：C和D的修改

# git reset --hard B
# HEAD移到B，C和D的修改完全丢失
A -- B (HEAD, main)
```

### HEAD引用说明

```bash
# HEAD~1 或 HEAD^ : 上一个提交
# HEAD~2 或 HEAD^^ : 上上个提交
# HEAD~3 : 往前数第3个提交

# 示例
git reset --soft HEAD~1   # 撤销最后1次提交
git reset --soft HEAD~3   # 撤销最后3次提交

# 使用commit ID
git reset --soft abc1234  # 重置到指定提交
```

## 回退到历史版本

### 查看提交历史

```bash
# 查看完整历史
git log

# 查看简洁历史
git log --oneline

# 查看最近n次提交
git log -n 5

# 查看图形化历史
git log --graph --oneline --all

# 查看某个文件的历史
git log <file>
```

### 回退到指定版本

```bash
# 方式1：使用reset（会改变历史）
git reset --hard <commit-id>

# 方式2：使用revert（创建新提交，不改变历史）
git revert <commit-id>
```

**示例**：

```bash
# 查看历史
git log --oneline
# d4f2e1a (HEAD -> main) feat: Add feature C
# b3e4a2c feat: Add feature B
# a1b2c3d feat: Add feature A
# 9f8e7d6 Initial commit

# 回退到feature A（使用reset）
git reset --hard a1b2c3d

# 或者使用revert撤销feature C和B
git revert d4f2e1a  # 撤销C
git revert b3e4a2c  # 撤销B
```

### reset vs revert

| 特性 | reset | revert |
|------|-------|--------|
| 改变历史 | 是 | 否 |
| 生成新提交 | 否 | 是 |
| 适用场景 | 本地修改 | 已推送的修改 |
| 安全性 | 较危险 | 较安全 |
| 团队协作 | 不推荐 | 推荐 |

**最佳实践**：

```bash
# 本地未推送的修改：使用reset
git reset --hard HEAD~1

# 已推送到远程的修改：使用revert
git revert <commit-id>
git push origin main
```

## 恢复已删除的提交

### 使用 git reflog

```bash
# 查看所有操作历史（包括被删除的提交）
git reflog

# 恢复到之前的状态
git reset --hard <commit-id>
```

**示例场景**：

```bash
# 1. 不小心hard reset了
git reset --hard HEAD~3
# 哦不！我的3个提交没了！

# 2. 查看reflog找到之前的提交
git reflog
# 输出：
# a1b2c3d HEAD@{0}: reset: moving to HEAD~3
# d4f2e1a HEAD@{1}: commit: feat: Add feature C
# b3e4a2c HEAD@{2}: commit: feat: Add feature B
# ...

# 3. 恢复到reset之前的状态
git reset --hard d4f2e1a

# 4. 提交又回来了！
git log --oneline
```

### 恢复已删除的分支

```bash
# 1. 查看reflog
git reflog

# 2. 找到分支删除前的commit
# 3. 重新创建分支
git branch <branch-name> <commit-id>

# 或者直接checkout创建分支
git checkout -b <branch-name> <commit-id>
```

## 常见场景与解决方案

### 场景1：修改了文件但想撤销

```bash
# 问题：修改了index.html但想恢复
vim index.html  # 做了一些修改

# 解决：
git restore index.html
```

### 场景2：不小心add了不该提交的文件

```bash
# 问题：
git add secret.key  # 不应该add这个文件

# 解决：
git restore --staged secret.key

# 确保不再误提交
echo "secret.key" >> .gitignore
```

### 场景3：提交信息写错了

```bash
# 问题：
git commit -m "fix bug"  # 提交信息太简单

# 解决：
git commit --amend -m "fix: 修复用户登录验证的空指针异常"
```

### 场景4：提交了但发现漏了文件

```bash
# 问题：
git commit -m "feat: Add login page"
# 发现忘记add login.css了

# 解决：
git add login.css
git commit --amend --no-edit
```

### 场景5：把多个修改误提交到一起了

```bash
# 问题：
git add .
git commit -m "Update files"
# 应该分成多个提交

# 解决：
git reset HEAD~1  # 撤销提交
git add file1.js
git commit -m "feat: Add feature A"
git add file2.js
git commit -m "fix: Fix bug in feature B"
```

### 场景6：提交到错误的分支了

```bash
# 问题：在main分支上提交了应该在feature分支的代码

# 解决方案1：移动提交到新分支
git branch feature-branch    # 创建新分支（包含当前提交）
git reset --hard HEAD~1       # 在main上撤销提交
git checkout feature-branch   # 切换到新分支

# 解决方案2：使用cherry-pick
git checkout feature-branch
git cherry-pick main         # 复制main的提交到当前分支
git checkout main
git reset --hard HEAD~1      # 在main上撤销提交
```

### 场景7：想要撤销某个历史提交

```bash
# 问题：提交历史中某个提交有问题
# A -- B -- C -- D -- E (HEAD)
# 想要撤销C的修改

# 解决：使用revert
git revert C的commit-id

# 这会创建一个新提交E'，撤销C的修改
# A -- B -- C -- D -- E -- E' (HEAD)
```

### 场景8：合并了错误的分支

```bash
# 问题：合并了错误的分支
git merge wrong-branch

# 解决：撤销合并
git reset --hard HEAD~1

# 或者如果已经推送了
git revert -m 1 HEAD
```

### 场景9：文件冲突想要放弃合并

```bash
# 问题：合并时出现冲突，想要放弃合并

# 解决：
git merge --abort

# 或者在rebase时
git rebase --abort
```

### 场景10：想要临时保存当前修改

```bash
# 问题：正在开发功能A，突然需要修复紧急bug
# 但当前修改还不能提交

# 解决：使用stash
git stash save "功能A开发中"

# 切换分支修复bug
git checkout -b hotfix
# ... 修复bug ...
git commit -am "fix: 紧急bug修复"

# 回到原来的工作
git checkout main
git stash pop  # 恢复之前保存的修改
```

## 危险操作警告

### 永远不要对已推送的提交执行以下操作

```bash
# ❌ 危险：会改变历史
git reset --hard <commit>
git rebase
git commit --amend

# ✅ 安全：不改变历史
git revert <commit>
```

### 推送强制更新的风险

```bash
# ⚠️ 非常危险：会覆盖远程仓库
git push --force

# 相对安全：如果远程有别人的提交会失败
git push --force-with-lease
```

## 撤销操作决策树

```
需要撤销吗？
├─ 是，文件还未add
│  └─ git restore <file>
│
├─ 是，文件已add但未commit
│  ├─ 只从暂存区移除：git restore --staged <file>
│  └─ 完全撤销：git restore --staged <file> && git restore <file>
│
├─ 是，已经commit但未push
│  ├─ 保留修改：git reset --soft HEAD~1
│  ├─ 修改提交信息：git commit --amend
│  └─ 完全撤销：git reset --hard HEAD~1
│
└─ 是，已经push到远程
   ├─ 撤销最近的提交：git revert HEAD
   └─ 撤销某个历史提交：git revert <commit-id>
```

## 总结

本节学习了Git的各种撤销操作：

✅ 撤销工作区的修改（git restore）  
✅ 撤销暂存区的修改（git restore --staged）  
✅ 撤销提交（git reset / git revert）  
✅ 理解reset的三种模式  
✅ 回退到历史版本  
✅ 恢复已删除的提交（git reflog）  
✅ 各种常见场景的解决方案

**关键点**：
- 未push的修改可以大胆使用reset
- 已push的修改应该使用revert
- git reflog是撤销操作的后悔药

## 下一步

掌握了代码回退操作后，下一节我们将学习如何解决代码推送时的冲突问题。

---

[← 上一节：Git常用命令](03-git-common-commands.md) | [返回目录](../../README.md) | [下一节：Git冲突解决 →](05-git-conflict-resolution.md)
