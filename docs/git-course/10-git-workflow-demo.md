# 10. Git工作流实践演示

## 目录
- [项目场景设定](#项目场景设定)
- [完整开发流程演示](#完整开发流程演示)
- [团队协作场景](#团队协作场景)
- [冲突解决实战](#冲突解决实战)
- [版本发布流程](#版本发布流程)
- [紧急修复流程](#紧急修复流程)
- [综合案例](#综合案例)

## 项目场景设定

### 项目背景

我们将演示一个在线商城项目的开发流程：

```
项目：E-Commerce Web应用
团队：5人开发团队
工作流：GitHub Flow
技术栈：Node.js + React
```

### 团队成员

```
- Alice：前端开发（你）
- Bob：后端开发
- Carol：全栈开发
- Dave：测试工程师
- Eve：项目经理/代码审查者
```

### 仓库结构

```
e-commerce/
├── frontend/          # React前端
├── backend/           # Node.js后端
├── docs/              # 文档
├── tests/             # 测试
└── README.md
```

## 完整开发流程演示

### 任务：实现用户购物车功能

#### 第一天：准备工作

```bash
# Alice开始工作

# 1. 克隆项目（首次）
git clone git@github.com:team/e-commerce.git
cd e-commerce

# 2. 查看项目结构
ls -la
cat README.md

# 3. 安装依赖
npm install

# 4. 运行测试确保环境正常
npm test

# 5. 查看当前分支和远程分支
git branch -a
git remote -v

# 6. 同步最新代码
git checkout main
git pull origin main

# 7. 查看最近的提交
git log --oneline -10
```

#### 创建功能分支

```bash
# 8. 为购物车功能创建分支
git checkout -b feature/shopping-cart

# 9. 推送分支到远程（开始协作）
git push -u origin feature/shopping-cart

# 10. 在GitHub上创建Draft PR
# 标题：[WIP] feat: implement shopping cart
# 描述：实现购物车基本功能
# 标记为Draft（进行中）
```

#### 开发购物车组件

```bash
# 11. 创建购物车组件
mkdir -p frontend/src/components/Cart
touch frontend/src/components/Cart/Cart.jsx
touch frontend/src/components/Cart/Cart.css
touch frontend/src/components/Cart/Cart.test.js

# 12. 编写购物车组件代码
cat > frontend/src/components/Cart/Cart.jsx << 'EOF'
import React, { useState } from 'react';
import './Cart.css';

function Cart({ items, onUpdateQuantity, onRemoveItem }) {
  const total = items.reduce((sum, item) => 
    sum + item.price * item.quantity, 0
  );

  return (
    <div className="cart">
      <h2>购物车</h2>
      {items.length === 0 ? (
        <p>购物车为空</p>
      ) : (
        <>
          <ul className="cart-items">
            {items.map(item => (
              <li key={item.id}>
                <span>{item.name}</span>
                <span>¥{item.price}</span>
                <input 
                  type="number" 
                  value={item.quantity}
                  onChange={(e) => onUpdateQuantity(item.id, e.target.value)}
                />
                <button onClick={() => onRemoveItem(item.id)}>删除</button>
              </li>
            ))}
          </ul>
          <div className="cart-total">
            <strong>总计：¥{total.toFixed(2)}</strong>
          </div>
        </>
      )}
    </div>
  );
}

export default Cart;
EOF

# 13. 第一次提交
git add frontend/src/components/Cart/
git commit -m "feat(cart): create cart component structure"
```

#### 添加样式和测试

```bash
# 14. 添加样式
cat > frontend/src/components/Cart/Cart.css << 'EOF'
.cart {
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.cart-items {
  list-style: none;
  padding: 0;
}

.cart-items li {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.cart-total {
  margin-top: 20px;
  text-align: right;
  font-size: 1.2em;
}
EOF

git add frontend/src/components/Cart/Cart.css
git commit -m "style(cart): add cart component styles"

# 15. 添加测试
cat > frontend/src/components/Cart/Cart.test.js << 'EOF'
import { render, screen, fireEvent } from '@testing-library/react';
import Cart from './Cart';

describe('Cart Component', () => {
  const mockItems = [
    { id: 1, name: '商品1', price: 100, quantity: 2 },
    { id: 2, name: '商品2', price: 200, quantity: 1 }
  ];

  test('renders empty cart message', () => {
    render(<Cart items={[]} onUpdateQuantity={() => {}} onRemoveItem={() => {}} />);
    expect(screen.getByText('购物车为空')).toBeInTheDocument();
  });

  test('renders cart items', () => {
    render(<Cart items={mockItems} onUpdateQuantity={() => {}} onRemoveItem={() => {}} />);
    expect(screen.getByText('商品1')).toBeInTheDocument();
    expect(screen.getByText('商品2')).toBeInTheDocument();
  });

  test('calculates total correctly', () => {
    render(<Cart items={mockItems} onUpdateQuantity={() => {}} onRemoveItem={() => {}} />);
    expect(screen.getByText('总计：¥400.00')).toBeInTheDocument();
  });

  test('calls onRemoveItem when delete is clicked', () => {
    const mockRemove = jest.fn();
    render(<Cart items={mockItems} onUpdateQuantity={() => {}} onRemoveItem={mockRemove} />);
    
    const deleteButtons = screen.getAllByText('删除');
    fireEvent.click(deleteButtons[0]);
    
    expect(mockRemove).toHaveBeenCalledWith(1);
  });
});
EOF

git add frontend/src/components/Cart/Cart.test.js
git commit -m "test(cart): add cart component tests"

# 16. 运行测试
npm test

# 17. 推送到远程
git push origin feature/shopping-cart
```

#### 第二天：继续开发

```bash
# 18. 早上第一件事：同步main分支的更新
git checkout main
git pull origin main

# 19. 切回功能分支
git checkout feature/shopping-cart

# 20. 合并main的更新到功能分支
git merge main
# 或使用rebase保持线性历史
# git rebase main

# 21. 如果有冲突，解决后继续
# git add <resolved-files>
# git commit -m "merge: resolve conflicts with main"

# 22. 集成到应用中
cat > frontend/src/App.js << 'EOF'
import React, { useState } from 'react';
import Cart from './components/Cart/Cart';
import './App.css';

function App() {
  const [cartItems, setCartItems] = useState([
    { id: 1, name: 'MacBook Pro', price: 15999, quantity: 1 },
    { id: 2, name: 'iPhone 14', price: 6999, quantity: 2 }
  ]);

  const handleUpdateQuantity = (id, quantity) => {
    setCartItems(items =>
      items.map(item =>
        item.id === id ? { ...item, quantity: parseInt(quantity) || 0 } : item
      )
    );
  };

  const handleRemoveItem = (id) => {
    setCartItems(items => items.filter(item => item.id !== id));
  };

  return (
    <div className="App">
      <h1>我的商城</h1>
      <Cart 
        items={cartItems}
        onUpdateQuantity={handleUpdateQuantity}
        onRemoveItem={handleRemoveItem}
      />
    </div>
  );
}

export default App;
EOF

git add frontend/src/App.js
git commit -m "feat(cart): integrate cart component into app"

# 23. 推送更新
git push origin feature/shopping-cart
```

#### 准备代码审查

```bash
# 24. 更新文档
cat >> docs/components.md << 'EOF'

## Cart Component

购物车组件，用于显示和管理用户的购物车商品。

### Props

- `items`: 商品数组
- `onUpdateQuantity`: 更新数量的回调函数
- `onRemoveItem`: 删除商品的回调函数

### Usage

```jsx
<Cart 
  items={cartItems}
  onUpdateQuantity={(id, qty) => {...}}
  onRemoveItem={(id) => {...}}
/>
```
EOF

git add docs/components.md
git commit -m "docs(cart): add cart component documentation"

# 25. 最后检查
git status
npm run lint
npm test
npm run build

# 26. 推送所有更新
git push origin feature/shopping-cart

# 27. 在GitHub上将Draft PR标记为Ready for Review
# 更新PR描述：
```

**PR描述**：

```markdown
# feat: 实现购物车功能

## 变更说明
实现了基本的购物车组件，支持：
- 显示购物车商品列表
- 修改商品数量
- 删除商品
- 计算总价

## 变更类型
- [x] 新功能

## 测试
- [x] 添加了单元测试
- [x] 所有测试通过
- [x] 手动测试通过

## 截图
[添加购物车界面截图]

## 检查清单
- [x] 代码符合项目规范
- [x] 添加了文档
- [x] 添加了测试
- [x] 通过了CI检查

## 相关Issue
Closes #45

## 审查者
@Eve
```

## 团队协作场景

### Bob的后端API开发

同时，Bob在开发购物车API：

```bash
# Bob的工作流

# 1. 同步代码
git checkout main
git pull origin main

# 2. 创建API分支
git checkout -b feature/cart-api

# 3. 创建API端点
cat > backend/routes/cart.js << 'EOF'
const express = require('express');
const router = express.Router();

// 获取购物车
router.get('/cart/:userId', async (req, res) => {
  try {
    const cart = await Cart.findByUserId(req.params.userId);
    res.json(cart);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 添加商品到购物车
router.post('/cart/:userId/items', async (req, res) => {
  try {
    const { productId, quantity } = req.body;
    const cart = await Cart.addItem(req.params.userId, productId, quantity);
    res.json(cart);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 更新商品数量
router.put('/cart/:userId/items/:itemId', async (req, res) => {
  try {
    const { quantity } = req.body;
    const cart = await Cart.updateItem(req.params.userId, req.params.itemId, quantity);
    res.json(cart);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 删除商品
router.delete('/cart/:userId/items/:itemId', async (req, res) => {
  try {
    const cart = await Cart.removeItem(req.params.userId, req.params.itemId);
    res.json(cart);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
EOF

git add backend/routes/cart.js
git commit -m "feat(api): implement cart API endpoints"

# 4. 添加测试
# ... 编写API测试 ...
git commit -m "test(api): add cart API tests"

# 5. 推送并创建PR
git push -u origin feature/cart-api
```

### 集成前后端

```bash
# Alice集成Bob的API

# 1. Alice获取Bob的API分支
git fetch origin

# 2. 查看Bob的更改
git log origin/feature/cart-api

# 3. Alice更新购物车组件以使用API
cat > frontend/src/api/cart.js << 'EOF'
const API_BASE = '/api';

export async function fetchCart(userId) {
  const response = await fetch(`${API_BASE}/cart/${userId}`);
  return response.json();
}

export async function addToCart(userId, productId, quantity) {
  const response = await fetch(`${API_BASE}/cart/${userId}/items`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ productId, quantity })
  });
  return response.json();
}

export async function updateCartItem(userId, itemId, quantity) {
  const response = await fetch(`${API_BASE}/cart/${userId}/items/${itemId}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ quantity })
  });
  return response.json();
}

export async function removeCartItem(userId, itemId) {
  const response = await fetch(`${API_BASE}/cart/${userId}/items/${itemId}`, {
    method: 'DELETE'
  });
  return response.json();
}
EOF

git add frontend/src/api/cart.js
git commit -m "feat(cart): integrate with backend API"
git push origin feature/shopping-cart
```

## 冲突解决实战

### 场景：Alice和Carol同时修改了App.js

```bash
# Alice的修改
# feature/shopping-cart 分支

# Carol的修改
# feature/product-list 分支

# Carol先合并到main
git checkout main
git merge feature/product-list
git push origin main

# Alice尝试合并时发现冲突
git checkout feature/shopping-cart
git merge main

# 冲突！
# Auto-merging frontend/src/App.js
# CONFLICT (content): Merge conflict in frontend/src/App.js
```

**App.js冲突内容**：

```javascript
import React, { useState } from 'react';
<<<<<<< HEAD
import Cart from './components/Cart/Cart';
import './App.css';

function App() {
  const [cartItems, setCartItems] = useState([...]);
  // Alice的购物车代码
=======
import ProductList from './components/ProductList/ProductList';
import './App.css';

function App() {
  const [products, setProducts] = useState([...]);
  // Carol的产品列表代码
>>>>>>> main
```

**解决冲突**：

```bash
# 1. 查看冲突文件
git status

# 2. 编辑App.js，整合两者的代码
cat > frontend/src/App.js << 'EOF'
import React, { useState } from 'react';
import Cart from './components/Cart/Cart';
import ProductList from './components/ProductList/ProductList';
import './App.css';

function App() {
  const [products, setProducts] = useState([
    { id: 1, name: 'MacBook Pro', price: 15999 },
    { id: 2, name: 'iPhone 14', price: 6999 }
  ]);

  const [cartItems, setCartItems] = useState([]);

  const handleAddToCart = (product) => {
    const existingItem = cartItems.find(item => item.id === product.id);
    if (existingItem) {
      setCartItems(items =>
        items.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        )
      );
    } else {
      setCartItems([...cartItems, { ...product, quantity: 1 }]);
    }
  };

  const handleUpdateQuantity = (id, quantity) => {
    setCartItems(items =>
      items.map(item =>
        item.id === id ? { ...item, quantity: parseInt(quantity) || 0 } : item
      )
    );
  };

  const handleRemoveItem = (id) => {
    setCartItems(items => items.filter(item => item.id !== id));
  };

  return (
    <div className="App">
      <h1>我的商城</h1>
      <div className="container">
        <ProductList products={products} onAddToCart={handleAddToCart} />
        <Cart 
          items={cartItems}
          onUpdateQuantity={handleUpdateQuantity}
          onRemoveItem={handleRemoveItem}
        />
      </div>
    </div>
  );
}

export default App;
EOF

# 3. 标记为已解决
git add frontend/src/App.js

# 4. 完成合并
git commit -m "merge: integrate cart and product list features"

# 5. 测试确保一切正常
npm test
npm start  # 手动测试

# 6. 推送
git push origin feature/shopping-cart
```

## 版本发布流程

### 准备发布v1.0.0

```bash
# Eve（项目经理）准备发布

# 1. 确保所有功能都已合并到main
git checkout main
git pull origin main

# 2. 查看自上次发布以来的变更
git log v0.9.0..HEAD --oneline

# 3. 创建release分支（如果使用Git Flow）
# git checkout -b release/1.0.0

# 4. 更新版本号
npm version major  # 0.9.0 -> 1.0.0

# 5. 生成CHANGELOG
npx standard-version

# 6. 运行完整测试套件
npm run test:all
npm run test:e2e

# 7. 构建生产版本
npm run build:prod

# 8. 提交发布准备
git add .
git commit -m "chore: prepare release 1.0.0"

# 9. 创建标签
git tag -a v1.0.0 -m "Release version 1.0.0

Features:
- Shopping cart functionality
- Product listing
- User authentication

Bug Fixes:
- Fixed payment validation
- Resolved mobile layout issues"

# 10. 推送到远程
git push origin main --tags

# 11. 在GitHub上创建Release
# 上传构建产物
# 发布Release Notes

# 12. 部署到生产环境
npm run deploy:production
```

## 紧急修复流程

### 场景：生产环境发现严重bug

```bash
# 发现问题：购物车总价计算错误

# 1. Dave（测试）报告问题
# Issue #156: 购物车总价计算错误，未考虑折扣

# 2. 从main创建hotfix分支
git checkout main
git pull origin main
git checkout -b hotfix/cart-price-calculation

# 3. 修复bug
cat > frontend/src/components/Cart/Cart.jsx << 'EOF'
// ... 其他代码 ...

function Cart({ items, discount = 0, onUpdateQuantity, onRemoveItem }) {
  const subtotal = items.reduce((sum, item) => 
    sum + item.price * item.quantity, 0
  );
  
  const discountAmount = subtotal * (discount / 100);
  const total = subtotal - discountAmount;

  return (
    <div className="cart">
      {/* ... 商品列表 ... */}
      <div className="cart-total">
        <div>小计：¥{subtotal.toFixed(2)}</div>
        {discount > 0 && (
          <div>折扣({discount}%)：-¥{discountAmount.toFixed(2)}</div>
        )}
        <div><strong>总计：¥{total.toFixed(2)}</strong></div>
      </div>
    </div>
  );
}
EOF

git add frontend/src/components/Cart/Cart.jsx
git commit -m "fix(cart): correct total price calculation with discount

- Add discount parameter to Cart component
- Calculate and display discount amount
- Fix total price calculation

Fixes #156"

# 4. 更新测试
# ... 添加折扣测试用例 ...
git commit -m "test(cart): add discount calculation tests"

# 5. 运行测试
npm test

# 6. 推送并创建紧急PR
git push -u origin hotfix/cart-price-calculation

# 7. 快速代码审查后合并
git checkout main
git merge --no-ff hotfix/cart-price-calculation

# 8. 更新版本并打标签
npm version patch  # 1.0.0 -> 1.0.1
git tag -a v1.0.1 -m "Hotfix version 1.0.1

Fix:
- Correct cart total price calculation with discount

Fixes #156"

# 9. 推送并部署
git push origin main --tags
npm run deploy:production

# 10. 删除hotfix分支
git branch -d hotfix/cart-price-calculation
git push origin --delete hotfix/cart-price-calculation

# 11. 如果有develop分支，也要合并
# git checkout develop
# git merge hotfix/cart-price-calculation
```

## 综合案例

### 完整的冲刺周期（Sprint）

```bash
# Sprint 1: 两周开发周期

# 周一：Sprint Planning
# - 确定要完成的功能
# - 创建Issue和分配任务

# 每天：
git checkout main
git pull origin main
# 在功能分支上工作
# 频繁提交和推送
# 参与代码审查

# 周五：Sprint Review
git checkout main
git pull origin main
git log --since="1 week ago" --pretty=format:"%h - %an: %s"
# 演示完成的功能

# 实际示例：
```

**Sprint期间的Git活动**：

```bash
# Alice的一天
# 早上9:00
git checkout main && git pull origin main
git checkout feature/shopping-cart
git merge main

# 10:00-12:00: 编码
git add .
git commit -m "feat(cart): add item quantity validation"
git push origin feature/shopping-cart

# 12:00-13:00: 午休

# 13:00-15:00: 继续编码
git add .
git commit -m "refactor(cart): extract total calculation logic"
git push origin feature/shopping-cart

# 15:00-16:00: 代码审查Bob的PR
# 在GitHub上留下评论

# 16:00-17:00: 根据审查意见修改代码
git add .
git commit -m "refactor: address code review comments"
git push origin feature/shopping-cart

# 17:00-18:00: 更新PR，请求再次审查
# 准备明天的工作
```

## 最佳实践总结

### 日常工作检查清单

```markdown
□ 早上第一件事：同步main分支
□ 在功能分支上工作，不直接在main上修改
□ 频繁提交，每次提交只做一件事
□ 写清晰的提交信息
□ 定期推送到远程（至少每天一次）
□ 保持PR小而专注
□ 及时响应代码审查意见
□ 合并前确保测试通过
□ 合并后删除功能分支
□ 下班前推送所有更改
```

### Git命令速查

```bash
# 每天的标准流程
git checkout main && git pull origin main
git checkout -b feature/new-feature
# 编码...
git add .
git commit -m "feat: add new feature"
git push -u origin feature/new-feature
# 创建PR，等待审查
# 审查通过后合并
git checkout main && git pull origin main
git branch -d feature/new-feature
```

## 总结

本节通过完整的实战演示，展示了：

✅ 真实的开发工作流程  
✅ 团队协作的最佳实践  
✅ 冲突解决的实际操作  
✅ 版本发布的完整流程  
✅ 紧急修复的处理方法  
✅ 日常工作的标准流程

**关键要点**：

1. **频繁同步**：每天开始前同步main分支
2. **小步提交**：保持提交小而专注
3. **持续推送**：定期推送到远程
4. **及时沟通**：遇到问题及时与团队沟通
5. **代码审查**：认真对待每次代码审查
6. **测试先行**：推送前确保测试通过
7. **文档同步**：代码和文档同步更新

## 课程总结

恭喜你完成了Git课程的所有章节！

### 你已经学会了：

1. ✅ **基础概念**：集中式vs分布式版本控制
2. ✅ **环境搭建**：Git安装和配置
3. ✅ **基本操作**：克隆、提交、推送、拉取
4. ✅ **回退操作**：撤销各阶段的修改
5. ✅ **冲突解决**：处理代码冲突的方法
6. ✅ **分支管理**：本地和远程分支操作
7. ✅ **工作流程**：Git Flow、GitHub Flow、GitLab Flow
8. ✅ **团队协作**：代码审查、提交规范
9. ✅ **实战演练**：完整的项目开发流程

### 继续学习：

- 🚀 深入学习Git内部原理
- 🔧 掌握Git高级技巧（bisect、worktree等）
- 🤖 学习CI/CD自动化
- 📚 阅读团队的Git规范文档
- 💪 在实际项目中练习

### 推荐资源：

- [Pro Git](https://git-scm.com/book/zh/v2) - 官方权威指南
- [Learn Git Branching](https://learngitbranching.js.org/) - 交互式Git学习
- [Oh My Git!](https://ohmygit.org/) - Git游戏化学习
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf) - 速查表

**记住：熟能生巧，多实践才能真正掌握Git！**

---

[← 上一节：Git工作流实践02](09-git-workflow-practice-02.md) | [返回目录](../../README.md)
