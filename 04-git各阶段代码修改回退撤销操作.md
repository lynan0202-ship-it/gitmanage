# 04. Git各阶段代码修改回退撤销操作

## 4.1 Git 的三个工作区域

在学习回退和撤销操作之前，需要理解 Git 的三个重要区域：

```
工作区（Working Directory）
    ↓ git add
暂存区（Staging Area/Index）
    ↓ git commit
本地仓库（Local Repository）
    ↓ git push
远程仓库（Remote Repository）
```

## 4.2 文件的四种状态

```
Untracked（未跟踪）
    ↓ git add
Staged（已暂存）
    ↓ git commit
Committed（已提交）
    ↓ 修改后
Modified（已修改）
```

## 4.3 工作区的撤销操作

### 4.3.1 撤销单个文件的修改

```bash
# 旧版本命令（Git < 2.23）
git checkout -- <file>

# 新版本命令（Git >= 2.23，推荐）
git restore <file>

# 示例
git restore index.html
```

**注意**：这个操作会丢失工作区的修改，无法恢复！

### 4.3.2 撤销所有文件的修改

```bash
# 撤销所有修改
git restore .

# 或
git checkout -- .
```

### 4.3.3 撤销指定目录的修改

```bash
# 撤销指定目录下的所有修改
git restore src/

# 或
git checkout -- src/
```

### 4.3.4 查看将要撤销的内容

```bash
# 先查看修改内容
git diff <file>

# 确认后再撤销
git restore <file>
```

## 4.4 暂存区的撤销操作

### 4.4.1 从暂存区移除文件（保留工作区修改）

```bash
# 新版本命令（推荐）
git restore --staged <file>

# 旧版本命令
git reset HEAD <file>

# 示例
git restore --staged index.html
```

**效果**：文件从暂存区移除，但工作区的修改保留。

### 4.4.2 从暂存区移除所有文件

```bash
# 移除暂存区的所有文件
git restore --staged .

# 或
git reset HEAD .
```

### 4.4.3 同时撤销暂存区和工作区

```bash
# 从暂存区移除并丢弃工作区修改
git restore --staged --worktree <file>

# 或分两步
git restore --staged <file>
git restore <file>
```

## 4.5 提交的撤销操作

### 4.5.1 git reset 的三种模式

`git reset` 是最强大也最危险的撤销命令，有三种模式：

#### --soft：软重置

```bash
git reset --soft HEAD~1
```

**效果**：
- 移动 HEAD 指针
- 保留暂存区的修改
- 保留工作区的修改

**场景**：想重新编辑提交信息或合并多个提交。

#### --mixed：混合重置（默认）

```bash
git reset HEAD~1
# 等同于
git reset --mixed HEAD~1
```

**效果**：
- 移动 HEAD 指针
- 清空暂存区（修改回到工作区）
- 保留工作区的修改

**场景**：想重新组织提交内容。

#### --hard：硬重置

```bash
git reset --hard HEAD~1
```

**效果**：
- 移动 HEAD 指针
- 清空暂存区
- 丢弃工作区的修改

**场景**：完全放弃最近的提交和所有修改。

**⚠️ 警告**：这是危险操作，会永久丢失修改！

### 4.5.2 撤销到指定提交

```bash
# 撤销到指定 commit
git reset --soft <commit-id>
git reset --mixed <commit-id>
git reset --hard <commit-id>

# 撤销最近 n 次提交
git reset --soft HEAD~n
git reset --mixed HEAD~n
git reset --hard HEAD~n
```

### 4.5.3 三种模式对比

| 模式 | HEAD | 暂存区 | 工作区 |
|------|------|--------|--------|
| --soft | 移动 | 不变 | 不变 |
| --mixed | 移动 | 重置 | 不变 |
| --hard | 移动 | 重置 | 重置 |

### 4.5.4 实战示例

```bash
# 场景1：修改最后一次提交
git reset --soft HEAD~1
# 重新添加文件
git add .
# 重新提交
git commit -m "新的提交信息"

# 场景2：撤销提交但保留修改
git reset HEAD~1
# 修改会回到工作区，可以重新组织

# 场景3：完全撤销提交
git reset --hard HEAD~1
# 提交和修改都丢失
```

## 4.6 修改最后一次提交

### 4.6.1 修改提交信息

```bash
# 只修改提交信息
git commit --amend -m "新的提交信息"

# 打开编辑器修改
git commit --amend
```

### 4.6.2 添加遗漏的文件

```bash
# 提交后发现漏了文件
git add forgotten_file.txt
git commit --amend --no-edit

# 或者同时修改提交信息
git add forgotten_file.txt
git commit --amend -m "添加完整的功能"
```

### 4.6.3 注意事项

**⚠️ 警告**：
- `--amend` 会改变提交的 SHA-1 值
- 如果已经推送到远程，不要使用 `--amend`
- 或者使用 `git push --force-with-lease`（慎用）

## 4.7 使用 git revert 撤销提交

### 4.7.1 revert vs reset

**git reset**：
- 移动 HEAD 指针
- 改变历史
- 不适合已推送的提交

**git revert**：
- 创建新的提交来撤销
- 不改变历史
- 适合已推送的提交

### 4.7.2 使用 git revert

```bash
# 撤销指定提交
git revert <commit-id>

# 撤销最近一次提交
git revert HEAD

# 撤销多个提交
git revert HEAD~3..HEAD

# 只撤销但不提交（后续可以批量提交）
git revert -n <commit-id>
# 或
git revert --no-commit <commit-id>
```

### 4.7.3 示例

```bash
# 假设提交历史
# A - B - C - D (HEAD)

# 撤销提交 C
git revert C

# 提交历史变为
# A - B - C - D - C' (HEAD)
# 其中 C' 是撤销 C 的新提交
```

## 4.8 使用 git reflog 恢复丢失的提交

### 4.8.1 什么是 reflog

`git reflog` 记录了 HEAD 的移动历史，即使提交被删除也能找回。

```bash
# 查看 reflog
git reflog

# 查看详细信息
git reflog show
```

### 4.8.2 恢复丢失的提交

```bash
# 1. 查看 reflog
git reflog

# 输出示例：
# abc1234 HEAD@{0}: reset: moving to HEAD~1
# def5678 HEAD@{1}: commit: 添加新功能
# ...

# 2. 恢复到指定的提交
git reset --hard def5678

# 或使用 HEAD@{n} 语法
git reset --hard HEAD@{1}
```

### 4.8.3 实战示例

```bash
# 误操作：硬重置丢失了提交
git reset --hard HEAD~3

# 查看 reflog 找到丢失的提交
git reflog

# 恢复
git reset --hard HEAD@{1}
```

## 4.9 删除文件的撤销

### 4.9.1 撤销删除操作

```bash
# 误删了文件
rm file.txt

# 从 Git 恢复
git restore file.txt
# 或
git checkout -- file.txt
```

### 4.9.2 撤销 git rm

```bash
# 使用 git rm 删除了文件
git rm file.txt

# 从暂存区恢复
git restore --staged file.txt
git restore file.txt

# 或一步到位
git reset HEAD file.txt
git checkout -- file.txt
```

## 4.10 清理未跟踪的文件

### 4.10.1 查看将被删除的文件

```bash
# 查看将被删除的文件
git clean -n

# 查看将被删除的文件和目录
git clean -nd
```

### 4.10.2 删除未跟踪的文件

```bash
# 删除未跟踪的文件
git clean -f

# 删除未跟踪的文件和目录
git clean -fd

# 删除未跟踪的文件（包括 .gitignore 中的）
git clean -fx
```

**⚠️ 警告**：`git clean` 是危险操作，会永久删除文件！

## 4.11 暂存当前工作

### 4.11.1 使用 git stash

```bash
# 暂存当前修改
git stash

# 暂存并添加说明
git stash save "工作描述"

# 查看暂存列表
git stash list

# 恢复最近的暂存
git stash pop

# 恢复指定的暂存
git stash apply stash@{0}

# 删除暂存
git stash drop stash@{0}

# 清空所有暂存
git stash clear
```

### 4.11.2 stash 的应用场景

```bash
# 场景：正在开发功能，突然需要修复紧急bug

# 1. 暂存当前工作
git stash save "开发中的登录功能"

# 2. 切换到 main 分支修复 bug
git checkout main
# ... 修复 bug ...
git add .
git commit -m "fix: 修复紧急bug"
git push

# 3. 回到功能分支
git checkout feature/login

# 4. 恢复之前的工作
git stash pop
```

## 4.12 撤销远程仓库的提交

### 4.12.1 使用 git revert（推荐）

```bash
# 1. 撤销远程的某次提交
git revert <commit-id>

# 2. 推送到远程
git push
```

### 4.12.2 使用 git reset（慎用）

```bash
# 1. 本地重置
git reset --hard <commit-id>

# 2. 强制推送
git push --force-with-lease

# 或（更危险）
git push -f
```

**⚠️ 警告**：
- 强制推送会改写远程历史
- 可能导致其他人的代码冲突
- 团队协作时慎用

## 4.13 常见场景与解决方案

### 场景1：修改了文件但还没 add

```bash
# 撤销修改
git restore <file>
```

### 场景2：add 了但还没 commit

```bash
# 从暂存区移除
git restore --staged <file>
# 如果还想撤销工作区修改
git restore <file>
```

### 场景3：commit 了但还没 push

```bash
# 方案1：保留修改，重新提交
git reset --soft HEAD~1

# 方案2：修改最后一次提交
git commit --amend

# 方案3：完全撤销
git reset --hard HEAD~1
```

### 场景4：已经 push 到远程

```bash
# 推荐：使用 revert
git revert <commit-id>
git push

# 慎用：强制推送
git reset --hard <commit-id>
git push --force-with-lease
```

### 场景5：误删了未提交的文件

```bash
# 如果文件已被 Git 跟踪
git restore <file>

# 如果是未跟踪的新文件
# 无法恢复（可以尝试系统级恢复工具）
```

### 场景6：想回到某个历史版本查看

```bash
# 临时查看
git checkout <commit-id>

# 查看后回到最新版本
git checkout main
```

## 4.14 撤销操作速查表

| 场景 | 命令 |
|------|------|
| 撤销工作区修改 | `git restore <file>` |
| 从暂存区移除 | `git restore --staged <file>` |
| 撤销最后一次提交 | `git reset --soft HEAD~1` |
| 撤销提交并丢弃修改 | `git reset --hard HEAD~1` |
| 修改最后一次提交 | `git commit --amend` |
| 撤销已推送的提交 | `git revert <commit-id>` |
| 恢复删除的文件 | `git restore <file>` |
| 暂存当前工作 | `git stash` |
| 恢复暂存的工作 | `git stash pop` |
| 找回丢失的提交 | `git reflog` |

## 4.15 总结

- **工作区撤销**：`git restore <file>` 丢弃工作区修改
- **暂存区撤销**：`git restore --staged <file>` 移除暂存但保留修改
- **提交撤销**：`git reset` 有三种模式（soft/mixed/hard）
- **安全撤销**：`git revert` 创建新提交来撤销，不改变历史
- **救命稻草**：`git reflog` 可以找回丢失的提交
- **暂存工作**：`git stash` 临时保存未完成的工作

---

**上一章节**：[03. Git常用的代码拉取-修改-提交-推送命令](./03-git常用的代码拉取-修改-提交-推送命令.md)

**下一章节**：[05. Git推送代码冲突解决实践方案](./05-git推送代码冲突解决实践方案.md)
