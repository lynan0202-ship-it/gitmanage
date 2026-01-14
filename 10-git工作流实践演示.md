# 10. Git工作流实践演示

## 10.1 实战项目：开发一个 Web 应用

### 10.1.1 项目背景

我们将演示开发一个简单的待办事项（Todo）Web 应用，展示完整的 Git 工作流程。

**项目特点**：
- 团队规模：4人
- 技术栈：Node.js + React
- 工作流：GitHub Flow + 环境分支
- 部署环境：开发环境、生产环境

### 10.1.2 团队角色

- **Alice**：前端开发
- **Bob**：后端开发
- **Charlie**：全栈开发
- **David**：项目负责人/审查者

## 10.2 项目初始化

### 10.2.1 创建仓库（David）

```bash
# 1. 在 GitHub 上创建仓库 todo-app

# 2. 克隆到本地
git clone git@github.com:team/todo-app.git
cd todo-app

# 3. 初始化项目结构
mkdir -p client server docs
touch README.md .gitignore

# 4. 创建基础文件
cat > README.md << EOF
# Todo Application

一个简单的待办事项应用

## 技术栈
- Frontend: React
- Backend: Node.js + Express
- Database: MongoDB

## 开发指南
见 docs/development.md
EOF

cat > .gitignore << EOF
node_modules/
dist/
build/
.env
.DS_Store
*.log
EOF

# 5. 提交初始代码
git add .
git commit -m "chore: 初始化项目结构"
git push origin main

# 6. 创建 develop 分支（如果使用 Git Flow）
git checkout -b develop
git push origin develop
```

### 10.2.2 设置分支保护（David）

在 GitHub 上设置：

1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. 勾选：
   - ✅ Require pull request reviews before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Include administrators

## 10.3 Sprint 1：基础功能开发

### 10.3.1 任务分配

**Sprint 1 任务**：
- Task #1：搭建前端框架（Alice）
- Task #2：搭建后端 API（Bob）
- Task #3：设计数据库模型（Bob）
- Task #4：编写项目文档（Charlie）

### 10.3.2 Alice：搭建前端框架

```bash
# 1. 克隆仓库
git clone git@github.com:team/todo-app.git
cd todo-app

# 2. 创建功能分支
git checkout -b feature/setup-frontend

# 3. 初始化 React 项目
cd client
npx create-react-app .
cd ..

# 4. 提交更改
git add client/
git commit -m "feat: 初始化 React 项目结构

- 使用 create-react-app 搭建
- 配置基础目录结构
- 添加必要的依赖"

# 5. 推送分支
git push -u origin feature/setup-frontend

# 6. 创建 Pull Request
# 在 GitHub 上创建 PR: feature/setup-frontend → main
# Title: "feat: 搭建前端框架"
# Description: 
# - 初始化 React 项目
# - 配置基础结构
# - Closes #1

# 7. 等待审查...

# 8. 根据审查意见修改（如果有）
# ... 修改代码 ...
git add .
git commit -m "fix: 根据审查意见调整目录结构"
git push origin feature/setup-frontend

# 9. PR 合并后，更新本地 main 分支
git checkout main
git pull origin main
git branch -d feature/setup-frontend
```

### 10.3.3 Bob：搭建后端 API

```bash
# 1. 同步最新代码
git checkout main
git pull origin main

# 2. 创建功能分支
git checkout -b feature/setup-backend

# 3. 初始化后端项目
cd server
npm init -y
npm install express mongoose dotenv cors

# 4. 创建基础文件
mkdir -p src/models src/routes src/controllers
touch src/server.js src/config.js

# 5. 编写基础代码
cat > src/server.js << EOF
const express = require('express');
const cors = require('cors');
const mongoose = require('mongoose');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

// Routes will be added here

app.listen(PORT, () => {
  console.log(\`Server is running on port \${PORT}\`);
});
EOF

# 6. 提交
git add server/
git commit -m "feat: 搭建后端 API 框架

- 初始化 Express 应用
- 配置基础中间件
- 添加项目依赖

Closes #2"

# 7. 推送并创建 PR
git push -u origin feature/setup-backend

# 在 GitHub 上创建 PR
```

### 10.3.4 Charlie：编写项目文档

```bash
# 1. 同步并创建分支
git checkout main
git pull origin main
git checkout -b docs/development-guide

# 2. 创建文档
mkdir -p docs
cat > docs/development.md << EOF
# 开发指南

## 环境要求
- Node.js 16+
- MongoDB 4.4+
- npm 7+

## 本地开发

### 后端
\`\`\`bash
cd server
npm install
npm run dev
\`\`\`

### 前端
\`\`\`bash
cd client
npm install
npm start
\`\`\`

## Git 工作流程
1. 从 main 创建功能分支
2. 开发并提交
3. 创建 Pull Request
4. 代码审查
5. 合并到 main

## 提交规范
使用 Conventional Commits 格式：
- feat: 新功能
- fix: Bug修复
- docs: 文档
- refactor: 重构
EOF

# 3. 提交并创建 PR
git add docs/
git commit -m "docs: 添加开发指南文档"
git push -u origin docs/development-guide
```

### 10.3.5 David：代码审查和合并

```bash
# 审查 Alice 的 PR
# 1. 检出分支
git fetch origin
git checkout -b review-alice-pr origin/feature/setup-frontend

# 2. 查看代码
git log main..review-alice-pr
git diff main...review-alice-pr

# 3. 本地运行测试
cd client
npm install
npm start
# ... 测试功能 ...

# 4. 在 GitHub 上添加审查意见
# - "代码看起来不错！"
# - "建议：添加 ESLint 配置"

# 5. Alice 修改后，批准并合并 PR

# 同样审查 Bob 和 Charlie 的 PR
```

## 10.4 Sprint 2：核心功能开发

### 10.4.1 Alice：开发待办事项列表组件

```bash
# 1. 同步最新代码
git checkout main
git pull origin main

# 2. 创建功能分支
git checkout -b feature/todo-list-component

# 3. 开发组件
cd client/src
mkdir components
cat > components/TodoList.jsx << EOF
import React, { useState, useEffect } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [newTodo, setNewTodo] = useState('');

  useEffect(() => {
    // Fetch todos from API
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    try {
      const response = await fetch('http://localhost:5000/api/todos');
      const data = await response.json();
      setTodos(data);
    } catch (error) {
      console.error('Error fetching todos:', error);
    }
  };

  const addTodo = async () => {
    try {
      const response = await fetch('http://localhost:5000/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title: newTodo })
      });
      const data = await response.json();
      setTodos([...todos, data]);
      setNewTodo('');
    } catch (error) {
      console.error('Error adding todo:', error);
    }
  };

  return (
    <div>
      <h1>待办事项</h1>
      <input
        value={newTodo}
        onChange={(e) => setNewTodo(e.target.value)}
        placeholder="添加新的待办事项"
      />
      <button onClick={addTodo}>添加</button>
      <ul>
        {todos.map((todo) => (
          <li key={todo._id}>{todo.title}</li>
        ))}
      </ul>
    </div>
  );
}

export default TodoList;
EOF

# 4. 更新 App.js
cat > App.js << EOF
import React from 'react';
import TodoList from './components/TodoList';
import './App.css';

function App() {
  return (
    <div className="App">
      <TodoList />
    </div>
  );
}

export default App;
EOF

# 5. 提交
cd ../..
git add .
git commit -m "feat: 添加待办事项列表组件

- 实现待办事项显示
- 实现添加待办事项功能
- 集成后端 API

Closes #5"

# 6. 推送并创建 PR
git push -u origin feature/todo-list-component
```

### 10.4.2 Bob：开发后端 API

```bash
# 1. 同步并创建分支
git checkout main
git pull origin main
git checkout -b feature/todo-api

# 2. 创建 Todo 模型
cd server/src/models
cat > Todo.js << EOF
const mongoose = require('mongoose');

const todoSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  completed: {
    type: Boolean,
    default: false
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Todo', todoSchema);
EOF

# 3. 创建控制器
cd ../controllers
cat > todoController.js << EOF
const Todo = require('../models/Todo');

exports.getAllTodos = async (req, res) => {
  try {
    const todos = await Todo.find().sort({ createdAt: -1 });
    res.json(todos);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

exports.createTodo = async (req, res) => {
  const todo = new Todo({
    title: req.body.title
  });

  try {
    const newTodo = await todo.save();
    res.status(201).json(newTodo);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

exports.updateTodo = async (req, res) => {
  try {
    const todo = await Todo.findById(req.params.id);
    if (req.body.title) todo.title = req.body.title;
    if (req.body.completed !== undefined) todo.completed = req.body.completed;
    const updatedTodo = await todo.save();
    res.json(updatedTodo);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

exports.deleteTodo = async (req, res) => {
  try {
    await Todo.findByIdAndDelete(req.params.id);
    res.json({ message: 'Todo deleted' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};
EOF

# 4. 创建路由
cd ../routes
cat > todoRoutes.js << EOF
const express = require('express');
const router = express.Router();
const todoController = require('../controllers/todoController');

router.get('/', todoController.getAllTodos);
router.post('/', todoController.createTodo);
router.put('/:id', todoController.updateTodo);
router.delete('/:id', todoController.deleteTodo);

module.exports = router;
EOF

# 5. 更新 server.js
cd ..
cat > server.js << EOF
const express = require('express');
const cors = require('cors');
const mongoose = require('mongoose');
require('dotenv').config();

const todoRoutes = require('./routes/todoRoutes');

const app = express();
const PORT = process.env.PORT || 5000;

app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/todoapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

app.use('/api/todos', todoRoutes);

app.listen(PORT, () => {
  console.log(\`Server is running on port \${PORT}\`);
});
EOF

# 6. 添加测试
npm install --save-dev jest supertest
mkdir -p tests
cat > tests/todo.test.js << EOF
const request = require('supertest');
// Test code here
EOF

# 7. 提交
cd ../..
git add .
git commit -m "feat: 实现待办事项 CRUD API

- 创建 Todo 模型
- 实现增删改查接口
- 添加单元测试

Closes #6"

# 8. 推送并创建 PR
git push -u origin feature/todo-api
```

### 10.4.3 处理合并冲突

假设 Alice 和 Bob 的 PR 有冲突：

```bash
# Alice 的分支先合并了

# Bob 需要更新他的分支
git checkout feature/todo-api
git fetch origin
git merge origin/main

# 如果有冲突
# Auto-merging server/package.json
# CONFLICT (content): Merge conflict in server/package.json

# 解决冲突
vim server/package.json
# ... 手动解决冲突 ...

# 标记为已解决
git add server/package.json
git commit -m "merge: 解决与 main 分支的冲突"
git push origin feature/todo-api

# PR 更新，可以合并了
```

## 10.5 紧急修复生产环境 Bug

### 10.5.1 发现问题

```bash
# 生产环境报告：删除待办事项后页面崩溃
```

### 10.5.2 创建 Hotfix（Charlie）

```bash
# 1. 从 main 创建 hotfix 分支
git checkout main
git pull origin main
git checkout -b hotfix/delete-todo-crash

# 2. 定位并修复问题
cd client/src/components
vim TodoList.jsx

# 修复：在删除后正确更新状态
# 原代码：setTodos(todos.filter(t => t.id !== id));
# 修复后：setTodos(todos.filter(t => t._id !== id));

# 3. 添加测试防止回归
cd ../../tests
cat > TodoList.test.js << EOF
import { render, screen, fireEvent } from '@testing-library/react';
import TodoList from '../components/TodoList';

test('删除待办事项后不应崩溃', async () => {
  // Test code
});
EOF

# 4. 提交
cd ../..
git add .
git commit -m "fix: 修复删除待办事项后页面崩溃

问题：使用错误的属性名导致过滤失败
解决：使用正确的 _id 属性
添加：防止回归的测试用例

Fixes #15"

# 5. 推送并创建紧急 PR
git push -u origin hotfix/delete-todo-crash

# 在 GitHub 上创建 PR，标记为 "urgent"
```

### 10.5.3 快速审查和部署

```bash
# David 快速审查
git fetch origin
git checkout -b review-hotfix origin/hotfix/delete-todo-crash

# 验证修复
cd client
npm test
npm start
# ... 测试删除功能 ...

# 批准并合并

# 自动触发部署到生产环境
```

## 10.6 版本发布

### 10.6.1 准备发布 v1.0.0（David）

```bash
# 1. 创建 release 分支（如果使用 Git Flow）
git checkout main
git pull origin main
git checkout -b release/1.0.0

# 2. 更新版本号
cd client
npm version 1.0.0
cd ../server
npm version 1.0.0

# 3. 更新 CHANGELOG
cd ..
cat > CHANGELOG.md << EOF
# Changelog

## [1.0.0] - 2024-01-15

### Added
- 待办事项列表显示
- 添加待办事项功能
- 删除待办事项功能
- 标记待办事项完成功能

### Fixed
- 修复删除后页面崩溃问题

### Documentation
- 添加开发指南
- 更新 README
EOF

# 4. 提交
git add .
git commit -m "chore: 准备 1.0.0 版本发布"

# 5. 合并到 main 并打标签
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

# 6. 删除 release 分支
git branch -d release/1.0.0

# 7. 在 GitHub 上创建 Release
# - 使用 v1.0.0 标签
# - 添加 release notes
# - 上传构建产物（如果有）
```

## 10.7 持续集成配置

### 10.7.1 GitHub Actions 配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  test-client:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Install client dependencies
        run: |
          cd client
          npm ci
      
      - name: Run client tests
        run: |
          cd client
          npm test
      
      - name: Build client
        run: |
          cd client
          npm run build

  test-server:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:4.4
        ports:
          - 27017:27017
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Install server dependencies
        run: |
          cd server
          npm ci
      
      - name: Run server tests
        run: |
          cd server
          npm test
        env:
          MONGODB_URI: mongodb://localhost:27017/test
```

### 10.7.2 代码质量检查

```yaml
# .github/workflows/code-quality.yml
name: Code Quality

on:
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Lint client
        run: |
          cd client
          npm ci
          npm run lint
      
      - name: Lint server
        run: |
          cd server
          npm ci
          npm run lint
```

## 10.8 团队协作技巧

### 10.8.1 每日同步

```bash
# 每天开始工作前
git checkout main
git pull origin main

# 更新功能分支
git checkout feature/my-feature
git merge main
```

### 10.8.2 代码审查最佳实践

**作为作者**：
- 保持 PR 小而专注
- 提供清晰的描述
- 主动回应评论
- 及时更新代码

**作为审查者**：
- 及时审查（24小时内）
- 提供建设性反馈
- 区分阻塞性和建议性意见
- 批准前测试代码

### 10.8.3 解决争议

```bash
# 如果对某个实现有争议

# 1. 在 PR 中讨论
# 2. 如果无法达成一致，请项目负责人决定
# 3. 尊重最终决定
# 4. 文档化决策理由
```

## 10.9 总结

### 10.9.1 关键要点

- **明确的工作流程**：团队遵循统一的流程
- **小而频繁的提交**：便于审查和回滚
- **代码审查**：提高代码质量，知识共享
- **自动化测试**：保证代码质量
- **CI/CD**：自动化构建和部署
- **清晰的提交信息**：便于追踪历史

### 10.9.2 常见陷阱

- ❌ 分支存在太久
- ❌ PR 太大
- ❌ 提交信息不清晰
- ❌ 跳过代码审查
- ❌ 不写测试
- ❌ 不及时同步主分支

### 10.9.3 持续改进

- 定期回顾工作流程
- 收集团队反馈
- 优化开发体验
- 更新最佳实践文档

---

**上一章节**：[09. Git工作流实践02](./09-git工作流实践02.md)

**课程完结** 🎉

## 后续学习资源

- [Git 官方文档](https://git-scm.com/doc)
- [Pro Git 电子书](https://git-scm.com/book/zh/v2)
- [GitHub Guides](https://guides.github.com/)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
