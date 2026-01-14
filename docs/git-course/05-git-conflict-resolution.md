# 05. Git推送代码冲突解决实践方案

## 目录
- [什么是代码冲突](#什么是代码冲突)
- [冲突产生的原因](#冲突产生的原因)
- [如何预防冲突](#如何预防冲突)
- [识别冲突](#识别冲突)
- [解决冲突的方法](#解决冲突的方法)
- [实战演练](#实战演练)
- [使用工具解决冲突](#使用工具解决冲突)
- [最佳实践](#最佳实践)

## 什么是代码冲突

当两个或多个开发者修改了同一个文件的同一部分，Git无法自动决定保留哪个版本时，就会产生冲突（Conflict）。

### 冲突的本质

```
时间线：
    A ---- B ---- C (你的本地版本)
         \
          ---- D ---- E (远程版本)

当你尝试推送C到远程时，Git发现远程已经有了D和E，
如果C和D/E修改了相同的内容，就会产生冲突。
```

## 冲突产生的原因

### 1. 并行开发

两个开发者同时修改同一个文件：

```
开发者A：修改了 user.js 的第10行
开发者B：也修改了 user.js 的第10行

当A先推送，B后推送时 → 冲突！
```

### 2. 分支合并

```bash
# main分支有新提交
main:  A -- B -- C

# feature分支也有提交
feature: A -- D -- E

# 当尝试合并时，如果C和E修改了相同内容 → 冲突！
git merge feature
```

### 3. 代码重构

```
开发者A：重命名了一个函数
开发者B：调用了旧的函数名

合并时 → 冲突！
```

## 如何预防冲突

### 1. 频繁拉取更新

```bash
# 每天开始工作前
git pull origin main

# 或者在推送前
git pull origin main
git push origin main
```

### 2. 小步提交，及时推送

```bash
# 完成一个小功能就提交推送
git add feature.js
git commit -m "feat: 实现用户登录功能"
git push origin main

# 而不是积累很多修改再提交
```

### 3. 使用分支开发

```bash
# 为每个功能创建独立分支
git checkout -b feature-login
# 开发...
git push origin feature-login
```

### 4. 明确分工

- 团队约定：不同开发者负责不同的模块
- 减少在同一文件上并行工作
- 使用代码审查流程

### 5. 良好的代码结构

```javascript
// ❌ 不好：所有代码在一个文件
// app.js (5000行)

// ✅ 好：模块化
// auth.js
// user.js
// product.js
```

## 识别冲突

### 推送时的冲突提示

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

### 合并时的冲突提示

```bash
$ git merge feature-branch

Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

### 查看冲突状态

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

## 解决冲突的方法

### 方法1：手动解决冲突

#### 步骤1：拉取远程代码

```bash
git pull origin main
```

#### 步骤2：查看冲突文件

冲突标记格式：

```
<<<<<<< HEAD
你的修改
=======
远程的修改
>>>>>>> branch-name
```

**实际示例**：

```html
<!DOCTYPE html>
<html>
<head>
<<<<<<< HEAD
    <title>我的网站</title>
=======
    <title>My Website</title>
>>>>>>> origin/main
</head>
<body>
    <h1>欢迎</h1>
</body>
</html>
```

#### 步骤3：编辑文件解决冲突

选项1：保留你的修改

```html
<!DOCTYPE html>
<html>
<head>
    <title>我的网站</title>
</head>
<body>
    <h1>欢迎</h1>
</body>
</html>
```

选项2：保留远程修改

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>
    <h1>欢迎</h1>
</body>
</html>
```

选项3：保留两者

```html
<!DOCTYPE html>
<html>
<head>
    <title>我的网站 - My Website</title>
</head>
<body>
    <h1>欢迎</h1>
</body>
</html>
```

#### 步骤4：标记为已解决

```bash
# 添加已解决的文件
git add index.html

# 查看状态
git status
# 输出：All conflicts fixed but you are still merging.
```

#### 步骤5：完成合并

```bash
# 提交合并
git commit -m "解决index.html的合并冲突"

# 推送到远程
git push origin main
```

### 方法2：使用命令行工具

#### 接受当前分支的版本（ours）

```bash
# 解决冲突时，保留当前分支的版本
git checkout --ours <file>
git add <file>
```

#### 接受合并分支的版本（theirs）

```bash
# 解决冲突时，保留被合并分支的版本
git checkout --theirs <file>
git add <file>
```

**示例**：

```bash
# 拉取时产生冲突
git pull origin main

# 完全接受远程版本
git checkout --theirs index.html
git add index.html
git commit -m "接受远程版本的index.html"
```

### 方法3：放弃合并

```bash
# 放弃当前合并，恢复到合并前的状态
git merge --abort

# 或者在pull时
git pull origin main
# 如果有冲突且想放弃
git merge --abort
```

### 方法4：使用rebase避免合并提交

```bash
# 使用rebase而不是merge
git pull --rebase origin main

# 如果有冲突，解决后
git add <resolved-file>
git rebase --continue

# 如果想放弃rebase
git rebase --abort
```

## 实战演练

### 场景1：简单文本冲突

**初始文件 `README.md`**：

```markdown
# My Project
Description here
```

**开发者A的修改**：

```markdown
# My Project
This is a awesome project
```

**开发者B的修改**：

```markdown
# My Project
This is a great project
```

**解决步骤**：

```bash
# 开发者B拉取代码
git pull origin main

# 出现冲突，查看文件
cat README.md
```

**冲突内容**：

```markdown
# My Project
<<<<<<< HEAD
This is a great project
=======
This is a awesome project
>>>>>>> origin/main
```

**解决方案**：结合两者

```markdown
# My Project
This is an awesome and great project
```

**完成解决**：

```bash
git add README.md
git commit -m "resolve: 合并README描述"
git push origin main
```

### 场景2：代码逻辑冲突

**初始代码 `app.js`**：

```javascript
function getUserName() {
    return "User";
}
```

**开发者A的修改**：

```javascript
function getUserName(id) {
    return database.query(id).name;
}
```

**开发者B的修改**：

```javascript
function getUserName() {
    return session.currentUser.name;
}
```

**冲突内容**：

```javascript
<<<<<<< HEAD
function getUserName() {
    return session.currentUser.name;
=======
function getUserName(id) {
    return database.query(id).name;
>>>>>>> origin/main
}
```

**解决方案**：综合考虑需求

```javascript
function getUserName(id = null) {
    if (id) {
        return database.query(id).name;
    }
    return session.currentUser.name;
}
```

### 场景3：文件删除冲突

**场景**：
- 开发者A删除了一个文件
- 开发者B修改了同一个文件

**解决步骤**：

```bash
# 查看状态
git status
# deleted by us:   old-file.js

# 决定1：确认删除
git rm old-file.js
git commit -m "确认删除old-file.js"

# 决定2：保留修改
git add old-file.js
git commit -m "保留并更新old-file.js"
```

### 场景4：二进制文件冲突

二进制文件（如图片）无法自动合并，必须选择一个版本：

```bash
# 保留本地版本
git checkout --ours image.png
git add image.png

# 或保留远程版本
git checkout --theirs image.png
git add image.png

git commit -m "解决image.png冲突"
```

## 使用工具解决冲突

### VS Code

VS Code内置了优秀的冲突解决界面：

1. 打开有冲突的文件
2. VS Code会高亮冲突区域
3. 提供按钮选项：
   - Accept Current Change（接受当前修改）
   - Accept Incoming Change（接受传入修改）
   - Accept Both Changes（接受两者）
   - Compare Changes（比较差异）

```bash
# 用VS Code打开冲突文件
code <conflicted-file>
```

### Git GUI工具

#### GitKraken

```
1. 自动检测冲突
2. 可视化显示冲突内容
3. 点击选择要保留的版本
4. 自动完成合并提交
```

#### SourceTree

```
1. 在文件状态中显示冲突
2. 右键选择"解决冲突"
3. 使用内置或外部合并工具
4. 标记为已解决
```

### 命令行合并工具

#### vimdiff

```bash
# 配置vimdiff为合并工具
git config merge.tool vimdiff
git config merge.conflictstyle diff3
git config mergetool.prompt false

# 使用vimdiff解决冲突
git mergetool
```

#### meld

```bash
# 安装meld
# Ubuntu: sudo apt install meld
# Mac: brew install meld

# 配置meld
git config merge.tool meld

# 使用meld解决冲突
git mergetool
```

### 配置自定义合并工具

```bash
# 配置VS Code为合并工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# 使用
git mergetool
```

## 最佳实践

### 1. 合并前的准备

```bash
# 确保工作区干净
git status

# 提交所有修改
git add .
git commit -m "保存当前工作"

# 拉取最新代码
git pull origin main
```

### 2. 小心处理冲突

```bash
# ✅ 仔细阅读冲突内容
# ✅ 理解双方的意图
# ✅ 测试解决后的代码
# ✅ 如果不确定，询问相关开发者

# ❌ 不要随意删除冲突标记
# ❌ 不要盲目接受一方的修改
# ❌ 不要在未测试的情况下提交
```

### 3. 使用分支策略

```bash
# 功能开发在独立分支
git checkout -b feature-login

# 定期同步主分支
git checkout main
git pull origin main
git checkout feature-login
git merge main  # 在自己的分支解决冲突

# 功能完成后合并
git checkout main
git merge feature-login
```

### 4. 代码审查

```bash
# 在解决冲突后
# 1. 让团队成员审查冲突解决方案
# 2. 运行测试
# 3. 确认功能正常
# 4. 再推送到远程
```

### 5. 沟通协作

```
冲突不仅是技术问题，更是沟通问题

✅ 在修改重要模块前通知团队
✅ 使用任务管理系统协调工作
✅ 定期同步代码状态
✅ 遇到复杂冲突时讨论解决方案
```

## 高级技巧

### 1. 三方合并

```bash
# 使用diff3冲突样式，显示共同祖先
git config --global merge.conflictstyle diff3
```

**效果**：

```
<<<<<<< HEAD
你的修改
||||||| base
原始内容
=======
他人修改
>>>>>>> branch
```

### 2. 重新合并（rerere）

```bash
# 启用rerere（重用已记录的解决方案）
git config --global rerere.enabled true

# Git会记住如何解决冲突
# 下次遇到相同冲突时自动应用
```

### 3. 交互式rebase

```bash
# 在rebase过程中逐个处理冲突
git rebase -i main

# 对每个冲突的提交
# 1. 解决冲突
# 2. git add <file>
# 3. git rebase --continue
```

### 4. 策略性合并

```bash
# 使用ours策略（保留当前分支）
git merge -s ours branch-name

# 使用theirs策略（Git 2.20+）
git merge -X theirs branch-name

# 使用ours策略选项（优先当前分支）
git merge -X ours branch-name
```

## 故障排除

### 问题1：不小心提交了冲突标记

```bash
# 检查是否有冲突标记
git grep -n "<<<<<<< HEAD"

# 如果发现了，修复后重新提交
git add <file>
git commit --amend
```

### 问题2：合并后发现选错了版本

```bash
# 撤销合并提交
git reset --hard HEAD~1

# 重新合并
git merge branch-name
```

### 问题3：rebase过程中想放弃

```bash
# 放弃rebase
git rebase --abort

# 返回到rebase前的状态
```

### 问题4：合并时丢失了某些修改

```bash
# 查看reflog
git reflog

# 找到合并前的提交
git reset --hard HEAD@{n}

# 或使用cherry-pick恢复特定提交
git cherry-pick <commit-id>
```

## 冲突解决检查清单

```
□ 理解冲突的来源和原因
□ 与相关开发者沟通（如需要）
□ 仔细阅读双方的修改
□ 选择或整合修改内容
□ 删除所有冲突标记（<<<<, ====, >>>>）
□ 测试代码功能
□ 运行测试套件
□ 代码审查（如需要）
□ git add 标记为已解决
□ git commit 提交合并
□ git push 推送到远程
□ 通知团队冲突已解决
```

## 总结

本节学习了Git冲突解决的完整流程：

✅ 理解冲突产生的原因  
✅ 学会预防冲突的方法  
✅ 掌握识别冲突的技巧  
✅ 学会多种解决冲突的方法  
✅ 实战演练常见冲突场景  
✅ 使用工具辅助解决冲突  
✅ 掌握最佳实践和高级技巧

**关键点**：
- 频繁拉取，小步提交
- 仔细处理冲突，不要盲目操作
- 善用工具，提高效率
- 加强沟通，减少冲突

## 下一步

掌握了冲突解决技巧后，下一节我们将深入学习Git的本地分支管理。

---

[← 上一节：Git代码回退撤销操作](04-git-code-rollback-operations.md) | [返回目录](../../README.md) | [下一节：Git本地分支管理 →](06-git-local-branch-management.md)
