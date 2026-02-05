# 第四章：内置工具详解

OpenClaw的强大之处在于其丰富的内置工具集，这些工具使得系统能够执行各种实际任务，从简单的文件操作到复杂的网络交互。本章将详细介绍OpenClaw提供的各类内置工具。

## 4.1 文件操作工具（read/write/edit）

文件操作工具是OpenClaw中最基础也是最常用的工具集，允许系统读取、写入和编辑文件。

### 4.1.1 read工具

`read`工具用于读取文件内容，支持文本文件和图像文件。

**基本用法**：
```json
{
  "tool": "read",
  "arguments": {
    "file_path": "/path/to/file"
  }
}
```

**参数说明**：
- `file_path`: 要读取的文件路径（必需）
- `limit`: 最大读取行数（可选，默认无限制）
- `offset`: 从第几行开始读取（可选，默认从第1行开始）

**示例**：
```json
{
  "tool": "read",
  "arguments": {
    "file_path": "~/documents/notes.txt",
    "limit": 100
  }
}
```

**输出**：返回文件内容，如果是文本文件则返回内容字符串，如果是图像则作为附件发送。

**限制**：输出会被截断到2000行或50KB（以先达到的为准）。对于大文件，可以使用offset参数分段读取。

### 4.1.2 write工具

`write`工具用于创建或覆盖文件内容。

**基本用法**：
```json
{
  "tool": "write",
  "arguments": {
    "file_path": "/path/to/file",
    "content": "file content here"
  }
}
```

**参数说明**：
- `file_path`: 要写入的文件路径（必需）
- `content`: 要写入的文件内容（必需）

**示例**：
```json
{
  "tool": "write",
  "arguments": {
    "file_path": "~/myproject/todo.md",
    "content": "# To-Do List\n- Task 1\n- Task 2\n- Task 3"
  }
}
```

**注意**：该工具会创建必要的父目录，如果文件已存在则会被覆盖。

### 4.1.3 edit工具

`edit`工具用于精确编辑文件内容，通过替换特定文本片段来修改文件。

**基本用法**：
```json
{
  "tool": "edit",
  "arguments": {
    "file_path": "/path/to/file",
    "oldText": "text to be replaced",
    "newText": "replacement text"
  }
}
```

**参数说明**：
- `file_path`: 要编辑的文件路径（必需）
- `oldText`: 要查找并替换的精确文本（必需）
- `newText`: 替换的新文本（必需）

**示例**：
```json
{
  "tool": "edit",
  "arguments": {
    "file_path": "~/config/settings.json",
    "oldText": "\"debug\": false",
    "newText": "\"debug\": true"
  }
}
```

**注意**：`oldText`必须完全匹配（包括空白字符），否则操作会失败。

## 4.2 系统执行工具（exec/process）

系统执行工具允许OpenClaw运行shell命令和管理系统进程。

### 4.2.1 exec工具

`exec`工具用于执行shell命令，支持后台执行和TTY模式。

**基本用法**：
```json
{
  "tool": "exec",
  "arguments": {
    "command": "shell command here"
  }
}
```

**参数说明**：
- `command`: 要执行的shell命令（必需）
- `background`: 是否在后台运行（可选，默认false）
- `pty`: 是否使用伪终端（可选，默认false）
- `timeout`: 超时时间（秒，可选）
- `workdir`: 工作目录（可选，默认当前目录）
- `elevated`: 是否使用提升权限运行（可选）

**示例**：
```json
{
  "tool": "exec",
  "arguments": {
    "command": "ls -la ~/documents",
    "workdir": "~/projects"
  }
}
```

**高级用法**：
- 使用`background: true`可启动长时间运行的进程
- 使用`pty: true`可运行需要TTY的程序（如交互式CLI工具）
- 使用`process`工具管理后台进程

### 4.2.2 process工具

`process`工具用于管理后台执行的会话。

**基本用法**：
```json
{
  "tool": "process",
  "arguments": {
    "action": "list|poll|log|kill|write|send-keys|paste",
    "sessionId": "session-id"
  }
}
```

**参数说明**：
- `action`: 要执行的操作（必需）
  - `list`: 列出运行中的进程
  - `poll`: 获取进程状态
  - `log`: 获取进程日志
  - `kill`: 终止进程
  - `write`: 向进程写入数据
  - `send-keys`: 发送按键
  - `paste`: 粘贴文本

**示例**：
```json
{
  "tool": "process",
  "arguments": {
    "action": "list"
  }
}
```

## 4.3 网络工具（web_search/web_fetch/browser）

网络工具使OpenClaw能够访问互联网信息和进行网络交互。

### 4.3.1 web_search工具

`web_search`工具使用Brave Search API进行网络搜索。

**基本用法**：
```json
{
  "tool": "web_search",
  "arguments": {
    "query": "search query"
  }
}
```

**参数说明**：
- `query`: 搜索查询词（必需）
- `count`: 返回结果数量（1-10，默认10）
- `country`: 国家代码（如'US', 'DE'，默认'US'）
- `freshness`: 内容新鲜度（'pd': 24小时内, 'pw': 一周内等）
- `search_lang`: 搜索语言（如'de', 'en'）
- `ui_lang`: 界面语言

**示例**：
```json
{
  "tool": "web_search",
  "arguments": {
    "query": "OpenClaw documentation",
    "count": 5,
    "freshness": "pw"
  }
}
```

**输出**：返回搜索结果的标题、URL和摘要。

### 4.3.2 web_fetch工具

`web_fetch`工具用于获取网页内容并提取可读文本。

**基本用法**：
```json
{
  "tool": "web_fetch",
  "arguments": {
    "url": "https://example.com"
  }
}
```

**参数说明**：
- `url`: 要获取的网页URL（必需）
- `extract_mode`: 提取模式（"markdown"或"text"，默认"markdown"）
- `max_chars`: 最大返回字符数（可选）

**示例**：
```json
{
  "tool": "web_fetch",
  "arguments": {
    "url": "https://docs.molt.bot/introduction",
    "extract_mode": "markdown",
    "max_chars": 2000
  }
}
```

### 4.3.3 browser工具

`browser`工具提供完整的浏览器自动化功能。

**基本用法**：
```json
{
  "tool": "browser",
  "arguments": {
    "action": "open|snapshot|screenshot|navigate|act|..."
  }
}
```

**参数说明**：
- `action`: 浏览器操作类型
  - `status`: 检查浏览器状态
  - `start`: 启动浏览器
  - `open`: 打开网页
  - `snapshot`: 获取页面快照
  - `screenshot`: 截图
  - `navigate`: 导航到URL
  - `act`: 执行页面操作
- `profile`: 浏览器配置文件（"chrome"或"openclaw"）
- `targetId`: 目标标签页ID

**示例**：
```json
{
  "tool": "browser",
  "arguments": {
    "action": "open",
    "targetUrl": "https://www.example.com"
  }
}
```

## 4.4 消息工具（message）

消息工具用于通过各种渠道发送和管理消息。

**基本用法**：
```json
{
  "tool": "message",
  "arguments": {
    "action": "send|react|delete|...",
    "message": "message content"
  }
}
```

**参数说明**：
- `action`: 消息操作类型
  - `send`: 发送消息
  - `react`: 添加反应
  - `delete`: 删除消息
  - `edit`: 编辑消息
  - `poll`: 创建投票
  - `search`: 搜索消息
- `channel`: 目标频道
- `message`: 消息内容
- `target`: 目标用户或频道ID

**示例**：
```json
{
  "tool": "message",
  "arguments": {
    "action": "send",
    "channel": "discord",
    "message": "Hello from OpenClaw!",
    "target": "general-channel"
  }
}
```

## 4.5 记忆管理工具（memory_search/memory_get）

记忆管理工具用于检索和管理OpenClaw的记忆系统。

### 4.5.1 memory_search工具

`memory_search`工具用于语义搜索记忆文件。

**基本用法**：
```json
{
  "tool": "memory_search",
  "arguments": {
    "query": "search query"
  }
}
```

**参数说明**：
- `query`: 搜索查询（必需）
- `max_results`: 最大结果数（可选）
- `min_score`: 最小相似度分数（可选）

**示例**：
```json
{
  "tool": "memory_search",
  "arguments": {
    "query": "previous meeting notes about project X"
  }
}
```

### 4.5.2 memory_get工具

`memory_get`工具用于精确读取记忆文件的特定部分。

**基本用法**：
```json
{
  "tool": "memory_get",
  "arguments": {
    "path": "path/to/memory/file",
    "from": 10,
    "lines": 20
  }
}
```

**参数说明**：
- `path`: 记忆文件路径（必需）
- `from`: 开始行号（可选）
- `lines`: 读取行数（可选）

**示例**：
```json
{
  "tool": "memory_get",
  "arguments": {
    "path": "MEMORY.md",
    "from": 50,
    "lines": 10
  }
}
```

## 4.6 其他重要工具

### 4.6.1 tts工具

`text-to-speech`工具将文本转换为音频。

**基本用法**：
```json
{
  "tool": "tts",
  "arguments": {
    "text": "text to convert to speech"
  }
}
```

### 4.6.2 gateway工具

`gateway`工具用于管理系统服务。

**基本用法**：
```json
{
  "tool": "gateway",
  "arguments": {
    "action": "restart|config.get|config.apply|update.run"
  }
}
```

### 4.6.3 cron工具

`cron`工具用于管理定时任务。

**基本用法**：
```json
{
  "tool": "cron",
  "arguments": {
    "action": "status|list|add|update|remove|run"
  }
}
```

### 4.6.4 sessions工具

`sessions`相关工具用于管理会话：

- `sessions_list`: 列出会话
- `sessions_history`: 获取会话历史
- `sessions_send`: 发送消息到会话
- `sessions_spawn`: 创建子会话

这些工具构成了OpenClaw的核心功能集，使系统能够执行各种复杂的自动化任务。熟练掌握这些工具是有效使用OpenClaw的关键。