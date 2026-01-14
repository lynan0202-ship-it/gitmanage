# 08. Git工作流实践01

## 目录
- [什么是Git工作流](#什么是git工作流)
- [Git Flow工作流](#git-flow工作流)
- [GitHub Flow工作流](#github-flow工作流)
- [GitLab Flow工作流](#gitlab-flow工作流)
- [工作流对比](#工作流对比)
- [选择合适的工作流](#选择合适的工作流)

## 什么是Git工作流

Git工作流（Git Workflow）是团队在使用Git进行协作开发时遵循的一套规范和流程。它定义了：

- 如何组织分支结构
- 何时创建和合并分支
- 如何进行代码审查
- 怎样发布版本

### 为什么需要工作流

```
没有工作流的问题：
❌ 分支混乱，不知道哪个是稳定版本
❌ 代码冲突频繁
❌ 发布流程不清晰
❌ 难以回滚到特定版本
❌ 团队协作效率低

使用工作流的好处：
✅ 清晰的分支结构
✅ 标准化的开发流程
✅ 更好的代码质量控制
✅ 易于追踪和管理版本
✅ 提高团队协作效率
```

## Git Flow工作流

Git Flow是由Vincent Driessen提出的经典分支管理模型，适合有定期发布周期的项目。

### 分支结构

```
主分支（长期存在）：
├─ main (master)    # 生产环境，只包含发布版本
└─ develop          # 开发环境，最新的开发进度

辅助分支（临时存在）：
├─ feature/*        # 功能开发分支
├─ release/*        # 发布准备分支
└─ hotfix/*         # 紧急修复分支
```

### 分支详解

#### 1. Main分支（主分支）

```bash
# 特点：
- 永远处于生产就绪状态
- 只能从release或hotfix分支合并
- 每次合并都应该打标签

# 操作：
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags
```

#### 2. Develop分支（开发分支）

```bash
# 特点：
- 包含最新的开发进度
- 从main分支创建
- 是feature分支的基础

# 创建：
git checkout -b develop main

# 操作：
git checkout develop
git pull origin develop
```

#### 3. Feature分支（功能分支）

```bash
# 特点：
- 从develop分支创建
- 用于开发新功能
- 完成后合并回develop
- 可以推送到远程进行协作

# 命名：feature/<feature-name>
# 例如：feature/user-login, feature/payment

# 创建和开发：
git checkout develop
git checkout -b feature/user-authentication

# 开发功能...
git add .
git commit -m "feat: implement user authentication"

# 完成后合并回develop：
git checkout develop
git merge --no-ff feature/user-authentication
git branch -d feature/user-authentication
git push origin develop
```

#### 4. Release分支（发布分支）

```bash
# 特点：
- 从develop分支创建
- 用于准备新版本发布
- 只能修复bug，不能添加新功能
- 完成后合并到main和develop

# 命名：release/<version>
# 例如：release/1.0.0, release/2.1.0

# 创建：
git checkout develop
git checkout -b release/1.0.0

# 更新版本号等：
echo "1.0.0" > version.txt
git commit -am "chore: bump version to 1.0.0"

# Bug修复（如果需要）：
git commit -am "fix: correct typo in documentation"

# 完成发布：
# 1. 合并到main
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# 2. 合并回develop
git checkout develop
git merge --no-ff release/1.0.0

# 3. 删除release分支
git branch -d release/1.0.0

# 4. 推送
git push origin main develop --tags
```

#### 5. Hotfix分支（热修复分支）

```bash
# 特点：
- 从main分支创建
- 用于修复生产环境的紧急bug
- 完成后合并到main和develop（或当前的release）

# 命名：hotfix/<version>
# 例如：hotfix/1.0.1, hotfix/critical-bug

# 创建：
git checkout main
git checkout -b hotfix/1.0.1

# 修复bug：
git commit -am "fix: resolve critical security vulnerability"

# 更新版本号：
echo "1.0.1" > version.txt
git commit -am "chore: bump version to 1.0.1"

# 完成修复：
# 1. 合并到main
git checkout main
git merge --no-ff hotfix/1.0.1
git tag -a v1.0.1 -m "Hotfix version 1.0.1"

# 2. 合并到develop
git checkout develop
git merge --no-ff hotfix/1.0.1

# 3. 删除hotfix分支
git branch -d hotfix/1.0.1

# 4. 推送
git push origin main develop --tags
```

### Git Flow完整示例

```bash
# 初始化项目
git init
git commit --allow-empty -m "Initial commit"
git checkout -b develop

# 功能1：用户登录
git checkout -b feature/user-login develop
# 开发...
git commit -am "feat: add login page"
git commit -am "feat: implement login logic"
git checkout develop
git merge --no-ff feature/user-login
git branch -d feature/user-login

# 功能2：用户注册
git checkout -b feature/user-register develop
# 开发...
git commit -am "feat: add register page"
git checkout develop
git merge --no-ff feature/user-register
git branch -d feature/user-register

# 准备发布
git checkout -b release/1.0.0 develop
echo "1.0.0" > version.txt
git commit -am "chore: bump version to 1.0.0"
# 修复发布前的小bug...
git commit -am "fix: correct validation error"

# 发布到生产
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# 紧急修复
git checkout main
git checkout -b hotfix/1.0.1
git commit -am "fix: resolve security issue"
echo "1.0.1" > version.txt
git commit -am "chore: bump version to 1.0.1"
git checkout main
git merge --no-ff hotfix/1.0.1
git tag -a v1.0.1 -m "Hotfix version 1.0.1"
git checkout develop
git merge --no-ff hotfix/1.0.1
git branch -d hotfix/1.0.1
```

### Git Flow工具

#### 使用git-flow扩展

```bash
# 安装git-flow
# Mac
brew install git-flow

# Ubuntu
sudo apt-get install git-flow

# Windows
# 随Git for Windows安装

# 初始化git-flow
git flow init

# 使用默认配置或自定义分支名称

# 开始新功能
git flow feature start user-profile

# 完成功能
git flow feature finish user-profile

# 开始发布
git flow release start 1.0.0

# 完成发布
git flow release finish 1.0.0

# 开始热修复
git flow hotfix start 1.0.1

# 完成热修复
git flow hotfix finish 1.0.1
```

### Git Flow优缺点

**优点**：
✅ 清晰的分支结构  
✅ 适合版本化发布  
✅ 支持并行开发  
✅ 严格的发布流程  
✅ 易于回滚

**缺点**：
❌ 分支较多，较复杂  
❌ 不适合持续部署  
❌ 合并操作频繁  
❌ 学习曲线较陡

**适用场景**：
- 定期发布的软件产品
- 需要维护多个版本
- 大型团队协作
- 企业级应用

## GitHub Flow工作流

GitHub Flow是GitHub提出的轻量级工作流，专注于持续部署。

### 核心原则

```
1. main分支始终可部署
2. 从main创建描述性分支
3. 定期推送到远程
4. 通过Pull Request合并
5. 合并后立即部署
```

### 分支结构

```
main (唯一的长期分支)
├─ feature/add-payment
├─ fix/login-bug
├─ refactor/database
└─ docs/api-guide
```

### 工作流程

#### 1. 创建分支

```bash
# 从main创建特性分支
git checkout main
git pull origin main
git checkout -b feature/user-dashboard
```

#### 2. 添加提交

```bash
# 进行小步提交
git add dashboard.js
git commit -m "feat: create dashboard component"

git add dashboard.css
git commit -m "style: add dashboard styles"

git add dashboard.test.js
git commit -m "test: add dashboard tests"
```

#### 3. 推送到远程

```bash
# 定期推送到远程（作为备份和协作）
git push -u origin feature/user-dashboard

# 后续推送
git push origin feature/user-dashboard
```

#### 4. 创建Pull Request

```bash
# 在GitHub上：
1. 点击"New Pull Request"
2. 选择 base: main, compare: feature/user-dashboard
3. 填写标题和描述
4. 添加标签、里程碑、审查者
5. 点击"Create Pull Request"
```

#### 5. 讨论和审查

```markdown
# Pull Request描述模板

## 变更说明
实现用户仪表板功能

## 变更类型
- [x] 新功能
- [ ] Bug修复
- [ ] 重构
- [ ] 文档更新

## 测试
- [x] 单元测试通过
- [x] 集成测试通过
- [x] 手动测试通过

## 截图
[添加截图展示变更]

## 相关Issue
Closes #123
```

#### 6. 部署测试

```bash
# 在合并前部署到测试环境
# （通过CI/CD自动化）

# 或手动部署
git checkout feature/user-dashboard
# 部署到staging环境测试
```

#### 7. 合并到main

```bash
# 在GitHub上点击"Merge Pull Request"

# 或使用命令行
git checkout main
git merge --no-ff feature/user-dashboard
git push origin main
```

#### 8. 部署到生产

```bash
# 合并后自动触发部署
# （通过CI/CD pipeline）

# 或手动部署
git checkout main
git pull origin main
# 部署到生产环境
```

#### 9. 删除分支

```bash
# 在GitHub上自动删除

# 或手动删除
git branch -d feature/user-dashboard
git push origin --delete feature/user-dashboard
```

### GitHub Flow完整示例

```bash
# 1. 同步main分支
git checkout main
git pull origin main

# 2. 创建新分支
git checkout -b add-search-feature

# 3. 实现搜索功能
echo "search component" > search.js
git add search.js
git commit -m "feat: add search component"

echo "search styles" > search.css
git add search.css
git commit -m "style: add search styles"

# 4. 推送到远程
git push -u origin add-search-feature

# 5. 在GitHub上创建Pull Request

# 6. 响应代码审查
# 根据反馈修改代码
git add search.js
git commit -m "refactor: improve search algorithm"
git push origin add-search-feature

# 7. CI测试通过，审查通过，合并
# 在GitHub上点击"Merge"

# 8. 更新本地main
git checkout main
git pull origin main

# 9. 删除本地分支
git branch -d add-search-feature
```

### GitHub Flow优缺点

**优点**：
✅ 简单易懂  
✅ 适合持续部署  
✅ 快速迭代  
✅ 分支少，易管理  
✅ 强制代码审查

**缺点**：
❌ 不适合版本化发布  
❌ 不适合维护多版本  
❌ 依赖自动化测试  
❌ main分支压力大

**适用场景**：
- Web应用和SaaS产品
- 持续部署环境
- 小型到中型团队
- 快速迭代项目

## GitLab Flow工作流

GitLab Flow结合了Git Flow和GitHub Flow的优点，提供更灵活的工作流。

### 核心概念

```
1. 上游优先（Upstream First）
2. 环境分支（Environment Branches）
3. 发布分支（Release Branches）
```

### 工作模式

#### 模式1：环境分支

```
main (开发环境)
├─ feature/xxx
│
├─ staging (预发布环境)
│
└─ production (生产环境)

代码流向：
main → staging → production
```

**实施**：

```bash
# 功能开发
git checkout main
git checkout -b feature/new-feature
# 开发...
git commit -am "feat: new feature"

# 合并到main
git checkout main
git merge feature/new-feature
git push origin main
# 自动部署到开发环境

# 部署到staging
git checkout staging
git merge main
git push origin staging
# 自动部署到预发布环境
# 进行测试...

# 部署到production
git checkout production
git merge staging
git push origin production
# 自动部署到生产环境
```

#### 模式2：发布分支

```
main (开发)
├─ 1-0-stable (1.0版本维护)
├─ 2-0-stable (2.0版本维护)
└─ 3-0-stable (3.0版本维护)

新功能 → main → 下个版本
Bug修复 → main → cherry-pick到stable分支
```

**实施**：

```bash
# 开发新功能（进入下个版本）
git checkout main
git checkout -b feature/new-feature
git commit -am "feat: new feature"
git checkout main
git merge feature/new-feature

# 修复bug（需要修复到旧版本）
git checkout main
git checkout -b fix/critical-bug
git commit -am "fix: critical bug"
git checkout main
git merge fix/critical-bug

# 应用到稳定分支
git checkout 2-0-stable
git cherry-pick <commit-id>
git push origin 2-0-stable

git checkout 1-0-stable
git cherry-pick <commit-id>
git push origin 1-0-stable
```

### Merge Request工作流

```bash
# 1. 创建分支
git checkout -b feature/improvement main

# 2. 提交修改
git commit -am "feat: add improvement"

# 3. 推送
git push origin feature/improvement

# 4. 在GitLab创建Merge Request
#    - 源分支：feature/improvement
#    - 目标分支：main
#    - 分配审查者
#    - 关联Issue

# 5. CI/CD自动运行测试

# 6. 代码审查

# 7. 解决冲突（如果有）
git checkout feature/improvement
git merge main
# 解决冲突
git push origin feature/improvement

# 8. 合并后自动删除源分支
```

### GitLab Flow优缺点

**优点**：
✅ 灵活适应不同场景  
✅ 支持环境部署  
✅ 支持版本维护  
✅ 简单清晰  
✅ 集成CI/CD

**缺点**：
❌ 需要理解多种模式  
❌ 环境分支需要严格管理  
❌ cherry-pick可能出错

**适用场景**：
- 需要多环境部署
- 需要维护多版本
- 使用GitLab CI/CD
- 中型到大型团队

## 工作流对比

| 特性 | Git Flow | GitHub Flow | GitLab Flow |
|------|----------|-------------|-------------|
| 复杂度 | 高 | 低 | 中 |
| 分支数量 | 多 | 少 | 中 |
| 发布周期 | 定期发布 | 持续部署 | 灵活 |
| 学习曲线 | 陡 | 平缓 | 中等 |
| 版本维护 | ✅ | ❌ | ✅ |
| 持续部署 | ❌ | ✅ | ✅ |
| 适合团队 | 大型 | 小中型 | 中大型 |
| 适合项目 | 软件产品 | Web应用 | 灵活 |

## 选择合适的工作流

### 决策树

```
你的项目是？
├─ 需要维护多个版本的软件产品
│  └─ 选择：Git Flow
│
├─ 持续部署的Web应用
│  └─ 选择：GitHub Flow
│
├─ 需要多环境部署
│  └─ 选择：GitLab Flow (环境分支模式)
│
└─ 需要维护旧版本又要持续迭代
   └─ 选择：GitLab Flow (发布分支模式)
```

### 考虑因素

1. **团队规模**
   - 小团队（<10人）：GitHub Flow
   - 中型团队（10-50人）：GitLab Flow
   - 大型团队（>50人）：Git Flow

2. **发布频率**
   - 每天多次：GitHub Flow
   - 每周/每两周：GitLab Flow
   - 每月/每季度：Git Flow

3. **产品类型**
   - SaaS产品：GitHub Flow
   - 移动应用：Git Flow
   - 企业软件：Git Flow
   - 开源项目：GitHub Flow / GitLab Flow

4. **版本要求**
   - 只维护最新版本：GitHub Flow
   - 需要维护多版本：Git Flow / GitLab Flow

5. **部署环境**
   - 单一生产环境：GitHub Flow
   - 多环境（dev/staging/prod）：GitLab Flow

## 总结

本节学习了三种主流Git工作流：

✅ **Git Flow**：完整严格，适合定期发布  
✅ **GitHub Flow**：简单高效，适合持续部署  
✅ **GitLab Flow**：灵活实用，适合多种场景

**关键点**：
- 没有完美的工作流，只有最适合的
- 工作流应该服务于团队，而不是束缚团队
- 可以根据项目需求调整和定制
- 保持简单，逐步演进

## 下一步

了解了主流工作流后，下一节我们将继续深入学习更多工作流实践技巧。

---

[← 上一节：Git远程分支管理](07-git-remote-branch-management.md) | [返回目录](../../README.md) | [下一节：Git工作流实践02 →](09-git-workflow-practice-02.md)
