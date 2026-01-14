# 02. Git客户端环境搭建

## 2.1 Git 下载与安装

### 2.1.1 Windows 系统

#### 下载 Git

1. 访问 Git 官方网站：https://git-scm.com/
2. 点击 "Download for Windows" 下载最新版本
3. 或者直接访问：https://git-scm.com/download/win

#### 安装步骤

1. **运行安装程序**：双击下载的 `.exe` 文件
2. **选择安装路径**：建议使用默认路径
3. **选择组件**：
   - ✅ Windows Explorer integration（Windows 资源管理器集成）
   - ✅ Git Bash Here
   - ✅ Git GUI Here
4. **选择默认编辑器**：推荐选择 Vim 或 VS Code
5. **调整 PATH 环境**：选择 "Git from the command line and also from 3rd-party software"
6. **选择 HTTPS 传输后端**：使用默认的 OpenSSL
7. **配置行尾转换**：选择 "Checkout Windows-style, commit Unix-style line endings"
8. **配置终端模拟器**：选择 "Use MinTTY"
9. **配置额外选项**：保持默认设置
10. **完成安装**

#### 验证安装

打开命令提示符（CMD）或 PowerShell，输入：

```bash
git --version
```

如果显示版本号，说明安装成功。

### 2.1.2 macOS 系统

#### 方法一：使用 Homebrew（推荐）

```bash
# 安装 Homebrew（如果还没有安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装 Git
brew install git
```

#### 方法二：使用安装包

1. 访问：https://git-scm.com/download/mac
2. 下载 `.dmg` 安装包
3. 双击安装

#### 方法三：使用 Xcode Command Line Tools

```bash
xcode-select --install
```

#### 验证安装

```bash
git --version
```

### 2.1.3 Linux 系统

#### Debian/Ubuntu

```bash
sudo apt-get update
sudo apt-get install git
```

#### Fedora

```bash
sudo dnf install git
```

#### CentOS/RHEL

```bash
sudo yum install git
```

#### Arch Linux

```bash
sudo pacman -S git
```

#### 验证安装

```bash
git --version
```

## 2.2 Git 基本配置

### 2.2.1 配置用户信息

Git 需要知道你是谁，这样在提交代码时可以记录作者信息。

```bash
# 配置用户名
git config --global user.name "你的名字"

# 配置邮箱
git config --global user.email "your.email@example.com"
```

**注意**：
- `--global` 参数表示全局配置，对当前用户所有仓库生效
- 可以省略 `--global` 为单个仓库配置不同的用户信息

### 2.2.2 查看配置

```bash
# 查看所有配置
git config --list

# 查看特定配置
git config user.name
git config user.email

# 查看配置及其来源
git config --list --show-origin
```

### 2.2.3 配置默认编辑器

```bash
# 配置为 Vim
git config --global core.editor vim

# 配置为 VS Code
git config --global core.editor "code --wait"

# 配置为 Sublime Text
git config --global core.editor "subl -n -w"

# 配置为 Notepad++（Windows）
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

### 2.2.4 配置行尾符

不同操作系统使用不同的行尾符：
- Windows：CRLF（`\r\n`）
- Linux/Mac：LF（`\n`）

```bash
# Windows 系统推荐配置
git config --global core.autocrlf true

# Mac/Linux 系统推荐配置
git config --global core.autocrlf input
```

### 2.2.5 配置默认分支名称

Git 2.28 版本之后，可以配置默认分支名称：

```bash
# 将默认分支名称设置为 main
git config --global init.defaultBranch main
```

### 2.2.6 配置别名

为常用命令设置别名，提高效率：

```bash
# 配置常用别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

使用别名：

```bash
git st        # 等同于 git status
git co main   # 等同于 git checkout main
git br        # 等同于 git branch
git lg        # 显示美化的提交日志
```

## 2.3 SSH 密钥配置

### 2.3.1 为什么需要 SSH 密钥

使用 SSH 密钥可以：
- 无需每次输入用户名和密码
- 更安全的身份验证方式
- 支持更高级的安全特性

### 2.3.2 生成 SSH 密钥

```bash
# 生成 SSH 密钥对
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# 或使用更安全的 ed25519 算法
ssh-keygen -t ed25519 -C "your.email@example.com"
```

按提示操作：
1. 选择保存位置（默认 `~/.ssh/id_rsa`，直接回车）
2. 设置密码（可选，直接回车表示不设置）

### 2.3.3 查看公钥

```bash
# 查看公钥内容
cat ~/.ssh/id_rsa.pub

# 或者（如果使用 ed25519）
cat ~/.ssh/id_ed25519.pub
```

### 2.3.4 添加 SSH 密钥到 GitHub

1. 复制公钥内容
2. 登录 GitHub
3. 点击右上角头像 → Settings
4. 左侧菜单选择 "SSH and GPG keys"
5. 点击 "New SSH key"
6. 填写 Title（如：My Laptop）
7. 粘贴公钥到 Key 文本框
8. 点击 "Add SSH key"

### 2.3.5 添加 SSH 密钥到 GitLab

1. 登录 GitLab
2. 点击右上角头像 → Settings
3. 左侧菜单选择 "SSH Keys"
4. 粘贴公钥到 Key 文本框
5. 设置过期时间（可选）
6. 点击 "Add key"

### 2.3.6 测试 SSH 连接

```bash
# 测试 GitHub 连接
ssh -T git@github.com

# 测试 GitLab 连接
ssh -T git@gitlab.com
```

成功的话会看到欢迎消息。

## 2.4 Git 客户端工具

### 2.4.1 命令行工具

**Git Bash（Windows）**
- 随 Git for Windows 安装
- 提供类 Unix 的命令行环境

**终端（Mac/Linux）**
- 系统自带
- 可以使用 iTerm2（Mac）、Terminator（Linux）等增强终端

### 2.4.2 图形界面工具

**Git GUI（内置）**
- 随 Git 安装
- 基础的图形界面
- 命令启动：`git gui`

**gitk（内置）**
- 随 Git 安装
- 查看提交历史的图形工具
- 命令启动：`gitk`

**第三方图形工具**

1. **SourceTree**
   - 免费
   - 支持 Windows 和 Mac
   - 功能强大，界面友好
   - 官网：https://www.sourcetreeapp.com/

2. **GitHub Desktop**
   - 免费
   - 支持 Windows 和 Mac
   - 简洁易用
   - 官网：https://desktop.github.com/

3. **GitKraken**
   - 免费版和付费版
   - 跨平台
   - 界面炫酷
   - 官网：https://www.gitkraken.com/

4. **TortoiseGit（Windows）**
   - 免费
   - 集成到 Windows 资源管理器
   - 官网：https://tortoisegit.org/

5. **Tower**
   - 付费
   - 支持 Windows 和 Mac
   - 功能强大
   - 官网：https://www.git-tower.com/

### 2.4.3 IDE 集成

大多数现代 IDE 都内置了 Git 支持：

- **Visual Studio Code**：内置 Git 支持，插件丰富
- **IntelliJ IDEA**：强大的 Git 集成
- **Eclipse**：通过 EGit 插件支持
- **Xcode**：内置 Git 支持
- **Android Studio**：基于 IntelliJ，Git 支持完善

## 2.5 配置文件位置

### 2.5.1 配置文件优先级

Git 有三个级别的配置：

1. **系统级配置**（`--system`）
   - 位置：`/etc/gitconfig`（Linux/Mac）或 `C:\Program Files\Git\etc\gitconfig`（Windows）
   - 对所有用户生效

2. **全局配置**（`--global`）
   - 位置：`~/.gitconfig` 或 `~/.config/git/config`
   - 对当前用户所有仓库生效

3. **仓库配置**（`--local`）
   - 位置：`.git/config`（项目目录下）
   - 仅对当前仓库生效

优先级：仓库配置 > 全局配置> 系统配置

### 2.5.2 编辑配置文件

```bash
# 编辑全局配置
git config --global --edit

# 编辑仓库配置
git config --local --edit

# 编辑系统配置
git config --system --edit
```

## 2.6 常见问题

### 2.6.1 中文文件名显示为数字

```bash
git config --global core.quotepath false
```

### 2.6.2 解决 Git 命令行中文乱码

**Git Bash（Windows）**

```bash
# 设置 Git Bash 显示中文
git config --global core.quotepath false
git config --global gui.encoding utf-8
git config --global i18n.commit.encoding utf-8
git config --global i18n.logoutputencoding utf-8

# 设置环境变量
export LESSCHARSET=utf-8
```

### 2.6.3 配置 Git 忽略文件权限变化

```bash
git config --global core.filemode false
```

## 2.7 验证环境配置

完成所有配置后，运行以下命令验证：

```bash
# 查看 Git 版本
git --version

# 查看用户配置
git config user.name
git config user.email

# 查看所有配置
git config --list

# 测试 SSH（如果配置了）
ssh -T git@github.com
```

## 2.8 总结

- Git 支持 Windows、macOS 和 Linux 多个平台
- 安装后需要配置用户名和邮箱
- SSH 密钥配置可以免密访问远程仓库
- 可以使用命令行或图形界面工具
- 合理配置别名和选项可以提高工作效率

---

**上一章节**：[01. 集中式仓库管理和分布式仓库管理](./01-集中式仓库管理和分布式仓库管理.md)

**下一章节**：[03. Git常用的代码拉取-修改-提交-推送命令](./03-git常用的代码拉取-修改-提交-推送命令.md)
