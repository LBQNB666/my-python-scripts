# 🤖 调用专业 Agent 配置指南

> 让更专业的 Agent 帮你处理特定领域的问题

---

## 📋 当前可用的 Agent

| Agent | ID | 用途 |
|-------|-----|------|
| **Default Agent** | `default` | 当前正在用的，万金油 |
| **QA Agent** | `QwenPaw_QA_Agent_0.2` | QwenPaw 官方问答助手 |

---

## 🚀 如何调用其他 Agent

### 方法1：直接通过 chat_with_agent

```python
# 当你需要咨询特定领域的专家时，可以说：
# "帮我咨询一下 QA Agent 关于 QwenPaw 配置的问题"

# 或者直接调用：
result = await chat_with_agent(
    to_agent='QwenPaw_QA_Agent_0.2',
    text='如何配置 MCP 服务？',
    timeout=120
)
```

### 方法2：通过 Skill 调用

```python
# 使用 multi_agent_collaboration skill
# 可以同时咨询多个 Agent
```

---

## 🎯 常见的 Agent 协作场景

### 场景1：遇到复杂的技术问题

```
你 → 咨询 QA Agent（技术细节）
       ↓
       得到详细解答
       ↓
我 → 整合答案给你
```

### 场景2：需要数据分析

```
你 → "帮我分析这份销售数据"
       ↓
我 → 调用数据分析逻辑
       ↓
       或者咨询专业 Agent
       ↓
       生成分析报告
```

### 场景3：需要安全评估

```
你 → "帮我检查这段代码有没有安全问题"
       ↓
我 → 分析代码
       ↓
       或者调用安全 Agent
       ↓
       给出修复建议
```

---

## 🔧 配置更多 Agent（需要管理员配置）

如果你是 QwenPaw 管理员，可以在 `agent.json` 中配置更多 Agent：

```json
{
  "agents": [
    {
      "id": "data_analyst",
      "name": "数据分析 Agent",
      "description": "专门处理数据分析、统计、可视化的 Agent",
      "workspace_dir": "/path/to/data_agent",
      "enabled": true,
      "mcp_servers": ["python", "pandas"]
    },
    {
      "id": "security_expert",
      "name": "安全专家 Agent",
      "description": "专门处理代码审计、渗透测试、安全评估",
      "workspace_dir": "/path/to/security_agent",
      "enabled": true
    },
    {
      "id": "devops_engineer",
      "name": "运维 Agent",
      "description": "专门处理服务器部署、自动化、监控",
      "workspace_dir": "/path/to/devops_agent",
      "enabled": true
    }
  ]
}
```

---

## 📝 使用场景示例

### 示例1：需要深入的技术咨询

```python
# 假设你想深入了解某个技术问题
# 可以让我帮你咨询 QA Agent

# 问我："这个QwenPaw配置对不对？"
# 我会帮你查询文档或咨询 QA Agent
```

### 示例2：复杂任务规划

```python
# 使用 make_plan skill
# 可以让更强的 Agent 帮你制定计划

# 问我："帮我规划一下如何搭建一套自动化测试系统"
# 我会使用 make_plan skill
# 让专业 Agent 给出详细步骤
```

### 示例3：多 Agent 协作

```python
# 复杂任务可以分解给多个 Agent：
# 1. 安全 Agent → 检查代码安全
# 2. 数据 Agent → 分析性能数据
# 3. DevOps Agent → 制定部署方案

# 使用 multi_agent_collaboration skill
```

---

## 💡 什么时候应该让我调用其他 Agent？

| 情况 | 建议 |
|------|------|
| QwenPaw 配置问题 | 咨询 QA Agent |
| 需要详细技术方案 | 使用 make_plan skill |
| 复杂多步骤任务 | 多 Agent 协作 |
| 日常任务 | 我（Default Agent）直接处理 |

---

## 🎓 更好的方式：告诉我你的需求

其实最简单的方式是：**直接告诉我你想要什么**

- "我需要一个数据分析 Agent" → 我帮你配置
- "帮我咨询一下配置问题" → 我调用 QA Agent
- "这个任务很复杂，帮我规划" → 我用 make_plan

**你不用操心调用细节，说需求就行！** 😊

---

## 📌 记忆功能

我还有**记忆功能**，可以记住重要信息：

```python
# 告诉我这些，我会记住：
- "我的数据库密码是 xxx"
- "我常用的 API 是 xxx"
- "我每周五要提交周报"

# 下次直接问：
# "用我的数据库查一下今天的订单"
# "用我的 API 发送今天的日报"
```

---

## 🎯 下一步建议

告诉我你的**具体需求**，我来帮你：

1. **配置数据库连接**
2. **接入你的 API 服务**
3. **规划复杂任务**
4. **设置定时任务**
