# 05. Git推送代码冲突解决实践方案

## 5.1 什么是代码冲突

### 5.1.1 冲突的产生

当两个或多个开发者修改了同一个文件的同一部分代码，Git 无法自动合并时，就会产生冲突（Conflict）。

```
开发者A的本地仓库          开发者B的本地仓库
      ↓                          ↓
  修改 file.txt              修改 file.txt
  第10行代码                 第10行代码
      ↓                          ↓
  先推送到远程 ✓             后推送到远程 ✗
                                  ↓
                            产生冲突！
```

### 5.1.2 冲突的类型

1. **内容冲突**：同一文件的同一位置被不同的修改
2. **删除冲突**：一方删除文件，另一方修改文件
3. **重命名冲突**：一方重命名文件，另一方修改文件

## 5.2 冲突的表现形式

### 5.2.1 推送时的冲突提示

```bash
$ git push origin main

To https://github.com/username/repo.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/username/repo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
```

### 5.2.2 合并时的冲突提示

```bash
$ git pull origin main

Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

### 5.2.3 冲突文件的标记

```html
<<<<<<< HEAD
<h1>欢迎来到我的网站</h1>
=======
<h1>Welcome to My Website</h1>
>>>>>>> origin/main
```

**冲突标记说明**：
- `<<<<<<< HEAD`：当前分支的内容开始
- `=======`：分隔符
- `>>>>>>> origin/main`：远程分支的内容结束

## 5.3 预防冲突的最佳实践

### 5.3.1 频繁拉取更新

```bash
# 开始工作前先拉取
git pull

# 推送前再次拉取
git pull
git push
```

### 5.3.2 小步提交，频繁推送

```bash
# 完成一个小功能就提交推送
git add .
git commit -m "feat: 添加用户注册功能"
git push
```

### 5.3.3 使用功能分支

```bash
# 每个功能使用独立的分支
git checkout -b feature/user-login

# 开发完成后再合并到主分支
git checkout main
git pull
git merge feature/user-login
```

### 5.3.4 团队约定

- **分工明确**：避免多人同时修改同一文件
- **代码规范**：统一代码格式，减少格式冲突
- **及时沟通**：大的修改提前通知团队

## 5.4 解决推送冲突

### 5.4.1 基本流程

```bash
# 1. 拉取远程更新
git pull

# 2. 解决冲突（编辑冲突文件）
# ...

# 3. 添加已解决的文件
git add .

# 4. 提交合并
git commit -m "merge: 解决冲突"

# 5. 推送
git push
```

### 5.4.2 详细步骤

**步骤1：拉取远程代码**

```bash
$ git pull origin main

Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

**步骤2：查看冲突文件**

```bash
# 查看哪些文件有冲突
git status

# 输出示例：
# On branch main
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   index.html
```

**步骤3：打开冲突文件**

```html
<!DOCTYPE html>
<html>
<head>
    <title>我的网站</title>
</head>
<body>
<<<<<<< HEAD
    <h1>欢迎来到我的网站</h1>
    <p>这是本地修改的内容</p>
=======
    <h1>Welcome to My Website</h1>
    <p>这是远程修改的内容</p>
>>>>>>> origin/main
</body>
</html>
```

**步骤4：手动解决冲突**

```html
<!DOCTYPE html>
<html>
<head>
    <title>我的网站</title>
</head>
<body>
    <h1>欢迎来到我的网站 - Welcome to My Website</h1>
    <p>这是合并后的内容</p>
</body>
</html>
```

**步骤5：标记为已解决**

```bash
# 添加已解决的文件
git add index.html

# 查看状态
git status
```

**步骤6：提交合并**

```bash
# 提交（Git 会自动生成合并信息）
git commit

# 或手动指定提交信息
git commit -m "merge: 解决 index.html 的冲突"
```

**步骤7：推送到远程**

```bash
git push origin main
```

## 5.5 使用 merge 还是 rebase

### 5.5.1 git pull = git fetch + git merge

```bash
# 默认行为
git pull
# 等同于
git fetch origin
git merge origin/main
```

**特点**：
- 保留完整的提交历史
- 会产生额外的合并提交
- 提交历史呈现分支状态

### 5.5.2 git pull --rebase

```bash
git pull --rebase
# 等同于
git fetch origin
git rebase origin/main
```

**特点**：
- 提交历史是线性的
- 不会产生额外的合并提交
- 本地提交会被"重放"到远程提交之后

### 5.5.3 merge vs rebase 对比

**使用 merge**：

```
A - B - C (本地)
     \
      D - E (远程)
           \
            F (合并提交)
```

**使用 rebase**：

```
A - B - D - E - C' (本地提交被重放)
```

### 5.5.4 选择建议

**使用 merge 的场景**：
- 团队协作的公共分支（如 main）
- 需要保留完整的合并历史
- 默认推荐的方式

**使用 rebase 的场景**：
- 个人的功能分支
- 希望保持线性的提交历史
- 在推送前整理本地提交

**⚠️ 注意**：不要对已推送的提交使用 rebase！

## 5.6 使用工具解决冲突

### 5.6.1 配置合并工具

```bash
# 使用 VSCode 作为合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# 使用 kdiff3
git config --global merge.tool kdiff3

# 使用 meld
git config --global merge.tool meld

# 使用 p4merge
git config --global merge.tool p4merge
```

### 5.6.2 使用合并工具解决冲突

```bash
# 拉取代码产生冲突后
git pull

# 启动合并工具
git mergetool

# 工具会逐个打开冲突文件，在图形界面中解决冲突

# 解决完成后
git commit
git push
```

### 5.6.3 VS Code 解决冲突

VS Code 内置了冲突解决功能：

```
<<<<<<< HEAD (Current Change)
本地内容
=======
远程内容
>>>>>>> origin/main (Incoming Change)
```

VS Code 会显示按钮：
- **Accept Current Change**：保留本地修改
- **Accept Incoming Change**：保留远程修改
- **Accept Both Changes**：保留两者
- **Compare Changes**：对比差异

### 5.6.4 命令行查看冲突

```bash
# 查看冲突的详细信息
git diff

# 查看我们的版本
git show :2:filename

# 查看他们的版本
git show :3:filename

# 查看基础版本
git show :1:filename
```

## 5.7 特殊冲突的解决

### 5.7.1 二进制文件冲突

二进制文件（如图片、视频）无法合并，需要选择一个版本：

```bash
# 保留我们的版本
git checkout --ours <file>
git add <file>

# 保留他们的版本
git checkout --theirs <file>
git add <file>
```

### 5.7.2 删除文件冲突

```bash
# 一方删除文件，另一方修改文件

# 保留删除操作
git rm <file>
git add <file>

# 保留修改操作
git add <file>
```

### 5.7.3 重命名冲突

```bash
# 一方重命名文件，另一方修改文件

# 查看冲突
git status

# 手动解决：重命名文件并保留修改
mv old_name new_name
git add new_name
git rm old_name
```

## 5.8 中止合并操作

### 5.8.1 放弃合并

```bash
# 如果冲突太复杂，可以放弃合并
git merge --abort

# 或者放弃 rebase
git rebase --abort

# 回到合并前的状态
```

### 5.8.2 重新开始

```bash
# 放弃当前的合并
git merge --abort

# 重新拉取
git fetch origin
git merge origin/main

# 或使用 rebase
git pull --rebase
```

## 5.9 实战案例

### 5.9.1 案例1：简单的内容冲突

**场景**：两人同时修改了 README.md 的同一行。

```bash
# 1. 拉取代码
$ git pull origin main
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md

# 2. 查看冲突
$ git status
Unmerged paths:
  both modified:   README.md

# 3. 打开文件解决冲突
$ vim README.md

# 原内容：
<<<<<<< HEAD
# 项目标题：用户管理系统
=======
# 项目标题：User Management System
>>>>>>> origin/main

# 解决后：
# 项目标题：用户管理系统 (User Management System)

# 4. 标记为已解决
$ git add README.md

# 5. 提交
$ git commit -m "merge: 解决 README 标题冲突"

# 6. 推送
$ git push origin main
```

### 5.9.2 案例2：多个文件冲突

```bash
# 1. 拉取代码
$ git pull origin main
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Auto-merging style.css
CONFLICT (content): Merge conflict in style.css

# 2. 查看所有冲突文件
$ git status
Unmerged paths:
  both modified:   index.html
  both modified:   style.css

# 3. 逐个解决冲突
$ vim index.html
# ... 解决冲突 ...
$ git add index.html

$ vim style.css
# ... 解决冲突 ...
$ git add style.css

# 4. 确认所有冲突已解决
$ git status
All conflicts fixed

# 5. 提交
$ git commit -m "merge: 解决多个文件的冲突"

# 6. 推送
$ git push origin main
```

### 5.9.3 案例3：使用 rebase 解决冲突

```bash
# 1. 使用 rebase 拉取
$ git pull --rebase origin main
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html

# 2. 解决冲突
$ vim index.html
# ... 解决冲突 ...
$ git add index.html

# 3. 继续 rebase
$ git rebase --continue

# 如果还有冲突，重复步骤2-3

# 4. 推送（可能需要强制推送）
$ git push origin main
# 如果推送失败
$ git push --force-with-lease origin main
```

## 5.10 团队协作的冲突管理策略

### 5.10.1 使用分支策略

**Git Flow 工作流**：

```
main (生产分支)
  ↓
develop (开发分支)
  ↓
feature/* (功能分支)
```

每个功能独立开发，减少冲突：

```bash
# 创建功能分支
git checkout -b feature/user-login develop

# 开发完成后合并
git checkout develop
git pull origin develop
git merge feature/user-login
git push origin develop
```

### 5.10.2 代码审查

使用 Pull Request (PR) 流程：

```bash
# 1. 推送功能分支
git push origin feature/user-login

# 2. 在 GitHub/GitLab 创建 PR

# 3. 团队审查代码

# 4. 解决审查意见和冲突

# 5. 合并到主分支
```

### 5.10.3 持续集成

- 自动化测试：确保合并后代码可用
- 自动化构建：及早发现问题
- 自动化部署：快速交付

## 5.11 冲突解决速查表

| 场景 | 命令 |
|------|------|
| 拉取并合并 | `git pull` |
| 拉取并 rebase | `git pull --rebase` |
| 查看冲突文件 | `git status` |
| 查看冲突详情 | `git diff` |
| 标记冲突已解决 | `git add <file>` |
| 提交合并 | `git commit` |
| 使用合并工具 | `git mergetool` |
| 保留我们的版本 | `git checkout --ours <file>` |
| 保留他们的版本 | `git checkout --theirs <file>` |
| 放弃合并 | `git merge --abort` |
| 放弃 rebase | `git rebase --abort` |
| 继续 rebase | `git rebase --continue` |

## 5.12 总结

- **冲突产生**：多人修改同一文件的同一位置
- **预防冲突**：频繁拉取、小步提交、使用分支
- **解决流程**：pull → 解决冲突 → add → commit → push
- **merge vs rebase**：团队分支用 merge，个人分支用 rebase
- **工具辅助**：使用 IDE 或图形工具解决冲突
- **团队协作**：使用分支策略和 PR 流程

---

**上一章节**：[04. Git各阶段代码修改回退撤销操作](./04-git各阶段代码修改回退撤销操作.md)

**下一章节**：[06. Git本地分支管理](./06-git本地分支管理.md)
