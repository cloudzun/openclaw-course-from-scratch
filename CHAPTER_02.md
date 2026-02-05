# 第二章：安装与配置

## 2.1 环境准备

在安装OpenClaw之前，您需要确保系统满足以下最低要求，并安装必要的依赖项。

### 系统要求

**操作系统**：
- Linux (推荐 Ubuntu 20.04+, Debian 11+, CentOS 8+)
- macOS 10.15+
- Windows 10/11 (通过WSL2)

**硬件要求**：
- CPU: 至少双核处理器
- 内存: 最低 4GB RAM，推荐 8GB 或以上
- 存储: 至少 2GB 可用空间
- 网络: 稳定的互联网连接

### 依赖软件

在开始安装之前，请确保您的系统已安装以下软件：

**Node.js** (版本 18.0 或更高)
```bash
# 检查当前版本
node --version

# 如果未安装或版本过低，请升级
# 使用官方安装包或 nvm (Node Version Manager)
```

**Git** (版本 2.0 或更高)
```bash
# 检查是否已安装
git --version

# 在 Ubuntu/Debian 上安装
sudo apt update && sudo apt install git

# 在 CentOS/RHEL 上安装
sudo yum install git

# 在 macOS 上安装
brew install git
```

**npm** (随 Node.js 自动安装)
```bash
# 检查版本
npm --version
```

### 环境变量配置

在安装前，建议设置以下环境变量：

```bash
# 设置全局 npm 安装路径（可选但推荐）
export NPM_CONFIG_PREFIX=~/.npm-global
export PATH=$PATH:~/.npm-global/bin

# 将上述设置添加到 ~/.bashrc 或 ~/.zshrc 以永久生效
echo 'export NPM_CONFIG_PREFIX=~/.npm-global' >> ~/.bashrc
echo 'export PATH=$PATH:~/.npm-global/bin' >> ~/.bashrc
source ~/.bashrc
```

## 2.2 安装OpenClaw

OpenClaw可以通过多种方式进行安装，我们推荐使用npm包管理器进行安装。

### 方法一：使用npm全局安装（推荐）

```bash
# 全局安装OpenClaw
npm install -g @moltbot/clawd

# 验证安装
openclaw --version
```

### 方法二：从源码安装

如果您希望从最新源码安装或进行开发，可以使用以下步骤：

```bash
# 克隆源码仓库
git clone https://github.com/moltbot/moltbot.git
cd moltbot

# 安装依赖
npm install

# 构建项目
npm run build

# 安装到全局
npm install -g
```

### 方法三：使用Docker安装

对于喜欢容器化部署的用户，可以使用Docker方式安装：

```bash
# 拉取最新镜像
docker pull moltbot/openclaw:latest

# 创建并运行容器
docker run -d \
  --name openclaw \
  -v ~/.openclaw:/home/node/.openclaw \
  -v /var/run/docker.sock:/var/run/docker.sock \
  moltbot/openclaw:latest
```

## 2.3 初始配置

安装完成后，需要进行初始配置才能使用OpenClaw。

### 初始化命令

```bash
# 初始化OpenClaw配置
openclaw init
```

这个命令会在 `~/.openclaw` 目录下创建必要的配置文件和目录结构。

### 配置文件结构

初始化后，您将在 `~/.openclaw` 目录下看到以下结构：

```
~/.openclaw/
├── config.json          # 主配置文件
├── AGENTS.md           # 代理配置
├── SOUL.md             # 人格设定
├── USER.md             # 用户信息
├── TOOLS.md            # 工具配置
├── MEMORY.md           # 长期记忆
├── HEARTBEAT.md        # 心跳任务
├── memory/             # 每日记忆文件
│   └── YYYY-MM-DD.md
└── skills/             # 自定义技能
```

### 配置API密钥

OpenClaw需要访问大语言模型API，您需要配置相应的密钥：

```bash
# 设置OpenAI API密钥（如果使用OpenAI模型）
export OPENAI_API_KEY=your_openai_api_key_here

# 或设置其他提供商的API密钥
export QWEN_API_KEY=your_qwen_api_key_here  # 通义千问
export CLAUDE_API_KEY=your_claude_api_key_here  # Claude
```

建议将这些环境变量添加到您的shell配置文件中（如 `~/.bashrc` 或 `~/.zshrc`）以确保每次启动都能加载。

## 2.4 启动和验证

完成配置后，您可以启动OpenClaw并验证安装是否成功。

### 启动OpenClaw服务

```bash
# 启动OpenClaw网关服务
openclaw gateway start

# 检查服务状态
openclaw gateway status
```

### 验证安装

```bash
# 检查OpenClaw版本
openclaw --version

# 检查网关状态
openclaw gateway status

# 运行简单测试
openclaw test
```

### 基本功能测试

让我们运行一些基本命令来验证OpenClaw是否正常工作：

```bash
# 检查可用工具
openclaw tools list

# 执行一个简单的文件读取操作
echo "Hello, OpenClaw!" > ~/test_file.txt
openclaw exec "cat ~/test_file.txt"

# 执行网络搜索
openclaw web_search "OpenClaw documentation"
```

### 停止服务

当您需要停止OpenClaw服务时：

```bash
# 停止网关服务
openclaw gateway stop

# 重启服务
openclaw gateway restart
```

## 2.5 故障排除

### 常见问题及解决方案

1. **权限错误**
   - 确保您有足够的权限访问 `.openclaw` 目录
   - 如需要，使用 `chmod` 命令修改权限

2. **API密钥错误**
   - 检查API密钥是否正确设置
   - 确认环境变量已正确加载

3. **网络连接问题**
   - 检查防火墙设置
   - 确认网络连接正常

4. **依赖缺失**
   - 确保Node.js和npm版本符合要求
   - 重新运行安装命令

### 日志检查

如果遇到问题，可以检查日志文件：

```bash
# 查看OpenClaw日志
tail -f ~/.openclaw/logs/gateway.log

# 查看系统日志
journalctl -u openclaw -f  # 如果使用systemd
```

现在您已经成功安装并配置了OpenClaw系统，可以开始使用其强大功能了。下一章我们将深入探讨OpenClaw的核心概念。