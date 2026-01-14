# 02. Git客户端环境搭建

## 目录
- [Git安装](#git安装)
- [Git配置](#git配置)
- [SSH密钥配置](#ssh密钥配置)
- [验证安装](#验证安装)
- [常用Git工具](#常用git工具)

## Git安装

### Windows系统

#### 方法1：使用官方安装包

1. 访问 [Git官网](https://git-scm.com/downloads)
2. 下载Windows版本的安装包
3. 运行安装程序，推荐配置：
   - 选择默认编辑器（推荐VS Code或Vim）
   - 选择"Use Git from the Windows Command Prompt"
   - 选择"Checkout Windows-style, commit Unix-style line endings"
   - 选择"Use MinTTY"
   - 启用文件系统缓存和Git Credential Manager

#### 方法2：使用包管理器

```bash
# 使用Chocolatey
choco install git

# 使用Scoop
scoop install git
```

### macOS系统

#### 方法1：使用Homebrew（推荐）

```bash
# 安装Homebrew（如果还没有安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装Git
brew install git
```

#### 方法2：使用Xcode Command Line Tools

```bash
xcode-select --install
```

#### 方法3：下载官方安装包

访问 [Git官网](https://git-scm.com/downloads) 下载macOS版本

### Linux系统

#### Debian/Ubuntu

```bash
sudo apt update
sudo apt install git
```

#### Fedora

```bash
sudo dnf install git
```

#### CentOS/RHEL

```bash
# CentOS 7
sudo yum install git

# CentOS 8/RHEL 8
sudo dnf install git
```

#### Arch Linux

```bash
sudo pacman -S git
```

## Git配置

安装完成后，需要进行基本配置。Git的配置分为三个级别：

- **系统级**：`git config --system`（影响所有用户）
- **用户级**：`git config --global`（影响当前用户）
- **仓库级**：`git config --local`（仅影响当前仓库）

### 配置用户信息

这是必须的配置，因为每次提交都会使用这些信息：

```bash
# 配置用户名
git config --global user.name "你的姓名"

# 配置邮箱
git config --global user.email "your.email@example.com"
```

### 配置默认分支名称

```bash
# 将默认分支名称设置为main（Git 2.28+）
git config --global init.defaultBranch main
```

### 配置编辑器

```bash
# 使用VS Code
git config --global core.editor "code --wait"

# 使用Vim
git config --global core.editor vim

# 使用Nano
git config --global core.editor nano
```

### 配置差异比较工具

```bash
# 使用VS Code作为diff工具
git config --global diff.tool vscode
git config --global difftool.vscode.cmd "code --wait --diff $LOCAL $REMOTE"
```

### 配置合并工具

```bash
# 使用VS Code作为merge工具
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd "code --wait $MERGED"
```

### 配置别名

创建常用命令的快捷方式：

```bash
# 状态查看
git config --global alias.st status

# 检出分支
git config --global alias.co checkout

# 提交
git config --global alias.ci commit

# 分支
git config --global alias.br branch

# 日志（美化显示）
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# 撤销上一次提交
git config --global alias.undo "reset HEAD~1 --mixed"

# 最后一次提交
git config --global alias.last "log -1 HEAD"
```

### 其他实用配置

```bash
# 启用颜色显示
git config --global color.ui auto

# 设置默认推送方式（推送当前分支到同名远程分支）
git config --global push.default current

# 启用自动换行转换（Windows用户）
git config --global core.autocrlf true

# 启用自动换行转换（Mac/Linux用户）
git config --global core.autocrlf input

# 设置凭证缓存（避免重复输入密码）
# Mac
git config --global credential.helper osxkeychain
# Windows
git config --global credential.helper manager
# Linux
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=3600'
```

### 查看配置

```bash
# 查看所有配置
git config --list

# 查看特定配置
git config user.name
git config user.email

# 查看配置来源
git config --list --show-origin
```

## SSH密钥配置

使用SSH密钥可以避免每次推送代码时输入密码。

### 检查现有SSH密钥

```bash
# 查看是否已有SSH密钥
ls -al ~/.ssh
```

常见的SSH密钥文件名：
- `id_rsa.pub`
- `id_ecdsa.pub`
- `id_ed25519.pub`

### 生成新的SSH密钥

```bash
# 使用ed25519算法（推荐）
ssh-keygen -t ed25519 -C "your.email@example.com"

# 如果系统不支持ed25519，使用rsa
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```

按提示操作：
1. 选择保存位置（直接回车使用默认位置）
2. 输入密码短语（可选，但推荐设置）

### 添加SSH密钥到ssh-agent

```bash
# 启动ssh-agent
eval "$(ssh-agent -s)"

# 添加私钥
ssh-add ~/.ssh/id_ed25519
# 或
ssh-add ~/.ssh/id_rsa
```

### 添加SSH公钥到GitHub/GitLab

#### GitHub

1. 复制SSH公钥内容：

```bash
# Linux/Mac
cat ~/.ssh/id_ed25519.pub

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub

# Windows (PowerShell)
Get-Content ~/.ssh/id_ed25519.pub
```

2. 登录GitHub
3. 点击右上角头像 → Settings
4. 左侧菜单选择"SSH and GPG keys"
5. 点击"New SSH key"
6. 粘贴公钥内容，添加描述
7. 点击"Add SSH key"

#### GitLab

1. 复制SSH公钥内容（同上）
2. 登录GitLab
3. 点击右上角头像 → Preferences
4. 左侧菜单选择"SSH Keys"
5. 粘贴公钥内容
6. 点击"Add key"

### 测试SSH连接

```bash
# 测试GitHub连接
ssh -T git@github.com

# 测试GitLab连接
ssh -T git@gitlab.com
```

成功的响应示例：
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

## 验证安装

### 检查Git版本

```bash
git --version
```

应该显示类似：`git version 2.x.x`

### 验证配置

```bash
# 查看用户配置
git config user.name
git config user.email

# 查看所有配置
git config --list
```

### 创建测试仓库

```bash
# 创建测试目录
mkdir git-test
cd git-test

# 初始化Git仓库
git init

# 创建测试文件
echo "# Git Test" > README.md

# 添加到暂存区
git add README.md

# 提交
git commit -m "Initial commit"

# 查看日志
git log
```

## 常用Git工具

### 命令行工具

1. **Git Bash**（Windows）
   - 随Git for Windows安装
   - 提供类Unix的shell环境

2. **Oh My Zsh**（Mac/Linux）
   - Zsh配置框架
   - 提供Git主题和插件

### 图形界面工具

1. **Git GUI**
   - Git官方图形界面工具
   - 随Git一起安装

2. **GitKraken**
   - 跨平台Git客户端
   - 美观易用的界面
   - [https://www.gitkraken.com/](https://www.gitkraken.com/)

3. **SourceTree**
   - Atlassian出品
   - 免费，功能强大
   - [https://www.sourcetreeapp.com/](https://www.sourcetreeapp.com/)

4. **GitHub Desktop**
   - GitHub官方客户端
   - 简单易用
   - [https://desktop.github.com/](https://desktop.github.com/)

5. **Tower**
   - 商业Git客户端
   - Mac和Windows版本
   - [https://www.git-tower.com/](https://www.git-tower.com/)

### IDE集成

大多数现代IDE都内置Git支持：

- **Visual Studio Code**：优秀的Git集成
- **IntelliJ IDEA**：强大的版本控制工具
- **Eclipse**：EGit插件
- **Sublime Text**：通过插件支持

## 故障排除

### 问题1：git命令未找到

**解决方案**：
- 确认Git已正确安装
- 检查环境变量PATH中是否包含Git的安装路径
- 重启终端或计算机

### 问题2：SSL证书验证失败

**解决方案**：
```bash
# 临时禁用SSL验证（不推荐）
git config --global http.sslVerify false

# 更新CA证书（推荐）
# Windows: 更新Git for Windows
# Mac: brew upgrade git
# Linux: 更新系统证书包
```

### 问题3：无法推送到远程仓库

**解决方案**：
- 检查网络连接
- 确认SSH密钥已正确配置
- 验证远程仓库URL是否正确
- 检查是否有推送权限

## 总结

本节我们完成了：

✅ 在不同操作系统上安装Git  
✅ 配置Git用户信息和常用设置  
✅ 生成和配置SSH密钥  
✅ 验证Git安装和配置  
✅ 了解常用的Git工具

## 下一步

环境搭建完成后，下一节我们将学习Git的常用命令，包括代码的拉取、修改、提交和推送。

---

[← 上一节：集中式仓库管理和分布式仓库管理](01-centralized-vs-distributed-repo-management.md) | [返回目录](../../README.md) | [下一节：Git常用命令 →](03-git-common-commands.md)
