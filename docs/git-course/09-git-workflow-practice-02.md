# 09. Git工作流实践02

## 目录
- [代码审查最佳实践](#代码审查最佳实践)
- [提交规范](#提交规范)
- [分支命名规范](#分支命名规范)
- [标签管理](#标签管理)
- [自动化工作流](#自动化工作流)
- [常见问题和解决方案](#常见问题和解决方案)

## 代码审查最佳实践

### 为什么需要代码审查

```
代码审查的价值：
✅ 提高代码质量
✅ 知识共享
✅ 发现潜在bug
✅ 保持代码风格一致
✅ 培养团队协作
✅ 防止安全漏洞
```

### Pull Request最佳实践

#### 1. 创建高质量的PR

```markdown
# PR标题格式
<type>: <简短描述>

例如：
feat: 添加用户认证功能
fix: 修复登录页面内存泄漏
docs: 更新API文档

# PR描述模板
## 变更说明
简要描述这个PR的目的和实现方法

## 变更类型
- [ ] 新功能 (feature)
- [ ] Bug修复 (fix)
- [ ] 性能优化 (perf)
- [ ] 重构 (refactor)
- [ ] 文档更新 (docs)
- [ ] 测试 (test)
- [ ] 构建/工具 (chore)

## 测试
- [ ] 添加了新的测试
- [ ] 所有测试通过
- [ ] 手动测试通过

## 截图/录屏
[如果是UI变更，添加截图或录屏]

## 相关Issue
Closes #123
Related to #456

## 检查清单
- [ ] 代码符合项目规范
- [ ] 添加/更新了文档
- [ ] 添加/更新了测试
- [ ] 没有引入新的警告
- [ ] 通过了CI检查
```

#### 2. 保持PR小而专注

```bash
# ✅ 好的PR：单一职责
PR #1: feat: 实现用户登录功能
- 3个文件修改
- 150行代码变更

# ❌ 不好的PR：包含多个不相关的变更
PR #1: 各种更新
- 20个文件修改
- 2000行代码变更
- 包含登录、注册、密码重置、UI更新等
```

#### 3. 及时响应审查意见

```bash
# 审查者提出意见后

# 方式1：直接修改并推送
git add modified-file.js
git commit -m "refactor: address review comments"
git push origin feature-branch

# 方式2：使用fixup commit（后续squash）
git commit --fixup=<original-commit>
git push origin feature-branch

# 方式3：交互式rebase整理提交
git rebase -i main
# 将多个修复提交合并到相关的原始提交
git push --force-with-lease origin feature-branch
```

### 代码审查指南

#### 作为审查者

```markdown
### 审查重点

1. **功能正确性**
   - [ ] 代码实现了PR描述的功能
   - [ ] 没有明显的逻辑错误
   - [ ] 边界条件处理正确

2. **代码质量**
   - [ ] 代码清晰易读
   - [ ] 命名准确
   - [ ] 没有重复代码
   - [ ] 函数职责单一

3. **测试覆盖**
   - [ ] 有足够的测试
   - [ ] 测试覆盖关键路径
   - [ ] 测试用例有意义

4. **性能考虑**
   - [ ] 没有明显的性能问题
   - [ ] 数据库查询优化
   - [ ] 避免不必要的计算

5. **安全性**
   - [ ] 输入验证
   - [ ] 防止SQL注入
   - [ ] 防止XSS攻击
   - [ ] 敏感数据处理

6. **文档**
   - [ ] 复杂逻辑有注释
   - [ ] API文档更新
   - [ ] README更新（如需要）
```

#### 审查评论示例

```markdown
# ✅ 好的评论：建设性，具体

**问题**：
这里可能存在内存泄漏，事件监听器没有被移除。

**建议**：
```javascript
componentWillUnmount() {
  window.removeEventListener('resize', this.handleResize);
}
```

---

# ✅ 提问式评论
这里使用了同步操作，是否考虑改为异步？这样可以避免阻塞主线程。

---

# ✅ 称赞好的实现
👍 很好的抽象！这个工具函数可以在其他地方复用。

---

# ❌ 不好的评论：不具体，不友好

这段代码写得不好。

# ❌ 命令式语气
必须重写这个函数。

# ❌ 过于主观
我不喜欢这种写法。
```

## 提交规范

### Conventional Commits规范

#### 提交信息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Type类型

```bash
feat:     新功能
fix:      Bug修复
docs:     文档更新
style:    代码格式（不影响代码运行）
refactor: 重构（既不是新功能也不是修复bug）
perf:     性能优化
test:     测试相关
build:    构建系统或外部依赖变更
ci:       CI配置文件和脚本变更
chore:    其他不修改src或test的变更
revert:   回滚之前的提交
```

#### 示例

```bash
# 简单提交
git commit -m "feat: 添加用户搜索功能"

# 带scope
git commit -m "fix(auth): 修复登录验证逻辑错误"

# 带body和footer
git commit -m "feat(api): 添加用户API端点

- 实现GET /api/users
- 实现POST /api/users
- 添加参数验证

Closes #123"

# Breaking change
git commit -m "feat(api)!: 重构API响应格式

BREAKING CHANGE: API响应格式从数组改为对象结构
迁移指南：更新客户端代码以处理新的响应格式"

# 回滚
git commit -m "revert: feat(api): 添加用户API端点

This reverts commit abc123.
原因：发现该功能导致性能下降"
```

### 使用Commitlint

```bash
# 安装commitlint
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# 配置commitlint
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js

# 安装husky（Git钩子）
npm install --save-dev husky

# 配置commit-msg钩子
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'

# 现在无效的提交信息会被拒绝
git commit -m "bad commit"
# ❌ 失败：提交信息不符合规范

git commit -m "feat: add new feature"
# ✅ 成功
```

### 使用Commitizen

```bash
# 安装commitizen
npm install --save-dev commitizen cz-conventional-changelog

# 初始化
npx commitizen init cz-conventional-changelog --save-dev --save-exact

# 在package.json添加脚本
{
  "scripts": {
    "commit": "cz"
  }
}

# 使用交互式提交
npm run commit

# 会启动交互式界面：
? Select the type of change: (Use arrow keys)
❯ feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  style:    Changes that do not affect the meaning of the code
  refactor: A code change that neither fixes a bug nor adds a feature
  perf:     A code change that improves performance
  test:     Adding missing tests

? What is the scope of this change: auth
? Write a short description: implement JWT authentication
? Provide a longer description: (press enter to skip)
? Are there any breaking changes? No
? Does this change affect any open issues? Yes
? Add issue references: Closes #123
```

## 分支命名规范

### 命名模式

```bash
<type>/<description>

# 类型
feature/    # 新功能
bugfix/     # Bug修复
hotfix/     # 紧急修复
release/    # 发布准备
refactor/   # 重构
docs/       # 文档
test/       # 测试
chore/      # 杂项

# 描述：小写，用连字符分隔
```

### 示例

```bash
# ✅ 好的分支名
feature/user-authentication
feature/payment-integration
bugfix/login-form-validation
hotfix/security-vulnerability
release/v2.0.0
refactor/database-layer
docs/api-documentation
test/unit-tests-for-auth

# ❌ 不好的分支名
test                    # 太简单
my-branch              # 不清楚目的
feature_user_auth      # 使用下划线而非连字符
Feature/UserAuth       # 使用大写
fix-bug                # 不具体
temp                   # 临时性名称
```

### 分支命名配置

```bash
# 使用Git钩子强制分支命名规范

# .husky/pre-push
#!/bin/sh

branch=$(git rev-parse --abbrev-ref HEAD)

# 定义有效的分支名模式
valid_pattern="^(feature|bugfix|hotfix|release|refactor|docs|test|chore)/[a-z0-9-]+$"

if ! echo "$branch" | grep -qE "$valid_pattern"; then
    echo "❌ 无效的分支名: $branch"
    echo "✅ 有效格式: <type>/<description>"
    echo "例如: feature/user-login"
    exit 1
fi
```

## 标签管理

### 语义化版本

```
版本号格式：MAJOR.MINOR.PATCH

例如：1.2.3

MAJOR：不兼容的API修改
MINOR：向下兼容的功能新增
PATCH：向下兼容的问题修正
```

### 创建标签

```bash
# 轻量标签（不推荐用于发布）
git tag v1.0.0

# 附注标签（推荐）
git tag -a v1.0.0 -m "Release version 1.0.0"

# 为历史提交打标签
git tag -a v0.9.0 <commit-id> -m "Release version 0.9.0"

# 推送标签
git push origin v1.0.0

# 推送所有标签
git push origin --tags
```

### 标签命名规范

```bash
# 正式发布
v1.0.0
v2.1.3

# 预发布版本
v1.0.0-alpha.1
v1.0.0-beta.2
v1.0.0-rc.1

# 构建元数据
v1.0.0+20230615
```

### 查看和管理标签

```bash
# 查看所有标签
git tag

# 查看特定模式的标签
git tag -l "v1.*"

# 查看标签信息
git show v1.0.0

# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0

# 检出标签
git checkout v1.0.0
# 会进入"detached HEAD"状态

# 从标签创建分支
git checkout -b hotfix-1.0.1 v1.0.0
```

### 自动化版本管理

```bash
# 使用npm version
npm version patch    # 1.0.0 → 1.0.1
npm version minor    # 1.0.0 → 1.1.0
npm version major    # 1.0.0 → 2.0.0

# 自动创建git标签
npm version patch -m "chore: release version %s"

# 使用standard-version
npm install --save-dev standard-version

# package.json
{
  "scripts": {
    "release": "standard-version"
  }
}

# 执行发布
npm run release
# 自动：
# 1. 更新版本号
# 2. 生成CHANGELOG
# 3. 创建git标签
# 4. 提交变更
```

## 自动化工作流

### Git Hooks

#### 常用钩子

```bash
# 客户端钩子
pre-commit       # 提交前
commit-msg       # 提交信息检查
post-commit      # 提交后
pre-push         # 推送前

# 服务端钩子
pre-receive      # 接收前
update           # 更新时
post-receive     # 接收后
```

#### 使用Husky

```bash
# 安装
npm install --save-dev husky

# 初始化
npx husky install

# 添加pre-commit钩子（运行测试）
npx husky add .husky/pre-commit "npm test"

# 添加pre-commit钩子（代码格式化）
npx husky add .husky/pre-commit "npm run lint:fix"

# 添加commit-msg钩子（检查提交信息）
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'

# 添加pre-push钩子（运行构建）
npx husky add .husky/pre-push "npm run build"
```

#### lint-staged配置

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{json,md,yml}": [
      "prettier --write",
      "git add"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
}
```

### CI/CD集成

#### GitHub Actions示例

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
    
    - name: Upload coverage
      uses: codecov/codecov-action@v2

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Deploy to production
      run: npm run deploy
      env:
        DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

#### GitLab CI示例

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "16"

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run lint
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:production:
  stage: deploy
  script:
    - npm run deploy
  only:
    - main
  environment:
    name: production
    url: https://example.com
```

## 常见问题和解决方案

### 问题1：提交了敏感信息

```bash
# 从历史中完全删除敏感文件
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive-file" \
  --prune-empty --tag-name-filter cat -- --all

# 或使用BFG Repo-Cleaner（更快）
java -jar bfg.jar --delete-files sensitive-file.txt
java -jar bfg.jar --replace-text passwords.txt  # 替换密码

# 强制推送
git push origin --force --all
git push origin --force --tags

# 通知团队重新克隆仓库
```

### 问题2：历史提交太多太乱

```bash
# 使用交互式rebase整理提交
git rebase -i HEAD~5

# 在编辑器中：
pick abc1234 feat: add login
squash def5678 fix: typo
squash ghi9012 refactor: improve code
pick jkl3456 test: add tests

# 保存后，多个提交会被合并
```

### 问题3：分支太多难以管理

```bash
# 查看已合并的分支
git branch --merged

# 批量删除已合并的分支
git branch --merged | grep -v "\*" | grep -v "main" | grep -v "develop" | xargs -n 1 git branch -d

# 清理远程已删除的分支引用
git fetch --prune

# 查看未合并的分支
git branch --no-merged
```

### 问题4：需要修改多个历史提交

```bash
# 修改作者信息
git filter-branch --env-filter '
if [ "$GIT_COMMITTER_EMAIL" = "old@email.com" ]; then
    export GIT_COMMITTER_NAME="New Name"
    export GIT_COMMITTER_EMAIL="new@email.com"
    export GIT_AUTHOR_NAME="New Name"
    export GIT_AUTHOR_EMAIL="new@email.com"
fi
' -- --all

# 或使用git-filter-repo（推荐）
git filter-repo --email-callback '
  return email.replace(b"old@email.com", b"new@email.com")
'
```

### 问题5：需要将子目录拆分为独立仓库

```bash
# 使用filter-branch
git filter-branch --subdirectory-filter path/to/subdir -- --all

# 或使用git-subtree
git subtree split -P path/to/subdir -b new-branch

# 创建新仓库
mkdir new-repo
cd new-repo
git init
git pull /path/to/old-repo new-branch
```

## 团队协作检查清单

```markdown
### 开始新功能前
- [ ] 同步最新代码
- [ ] 创建描述性分支
- [ ] 了解功能需求

### 开发过程中
- [ ] 频繁提交
- [ ] 写好提交信息
- [ ] 定期推送到远程
- [ ] 保持与主分支同步

### 提交PR前
- [ ] 运行所有测试
- [ ] 运行linter
- [ ] 检查代码格式
- [ ] 写好PR描述
- [ ] 指定审查者

### PR审查中
- [ ] 及时响应评论
- [ ] 修复发现的问题
- [ ] 保持沟通

### 合并后
- [ ] 删除功能分支
- [ ] 更新本地仓库
- [ ] 确认部署成功
```

## 总结

本节学习了Git工作流的高级实践：

✅ 代码审查最佳实践  
✅ 提交信息规范  
✅ 分支命名规范  
✅ 标签管理  
✅ 自动化工作流  
✅ 常见问题解决方案

**关键点**：
- 规范是为了提高效率，不是束缚
- 自动化可以避免人为错误
- 代码审查是团队成长的机会
- 持续改进工作流程

## 下一步

学习了工作流实践后，下一节我们将通过完整的演示来综合运用所学知识。

---

[← 上一节：Git工作流实践01](08-git-workflow-practice-01.md) | [返回目录](../../README.md) | [下一节：Git工作流实践演示 →](10-git-workflow-demo.md)
