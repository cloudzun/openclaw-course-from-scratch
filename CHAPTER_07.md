# 第七章：综合实战项目

在本章中，我们将构建一个完整的OpenClaw自动化项目，综合运用前面章节学到的所有知识。我们将创建一个智能监控和报告系统，该系统能够定期收集系统信息、分析数据、生成报告并通过多种渠道发送给相关人员。

## 7.1 项目概述

### 7.1.1 项目目标

我们将构建一个"企业IT基础设施监控助手"，该系统将具备以下功能：

1. **定期监控**：每小时检查服务器状态、磁盘使用率、网络状况
2. **数据收集**：收集系统性能指标和应用程序状态
3. **智能分析**：识别潜在问题并生成预警
4. **多渠道报告**：通过Discord、邮件等方式发送报告
5. **自动化响应**：在检测到严重问题时自动执行预定义操作

### 7.1.2 技术栈

- **监控工具**：使用OpenClaw的exec工具执行系统命令
- **数据存储**：使用memory工具管理系统状态
- **网络功能**：使用web_search和web_fetch获取外部信息
- **消息系统**：使用message工具发送报告
- **定时任务**：使用cron工具安排定期检查
- **安全机制**：应用最佳安全实践

## 7.2 项目设计

### 7.2.1 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   监控模块      │    │   分析模块      │    │   报告模块      │
│                │    │                │    │                │
│ • 系统状态检查  │───▶│ • 数据分析     │───▶│ • 生成报告     │
│ • 性能指标收集  │    │ • 异常检测     │    │ • 发送通知     │
│ • 日志审查     │    │ • 趋势预测     │    │ • 存档记录     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                        控制模块                                │
│                                                               │
│ • 定时任务管理                                                │
│ • 错误处理                                                    │
│ • 安全验证                                                    │
│ • 配置管理                                                    │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2.2 数据流

1. **数据采集**：系统每小时运行一次监控脚本
2. **数据处理**：分析采集到的数据，识别异常
3. **决策制定**：根据分析结果决定是否需要告警
4. **报告生成**：创建格式化的监控报告
5. **通知发送**：通过配置的渠道发送报告

## 7.3 项目实现

### 7.3.1 创建项目目录结构

首先，让我们创建项目的目录结构：

```bash
# 创建项目目录
mkdir -p ~/openclaw-monitoring-project/{scripts,config,data,logs}
```

### 7.3.2 配置文件设置

创建配置文件 `~/openclaw-monitoring-project/config/settings.json`：

```json
{
  "monitoring": {
    "intervals": {
      "system_check": 3600000,
      "disk_check": 7200000,
      "network_check": 1800000
    },
    "thresholds": {
      "cpu_usage": 80,
      "memory_usage": 85,
      "disk_usage": 90,
      "response_time_ms": 1000
    },
    "notification_channels": ["discord", "email"],
    "report_recipients": {
      "discord": ["IT-Alerts"],
      "email": ["admin@company.com"]
    }
  },
  "security": {
    "api_keys_required": true,
    "permission_levels": {
      "read_only": ["status", "reports"],
      "admin": ["all_operations"]
    }
  }
}
```

### 7.3.3 核心监控脚本

创建系统状态检查脚本 `~/openclaw-monitoring-project/scripts/system_monitor.js`：

```javascript
/**
 * 系统监控脚本
 * 收集系统性能指标并进行初步分析
 */

class SystemMonitor {
  constructor(config) {
    this.config = config;
    this.metrics = {};
  }

  async collectSystemMetrics() {
    try {
      // 收集CPU使用率
      const cpuUsage = await this.getCPUUsage();
      
      // 收集内存使用情况
      const memoryInfo = await this.getMemoryInfo();
      
      // 收集磁盘使用情况
      const diskUsage = await this.getDiskUsage();
      
      // 收集网络状态
      const networkStatus = await this.getNetworkStatus();
      
      // 收集进程信息
      const processInfo = await this.getProcessInfo();
      
      this.metrics = {
        timestamp: new Date().toISOString(),
        cpu: cpuUsage,
        memory: memoryInfo,
        disk: diskUsage,
        network: networkStatus,
        processes: processInfo
      };

      return this.metrics;
    } catch (error) {
      console.error("Error collecting system metrics:", error);
      throw error;
    }
  }

  async getCPUUsage() {
    // 使用exec工具执行系统命令获取CPU使用率
    const result = await exec({
      command: "top -bn1 | grep '%Cpu' | awk '{print $2}' | sed 's/%us,//'"
    });
    
    const usage = parseFloat(result.stdout.trim());
    return {
      usagePercent: usage,
      status: usage > this.config.thresholds.cpu_usage ? "WARNING" : "OK"
    };
  }

  async getMemoryInfo() {
    // 获取内存使用信息
    const result = await exec({
      command: "free -m | awk 'NR==2{printf \"%.2f\", $3*100/$2}'"
    });
    
    const usage = parseFloat(result.stdout.trim());
    return {
      usagePercent: usage,
      status: usage > this.config.thresholds.memory_usage ? "WARNING" : "OK"
    };
  }

  async getDiskUsage() {
    // 获取磁盘使用信息
    const result = await exec({
      command: "df -h | awk '$5 != \"Use%\" && $1 !~ /^tmpfs|cdrom/ {print $1,$5,$6}'"
    });
    
    const lines = result.stdout.trim().split('\n');
    const diskStats = [];
    
    for (const line of lines) {
      const [device, usage, mountPoint] = line.split(/\s+/);
      const usagePercent = parseInt(usage.replace('%', ''));
      
      diskStats.push({
        device,
        usagePercent,
        mountPoint,
        status: usagePercent > this.config.thresholds.disk_usage ? "CRITICAL" : "OK"
      });
    }
    
    return diskStats;
  }

  async getNetworkStatus() {
    // 检查网络连接状态
    const websites = ['google.com', 'github.com', 'docs.molt.bot'];
    const results = {};

    for (const site of websites) {
      try {
        const startTime = Date.now();
        const pingResult = await exec({
          command: `ping -c 1 ${site}`,
          timeout: 5000
        });
        
        const responseTime = Date.now() - startTime;
        results[site] = {
          reachable: true,
          responseTime: responseTime,
          status: responseTime > this.config.thresholds.response_time_ms ? "SLOW" : "OK"
        };
      } catch (error) {
        results[site] = {
          reachable: false,
          responseTime: null,
          status: "UNREACHABLE"
        };
      }
    }

    return results;
  }

  async getProcessInfo() {
    // 获取进程信息
    const result = await exec({
      command: "ps aux --sort=-%cpu | head -n 11"
    });

    return result.stdout;
  }

  async analyzeMetrics() {
    const alerts = [];
    
    // 分析CPU使用率
    if (this.metrics.cpu.status === "WARNING") {
      alerts.push({
        type: "CPU_USAGE_HIGH",
        severity: "WARNING",
        message: `CPU使用率达到${this.metrics.cpu.usagePercent}%，超过阈值${this.config.thresholds.cpu_usage}%`,
        timestamp: this.metrics.timestamp
      });
    }
    
    // 分析内存使用率
    if (this.metrics.memory.status === "WARNING") {
      alerts.push({
        type: "MEMORY_USAGE_HIGH", 
        severity: "WARNING",
        message: `内存使用率达到${this.metrics.memory.usagePercent}%，超过阈值${this.config.thresholds.memory_usage}%`,
        timestamp: this.metrics.timestamp
      });
    }
    
    // 分析磁盘使用率
    for (const disk of this.metrics.disk) {
      if (disk.status === "CRITICAL") {
        alerts.push({
          type: "DISK_SPACE_LOW",
          severity: "CRITICAL",
          message: `磁盘${disk.mountPoint}使用率达到${disk.usagePercent}%，超过阈值${this.config.thresholds.disk_usage}%`,
          timestamp: this.metrics.timestamp
        });
      }
    }
    
    // 分析网络状态
    for (const [site, status] of Object.entries(this.metrics.network)) {
      if (status.status === "UNREACHABLE") {
        alerts.push({
          type: "NETWORK_UNREACHABLE",
          severity: "ERROR",
          message: `网站${site}无法访问`,
          timestamp: this.metrics.timestamp
        });
      } else if (status.status === "SLOW") {
        alerts.push({
          type: "NETWORK_SLOW",
          severity: "WARNING", 
          message: `网站${site}响应时间${status.responseTime}ms，超过阈值${this.config.thresholds.response_time_ms}ms`,
          timestamp: this.metrics.timestamp
        });
      }
    }
    
    return {
      summary: {
        totalAlerts: alerts.length,
        criticalAlerts: alerts.filter(a => a.severity === "CRITICAL").length,
        warnings: alerts.filter(a => a.severity === "WARNING").length,
        errors: alerts.filter(a => a.severity === "ERROR").length
      },
      alerts: alerts
    };
  }
}

module.exports = SystemMonitor;
```

### 7.3.4 报告生成模块

创建报告生成脚本 `~/openclaw-monitoring-project/scripts/report_generator.js`：

```javascript
/**
 * 报告生成模块
 * 根据收集的数据生成格式化报告
 */
class ReportGenerator {
  constructor() {
    this.templates = {
      daily: this.dailyReportTemplate,
      alert: this.alertReportTemplate,
      summary: this.summaryReportTemplate
    };
  }

  dailyReportTemplate(data) {
    return `
# 系统监控日报
**生成时间**: ${data.timestamp}

## 系统状态摘要
- CPU使用率: ${data.metrics.cpu.usagePercent}%
- 内存使用率: ${data.metrics.memory.usagePercent}%
- 磁盘使用情况: ${data.metrics.disk.map(d => `${d.mountPoint}:${d.usagePercent}%`).join(', ')}

## 网络状态
${Object.entries(data.metrics.network).map(([site, status]) => 
  `- ${site}: ${status.reachable ? '✅' : '❌'} ${status.responseTime ? `(${status.responseTime}ms)` : ''}`
).join('\n')}

## 进程TOP 10
\`\`\`
${data.metrics.processes}
\`\`\`

## 告警信息
${data.analysis.summary.totalAlerts > 0 
  ? data.analysis.alerts.map(alert => `- **[${alert.severity}]** ${alert.message}`).join('\n')
  : '无告警'}

---
*此报告由OpenClaw监控系统自动生成*
    `;
  }

  alertReportTemplate(alertData) {
    return `
🚨 **系统告警** 🚨

**时间**: ${alertData.timestamp}
**类型**: ${alertData.type}
**级别**: ${alertData.severity}
**详情**: ${alertData.message}

请立即检查系统状态！
    `;
  }

  summaryReportTemplate(summaryData) {
    return `
📊 **监控摘要**
- 总告警数: ${summaryData.summary.totalAlerts}
- 严重告警: ${summaryData.summary.criticalAlerts}
- 警告: ${summaryData.summary.warnings}
- 错误: ${summaryData.summary.errors}

上次检查时间: ${summaryData.timestamp}
    `;
  }

  generateReport(reportType, data) {
    if (!this.templates[reportType]) {
      throw new Error(`Unknown report type: ${reportType}`);
    }
    
    return this.templates[reportType](data);
  }
}

module.exports = ReportGenerator;
```

### 7.3.5 通知发送模块

创建通知发送脚本 `~/openclaw-monitoring-project/scripts/notification_handler.js`：

```javascript
/**
 * 通知处理模块
 * 处理各种渠道的通知发送
 */
class NotificationHandler {
  constructor(config) {
    this.config = config;
  }

  async sendNotification(content, channels = null) {
    const notificationChannels = channels || this.config.notification_channels;
    const results = {};

    for (const channel of notificationChannels) {
      try {
        switch (channel) {
          case 'discord':
            results.discord = await this.sendToDiscord(content);
            break;
          case 'email':
            results.email = await this.sendToEmail(content);
            break;
          default:
            console.warn(`Unsupported notification channel: ${channel}`);
        }
      } catch (error) {
        console.error(`Failed to send notification to ${channel}:`, error);
        results[channel] = { success: false, error: error.message };
      }
    }

    return results;
  }

  async sendToDiscord(content) {
    // 发送Discord通知
    const messageResult = await message({
      action: "send",
      channel: "discord",
      message: content,
      target: this.config.report_recipients.discord[0]
    });

    return {
      success: true,
      messageId: messageResult.messageId,
      timestamp: new Date().toISOString()
    };
  }

  async sendToEmail(content) {
    // 这里我们模拟邮件发送（实际上OpenClaw没有内置邮件功能）
    // 在实际部署中，可以使用外部邮件服务API
    
    // 记录到日志作为替代
    const logEntry = {
      type: "email_simulation",
      recipients: this.config.report_recipients.email,
      content: content,
      timestamp: new Date().toISOString()
    };

    // 将邮件记录写入日志文件
    await write({
      file_path: `${process.env.HOME}/openclaw-monitoring-project/data/email_notifications.json`,
      content: JSON.stringify(logEntry, null, 2)
    });

    return {
      success: true,
      logFile: "email_notifications.json",
      timestamp: new Date().toISOString()
    };
  }
}

module.exports = NotificationHandler;
```

### 7.3.6 主控制器

创建主控制器脚本 `~/openclaw-monitoring-project/scripts/main_controller.js`：

```javascript
/**
 * 主控制器
 * 协调各个模块，执行完整的监控流程
 */
const SystemMonitor = require('./system_monitor');
const ReportGenerator = require('./report_generator');
const NotificationHandler = require('./notification_handler');

class MonitoringController {
  constructor(configPath) {
    this.config = this.loadConfig(configPath);
    this.monitor = new SystemMonitor(this.config.monitoring);
    this.reporter = new ReportGenerator();
    this.notifier = new NotificationHandler(this.config.monitoring);
  }

  loadConfig(configPath) {
    try {
      const fs = require('fs');
      const configContent = fs.readFileSync(configPath, 'utf8');
      return JSON.parse(configContent);
    } catch (error) {
      console.error("Failed to load config:", error);
      throw error;
    }
  }

  async executeFullCycle() {
    try {
      console.log("Starting monitoring cycle...");
      
      // 1. 收集系统指标
      console.log("Collecting system metrics...");
      await this.monitor.collectSystemMetrics();
      
      // 2. 分析指标
      console.log("Analyzing metrics...");
      const analysis = await this.monitor.analyzeMetrics();
      
      // 3. 生成报告
      console.log("Generating reports...");
      const reportData = {
        timestamp: new Date().toISOString(),
        metrics: this.monitor.metrics,
        analysis: analysis
      };
      
      // 生成日常报告
      const dailyReport = this.reporter.generateReport('daily', reportData);
      
      // 4. 发送通知
      console.log("Sending notifications...");
      const notificationResults = await this.notifier.sendNotification(dailyReport);
      
      // 5. 处理告警
      if (analysis.summary.totalAlerts > 0) {
        console.log(`Processing ${analysis.summary.totalAlerts} alerts...`);
        
        for (const alert of analysis.alerts) {
          const alertReport = this.reporter.generateReport('alert', alert);
          
          // 发送紧急告警通知
          await this.notifier.sendNotification(
            alertReport, 
            ['discord'] // 紧急告警只发到Discord
          );
        }
      }
      
      // 6. 保存结果到记忆系统
      await this.saveToMemory(reportData, analysis);
      
      console.log("Monitoring cycle completed successfully.");
      
      return {
        success: true,
        reportGenerated: true,
        notificationsSent: notificationResults,
        alertsProcessed: analysis.summary.totalAlerts
      };
    } catch (error) {
      console.error("Error in monitoring cycle:", error);
      
      // 发送错误通知
      const errorReport = `🚨 **监控系统错误** 🚨\n\n监控周期执行失败:\n${error.message}`;
      await this.notifier.sendNotification(errorReport, ['discord']);
      
      return {
        success: false,
        error: error.message
      };
    }
  }

  async saveToMemory(reportData, analysis) {
    // 将监控结果保存到OpenClaw的记忆系统
    const memoryEntry = {
      type: "monitoring_result",
      timestamp: reportData.timestamp,
      metrics: reportData.metrics,
      analysis: analysis,
      summary: analysis.summary
    };

    // 将数据追加到记忆文件
    const memoryPath = `${process.env.HOME}/openclaw-monitoring-project/data/monitoring_history.json`;
    
    let history = [];
    try {
      const existingData = await read({ file_path: memoryPath });
      if (existingData) {
        history = JSON.parse(existingData);
      }
    } catch (e) {
      // 如果文件不存在，从空数组开始
      history = [];
    }

    history.push(memoryEntry);
    
    // 只保留最近100条记录
    if (history.length > 100) {
      history = history.slice(-100);
    }

    await write({
      file_path: memoryPath,
      content: JSON.stringify(history, null, 2)
    });
  }

  async getHistoricalData(limit = 10) {
    const memoryPath = `${process.env.HOME}/openclaw-monitoring-project/data/monitoring_history.json`;
    
    try {
      const data = await read({ file_path: memoryPath });
      const history = JSON.parse(data);
      
      return history.slice(-limit);
    } catch (error) {
      console.error("Error retrieving historical data:", error);
      return [];
    }
  }
}

module.exports = MonitoringController;

// 如果直接运行此脚本，执行一次完整的监控周期
if (require.main === module) {
  const controller = new MonitoringController(`${process.env.HOME}/openclaw-monitoring-project/config/settings.json`);
  controller.executeFullCycle()
    .then(result => console.log("Cycle result:", result))
    .catch(err => console.error("Cycle failed:", err));
}
```

## 7.4 定时任务配置

现在我们需要配置定时任务来定期执行监控。创建cron任务：

```json
{
  "name": "system-monitoring-job",
  "schedule": {
    "kind": "every",
    "everyMs": 3600000  // 每小时执行一次
  },
  "payload": {
    "kind": "agentTurn",
    "message": "Execute the system monitoring cycle",
    "model": "qwen-portal/coder-model",
    "timeoutSeconds": 300
  },
  "sessionTarget": "isolated",
  "enabled": true
}
```

## 7.5 实施步骤

### 7.5.1 准备工作

1. **创建项目目录结构**：
   ```bash
   mkdir -p ~/openclaw-monitoring-project/{scripts,config,data,logs}
   ```

2. **创建配置文件**：
   ```bash
   # 使用之前创建的settings.json内容
   ```

3. **创建脚本文件**：
   - system_monitor.js
   - report_generator.js
   - notification_handler.js
   - main_controller.js

### 7.5.2 部署监控系统

1. **测试脚本功能**：
   ```bash
   # 手动运行一次监控周期来测试
   node ~/openclaw-monitoring-project/scripts/main_controller.js
   ```

2. **配置定时任务**：
   ```bash
   # 添加到OpenClaw的cron系统
   cron action=add job='{"name":"system-monitoring","schedule":{"kind":"every","everyMs":3600000},"payload":{"kind":"agentTurn","message":"Execute the system monitoring cycle"},"sessionTarget":"isolated","enabled":true}'
   ```

3. **验证部署**：
   - 确认脚本可以正常运行
   - 确认报告可以正确生成
   - 确认通知可以正确发送
   - 确认定时任务已添加

### 7.5.3 安全加固

1. **权限限制**：
   - 确保监控脚本只能访问必要的系统信息
   - 限制对敏感文件的访问

2. **日志记录**：
   - 记录所有监控活动
   - 记录异常和错误

3. **数据保护**：
   - 加密存储敏感配置信息
   - 定期清理旧的日志文件

## 7.6 项目优化

### 7.6.1 性能优化

1. **缓存机制**：对频繁查询的数据实施缓存
2. **异步处理**：将非关键操作异步化
3. **资源限制**：设置合理的资源使用上限

### 7.6.2 可扩展性改进

1. **插件架构**：允许添加新的监控指标
2. **配置驱动**：通过配置文件调整行为
3. **模块化设计**：各组件松耦合

### 7.6.3 错误处理增强

1. **重试机制**：对临时性错误实施重试
2. **降级策略**：在部分组件失效时保持核心功能
3. **健康检查**：监控系统自身的健康状态

## 7.7 项目总结

通过这个综合实战项目，我们成功构建了一个完整的OpenClaw自动化监控系统，综合运用了以下知识点：

1. **工具使用**：read、write、exec、message、memory等工具
2. **系统集成**：将多个功能模块整合为完整系统
3. **定时任务**：使用cron实现自动化执行
4. **错误处理**：全面的异常处理机制
5. **安全实践**：权限管理、数据保护等
6. **最佳实践**：代码组织、日志记录、性能优化

这个项目展示了如何将OpenClaw的各种功能组合起来，创建一个实用的自动化解决方案。学员可以通过修改和扩展这个项目来深化对OpenClaw的理解，并应用于实际工作中。