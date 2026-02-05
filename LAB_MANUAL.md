# 实验手册

## 实验环境要求
- Linux/MacOS系统
- Node.js v18+
- Git客户端
- Docker（可选，用于某些高级实验）
- 稳定的互联网连接

## 实验列表

### 实验一：OpenClaw环境搭建与配置
**目的**：安装和配置OpenClaw基础环境
**时长**：45分钟
**难度**：初级
**关联章节**：第2章 安装与配置

#### 实验步骤：
1. **检查系统环境和依赖**
   - 检查Node.js版本: `node --version` (需v18+)
   - 检查npm版本: `npm --version`
   - 检查Git安装: `git --version`
   - 确保有足够磁盘空间

2. **通过npm安装OpenClaw**
   - 全局安装: `npm install -g @moltbot/clawd`
   - 验证安装: `openclaw --version`
   - 检查可用命令: `openclaw --help`

3. **初始化OpenClaw配置**
   - 创建配置目录: `openclaw init`
   - 检查生成的配置文件: `ls -la ~/.openclaw/`
   - 验证配置结构是否完整

4. **设置API密钥和环境变量**
   - 编辑环境变量文件: `nano ~/.bashrc` 或 `nano ~/.zshrc`
   - 添加API密钥（根据使用的模型提供商）:
     ```
     export QWEN_API_KEY=your_qwen_api_key_here
     export OPENAI_API_KEY=your_openai_api_key_here
     ```
   - 重新加载环境变量: `source ~/.bashrc` 或 `source ~/.zshrc`

5. **启动服务并验证安装**
   - 启动Gateway: `openclaw gateway start`
   - 检查服务状态: `openclaw gateway status`
   - 等待服务完全启动（约30秒）

6. **验证基本功能**
   - 运行简单测试: `openclaw exec "echo 'Hello OpenClaw'"`

#### 预期结果：
- OpenClaw服务正常运行
- 基础配置文件正确生成
- 可通过命令行访问系统
- API密钥配置成功

#### 评估标准：
- [ ] 成功安装OpenClaw
- [ ] 服务启动无错误
- [ ] 配置文件正确生成
- [ ] 基本命令正常工作

#### 常见问题排查：
- 如果服务启动失败，检查端口占用: `netstat -tulpn | grep 3000`
- 检查API密钥是否正确设置: `echo $QWEN_API_KEY`
- 查看日志文件: `tail -f ~/.openclaw/logs/gateway.log`

---

### 实验二：核心概念验证
**目的**：理解和验证OpenClaw的核心组件功能
**时长**：45分钟
**难度**：初级
**关联章节**：第3章 核心概念

#### 实验步骤：
1. **检查Gateway状态和配置**
   - 查看Gateway状态: `openclaw gateway status`
   - 查看当前配置: `openclaw gateway config.get`
   - 验证端口监听: `ss -tlnp | grep 3000` 或 `netstat -tlnp | grep 3000`

2. **验证Agent系统功能**
   - 检查主Agent状态: `openclaw sessions_list`
   - 创建测试交互: 向OpenClaw发送简单消息测试响应
   - 检查Agent配置文件: `cat ~/.openclaw/AGENTS.md`
   - 查看人格设定: `cat ~/.openclaw/SOUL.md`

3. **测试Node管理（如果有分布式节点）**
   - 列出可用节点: `openclaw nodes action=status`
   - 检查节点连接状态: `openclaw nodes action=describe`
   - （如果没有额外节点，跳过此步骤）

4. **验证消息路由机制**
   - 发送测试消息并观察处理流程
   - 检查消息历史: `openclaw sessions_history sessionKey=main`
   - 验证消息是否正确路由到处理单元

5. **检查会话管理功能**
   - 列出当前会话: `openclaw sessions_list`
   - 检查会话状态和活动时间
   - 创建新会话（如果需要）: `openclaw sessions_spawn`

#### 预期结果：
- Gateway组件正常工作
- Agent响应请求正确
- 消息路由按预期工作
- 会话管理功能正常

#### 评估标准：
- [ ] Gateway状态显示正常
- [ ] Agent能正确响应消息
- [ ] 消息处理流程正常
- [ ] 会话管理功能正常

#### 常见问题排查：
- 如果Gateway未运行: `openclaw gateway start`
- 检查配置文件权限: `ls -la ~/.openclaw/`
- 重启服务后再次测试: `openclaw gateway restart`

---

### 实验三：内置工具实践
**目的**：熟练掌握OpenClaw的各种内置工具
**时长**：60分钟
**难度**：初级
**关联章节**：第4章 内置工具详解

#### 实验步骤：
1. **使用read工具读取文件和图像**
   - 创建测试文件: `echo "This is a test file for OpenClaw" > ~/test_read.txt`
   - 读取文件内容: `openclaw read file_path=~/test_read.txt`
   - 尝试读取大文件（使用limit和offset参数）
   - （如果系统中有图像文件）测试读取图像

2. **使用write工具创建和修改文件**
   - 创建新文件: `openclaw write file_path=~/claw_test.txt content="# My OpenClaw Test File\n\nThis file was created using OpenClaw's write tool.\n\nDate: $(date)"` 
   - 验证文件创建: `cat ~/claw_test.txt`
   - 创建多级目录文件: `openclaw write file_path=~/openclaw/experiments/demo.txt content="Demo content"`

3. **使用edit工具精确编辑文件内容**
   - 创建待编辑文件: `openclaw write file_path=~/edit_test.txt content="Line 1\nOld content to be replaced\nLine 3"`
   - 编辑文件内容: `openclaw edit file_path=~/edit_test.txt oldText="Old content to be replaced" newText="New updated content"`
   - 验证编辑结果: `cat ~/edit_test.txt`

4. **使用exec工具执行系统命令**
   - 执行简单命令: `openclaw exec command="ls -la ~/"`  
   - 执行带管道的命令: `openclaw exec command="ps aux | grep claw"`
   - 执行后台任务: `openclaw exec command="sleep 10 && echo 'Task completed' > ~/bg_task_result.txt" background=true`
   - 使用process工具管理后台任务

5. **使用web_search进行网络搜索**
   - 执行搜索: `openclaw web_search query="OpenClaw documentation"`
   - 尝试不同参数: `openclaw web_search query="AI agents" count=5 freshness=pw`
   - 验证搜索结果的相关性

6. **使用web_fetch获取网页内容**
   - 获取网页: `openclaw web_fetch url="https://docs.molt.bot/introduction"`
   - 限制内容长度: `openclaw web_fetch url="https://docs.molt.bot/" max_chars=500`
   - 验证内容提取质量

7. **使用browser工具进行浏览器自动化**
   - 检查浏览器状态: `openclaw browser action=status`
   - 启动浏览器（如果未启动）: `openclaw browser action=start`
   - 打开网页: `openclaw browser action=open targetUrl="https://www.example.com"`

8. **使用message工具发送消息**
   - 发送测试消息: `openclaw message action=send message="Test message from lab exercise"`
   - （如果配置了多渠道）测试不同渠道发送

9. **使用memory工具管理记忆系统**
   - 搜索记忆: `openclaw memory_search query="initial setup"`
   - 读取记忆文件: `openclaw memory_get path=MEMORY.md lines=10`

#### 预期结果：
- 成功执行所有基本工具操作
- 理解各工具的参数和返回值
- 掌握工具组合使用方法
- 理解工具的限制和最佳实践

#### 评估标准：
- [ ] 所有工具操作成功
- [ ] 输出结果符合预期
- [ ] 错误处理正确
- [ ] 理解工具限制

#### 常见问题排查：
- 如果exec工具被限制，检查安全配置: `openclaw gateway config.get`
- 如果web工具无法访问，检查网络连接和API密钥
- 如果工具响应慢，检查系统资源使用情况

---

### 实验四：高级功能应用
**目的**：实践OpenClaw的高级功能
**时长**：75分钟
**难度**：中级
**关联章节**：第5章 高级功能

#### 实验步骤：
1. **创建自定义技能**
   - **创建技能目录结构**:
     ```bash
     mkdir -p ~/.openclaw/skills/hello-world/{scripts,templates,resources}
     ```
   
   - **编写技能描述文件**:
     ```bash
     cat > ~/.openclaw/skills/hello-world/SKILL.md << 'EOF'
# hello-world

## 描述
一个简单的问候技能，用于演示技能创建过程。

## 功能
- 根据时间生成个性化问候
- 支持多种语言的问候

## 使用方法
当用户提及"hello"、"hi"或"greeting"时，此技能将自动激活。
EOF
     ```
   
   - **实现技能逻辑**（创建脚本）:
     ```bash
     cat > ~/.openclaw/skills/hello-world/scripts/main.js << 'EOF'
// 简单的问候技能实现
function generateGreeting(userName, timeOfDay) {
  const greetings = {
    morning: ["Good morning", "早上好", "Buenos días"],
    afternoon: ["Good afternoon", "下午好", "Buenas tardes"],
    evening: ["Good evening", "晚上好", "Buenas noches"],
    night: ["Good night", "晚安", "Buenas noches"]
  };
  
  const selectedGreetings = greetings[timeOfDay] || greetings.morning;
  const randomGreeting = selectedGreetings[Math.floor(Math.random() * selectedGreetings.length)];
  
  return `${randomGreeting}, ${userName || 'friend'}! Welcome to OpenClaw.`;
}

// 导出函数供OpenClaw调用
module.exports = {
  generateGreeting
};
EOF
     ```
   
   - **注册技能到系统**:
     检查技能是否被识别: `openclaw agents_list` 或查看可用技能列表

2. **配置定时任务（cron）**
   - **创建定时任务定义**:
     ```json
     {
       "name": "hello-world-task",
       "schedule": {
         "kind": "every",
         "everyMs": 300000  // 每5分钟执行一次
       },
       "payload": {
         "kind": "agentTurn",
         "message": "Generate a friendly greeting message"
       },
       "sessionTarget": "isolated",
       "enabled": true
     }
     ```
   
   - **添加到cron系统**:
     ```bash
     openclaw cron action=add job='{"name":"hello-world-task","schedule":{"kind":"every","everyMs":300000},"payload":{"kind":"agentTurn","message":"Generate a friendly greeting message"},"sessionTarget":"isolated","enabled":true}'
     ```
   
   - **验证任务执行**:
     - 查看任务列表: `openclaw cron action=list`
     - 检查任务状态: `openclaw cron action=runs jobId=hello-world-task`
     - 手动触发任务: `openclaw cron action=run jobId=hello-world-task`

3. **实践多平台集成**
   - **检查当前平台配置**:
     查看当前配置的通信渠道: `openclaw gateway config.get | grep -A 20 channels`
   
   - **测试消息发送功能**:
     - 发送到默认渠道: `openclaw message action=send message="Multi-platform test message"`
     - （如果配置了多个平台）测试发送到特定平台
     - 测试不同类型的消息（文本、反应等）

4. **管理会话**
   - **创建和管理多个会话**:
     - 查看当前会话: `openclaw sessions_list`
     - 创建新会话: `openclaw sessions_spawn task="Handle customer inquiry" timeoutSeconds=300`
     - 检查新会话状态: `openclaw sessions_list`
   
   - **测试会话间通信**:
     - 获取会话历史: `openclaw sessions_history sessionKey=<session-id> limit=10`
     - 向特定会话发送消息: `openclaw sessions_send sessionKey=<target-session> message="Message from another session"`

#### 预期结果：
- 自定义技能成功注册和运行
- 定时任务按计划执行
- 多平台消息功能正常
- 会话管理功能正常

#### 评估标准：
- [ ] 技能成功加载和运行
- [ ] 定时任务正常执行
- [ ] 多平台功能正常
- [ ] 会话管理功能正常

#### 常见问题排查：
- 如果技能未被识别，检查目录结构和文件名
- 如果定时任务未执行，检查cron服务状态和任务配置
- 如果消息发送失败，检查平台配置和API密钥

---

### 实验五：安全配置与优化
**目的**：配置安全策略和系统优化
**时长**：60分钟
**难度**：中级
**关联章节**：第6章 安全与最佳实践

#### 实验步骤：
1. **配置权限管理**
   - **查看当前安全配置**: `openclaw gateway config.get | grep -A 30 security`
   
   - **设置工具访问权限**:
     创建安全配置文件修改，限制exec工具的命令:
     ```json
     {
       "security": {
         "exec": {
           "mode": "allowlist",
           "allowlist": [
             "ls", "cat", "echo", "grep", "find", "pwd", "whoami",
             "date", "ps", "top", "free", "df", "du"
           ],
           "denylist": [
             "rm", "mv", "cp", "chmod", "chown", "userdel", "usermod"
           ]
         }
       }
     }
     ```
   
   - **配置命令白名单/黑名单**:
     使用gateway工具更新配置（如果支持）或直接编辑配置文件:
     `nano ~/.openclaw/config.json`
   
   - **测试权限限制**:
     - 尝试执行允许的命令: `openclaw exec command="ls -la"`
     - 尝试执行被禁止的命令: `openclaw exec command="rm test.txt"` (应该被拒绝)

2. **实施安全配置**
   - **配置网络访问限制**:
     在配置文件中添加网络限制:
     ```json
     {
       "security": {
         "network": {
           "blocked_domains": [
             "malicious-site.com",
             "*.dangerous-domain.org"
           ],
           "allowed_endpoints": [
             "api.openai.com",
             "docs.molt.bot",
             "github.com"
           ]
         }
       }
     }
     ```
   
   - **设置资源限制**:
     配置执行限制:
     ```json
     {
       "security": {
         "execution": {
           "timeout_default": 30,
           "memory_limit": "512MB",
           "disk_quota": "1GB"
         }
       }
     }
     ```
   
   - **启用审计日志**:
     配置日志设置:
     ```json
     {
       "logging": {
         "audit_log_level": "info",
         "log_sensitive_data": false,
         "retention_days": 90
       }
     }
     ```

3. **性能优化**
   - **配置资源限制**:
     设置并发限制:
     ```json
     {
       "concurrency": {
         "max_concurrent_exec": 5,
         "max_concurrent_web_requests": 10,
         "queue_size": 100
       }
     }
     ```
   
   - **设置缓存策略**:
     配置内存和结果缓存:
     - 调整内存管理参数
     - 配置结果缓存时间
     - 设置缓存大小限制
   
   - **优化并发设置**:
     根据系统资源调整并发参数:
     - 检查系统资源: `free -h` 和 `htop`
     - 根据资源情况调整并发数

#### 预期结果：
- 权限配置正确生效
- 安全策略有效实施
- 系统性能得到优化
- 审计日志正常记录

#### 评估标准：
- [ ] 权限配置生效（禁止的命令被拒绝）
- [ ] 安全策略有效（网络限制等生效）
- [ ] 性能配置正确（资源使用在限制内）
- [ ] 日志正常记录（审计日志包含必要信息）

#### 常见问题排查：
- 如果配置更改不生效，重启OpenClaw服务: `openclaw gateway restart`
- 检查配置文件语法: `cat ~/.openclaw/config.json | python -m json.tool`
- 查看安全相关的日志: `tail -f ~/.openclaw/logs/gateway.log`

---

### 实验六：综合实战项目
**目的**：应用所学知识完成综合项目
**时长**：120分钟
**难度**：高级
**关联章节**：第7章 综合实战项目

#### 实验步骤：
1. **项目规划**
   - **分析需求**:
     - 确定项目目标：创建一个简单的系统监控助手
     - 列出所需功能：CPU/内存监控、磁盘使用情况、网络连通性检查
     - 确定输出格式：每日报告、异常告警
   
   - **设计系统架构**:
     - 数据收集模块：使用exec工具执行系统命令
     - 数据处理模块：分析收集的数据
     - 报告生成模块：使用write工具创建报告
     - 通知模块：使用message工具发送通知
   
   - **规划实现步骤**:
     - 第一步：实现数据收集功能
     - 第二步：实现数据分析功能
     - 第三步：实现报告生成功能
     - 第四步：实现通知发送功能
     - 第五步：整合并配置定时任务

2. **核心功能实现**
   - **实现数据收集模块**:
     ```bash
     # 收集CPU使用率
     openclaw exec command="top -bn1 | grep '%Cpu' | awk '{print $2}' | sed 's/%us,//'"
     
     # 收集内存使用率
     openclaw exec command="free -m | awk 'NR==2{printf \"%.2f\", $3*100/$2}'"
     
     # 收集磁盘使用率
     openclaw exec command="df -h | awk '$5 != \"Use%\" && $1 !~ /^tmpfs|cdrom/ {print $1,$5,$6}'"
     
     # 收集网络状态
     openclaw exec command="ping -c 1 google.com"
     ```
   
   - **开发分析模块**:
     创建一个脚本来分析收集的数据：
     ```bash
     # 创建分析脚本
     openclaw write file_path=~/monitoring_scripts/analyze.sh content='#!/bin/bash
# Simple analysis script
cpu_usage=$(top -bn1 | grep "%Cpu" | awk "{print \$2}" | sed "s/%us,//")
memory_usage=$(free -m | awk NR==2"{printf "%.2f", $3*100/$2}")

if (( $(echo "$cpu_usage > 80" | bc -l) )); then
  echo "ALERT: High CPU usage detected: ${cpu_usage}%"
fi

if (( $(echo "$memory_usage > 85" | bc -l) )); then
  echo "ALERT: High memory usage detected: ${memory_usage}%"
fi'
     ```
   
   - **创建报告生成模块**:
     ```bash
     # 创建报告生成脚本
     openclaw write file_path=~/monitoring_scripts/generate_report.sh content='#!/bin/bash
# Generate monitoring report
timestamp=$(date)
cpu_usage=$(top -bn1 | grep "%Cpu" | awk "{print \$2}" | sed "s/%us,//")
memory_usage=$(free -m | awk NR==2"{printf "%.2f", $3*100/$2}")
disk_usage=$(df -h / | awk NR==2"{print \$5}")

report_content="# System Monitoring Report
**Generated at:** ${timestamp}

## Resource Usage
- CPU Usage: ${cpu_usage}%
- Memory Usage: ${memory_usage}%
- Disk Usage: ${disk_usage}

## Status
All systems nominal."
     
     # Save report to file
     echo "$report_content" > ~/monitoring_reports/$(date +%Y-%m-%d)_report.md
     echo "Report generated: ~/monitoring_reports/$(date +%Y-%m-%d)_report.md"
     '
     ```

3. **集成与部署**
   - **集成各模块**:
     创建主脚本整合所有功能：
     ```bash
     openclaw write file_path=~/monitoring_scripts/main_monitor.sh content='#!/bin/bash
# Main monitoring script
echo "Starting system monitoring..."

# Collect data
./analyze.sh > ~/monitoring_logs/$(date +%Y%m%d_%H%M%S)_analysis.log 2>&1
./generate_report.sh >> ~/monitoring_logs/$(date +%Y%m%d_%H%M%S)_analysis.log 2>&1

echo "Monitoring completed. Check logs for details."
     ```
   
   - **配置定时任务**:
     ```bash
     # Make scripts executable
     openclaw exec command="chmod +x ~/monitoring_scripts/*.sh"
     
     # Create cron job for regular monitoring
     openclaw cron action=add job='{"name":"system-monitoring","schedule":{"kind":"every","everyMs":3600000},"payload":{"kind":"agentTurn","message":"Execute system monitoring script"},"sessionTarget":"isolated","enabled":true}'
     ```
   
   - **设置通知机制**:
     创建告警通知功能：
     ```bash
     openclaw write file_path=~/monitoring_scripts/send_alert.sh content='#!/bin/bash
# Send alert based on monitoring results
alert_message="$1"
if [ ! -z "$alert_message" ]; then
  echo "Sending alert: $alert_message"
  # In a real implementation, this would use the message tool
  # openclaw message action=send message="$alert_message"
else
  echo "No alert to send"
fi
'
     ```

4. **测试与优化**
   - **功能测试**:
     - 手动运行主监控脚本: `openclaw exec command="bash ~/monitoring_scripts/main_monitor.sh"`
     - 检查生成的报告: `openclaw read file_path=~/monitoring_reports/$(ls -t ~/monitoring_reports/ | head -1)`
     - 检查日志文件: `openclaw read file_path=~/monitoring_logs/$(ls -t ~/monitoring_logs/ | head -1)`
   
   - **错误处理测试**:
     - 模拟错误情况（如磁盘空间不足）
     - 验证错误处理机制
     - 检查日志中的错误记录
   
   - **性能优化**:
     - 检查脚本执行时间
     - 优化资源使用
     - 调整定时任务频率

#### 预期结果：
- 完整的自动化系统
- 各模块协同工作
- 定时任务正常运行
- 错误处理机制有效

#### 评估标准：
- [ ] 项目功能完整（所有模块正常工作）
- [ ] 模块间协作正常（数据正确传递）
- [ ] 定时任务运行正常（按计划执行）
- [ ] 错误处理有效（异常情况得到妥善处理）

#### 常见问题排查：
- 如果脚本权限不足：`openclaw exec command="chmod +x ~/monitoring_scripts/*.sh"`
- 如果定时任务未执行：检查cron配置和日志
- 如果报告未生成：检查脚本路径和权限