# Ch15: Inter-Agent（代理间通信）总结

## 核心思想

Agent 之间需要通信和协作。A2A（Agent-to-Agent）协议定义了 Agent 间通信的标准方式。

## 通信模式

```
Agent Card（能力声明）
    ↓
Agent Registry（注册表）
    ↓
Coordinator（协调者）→ 任务分配 → Agent A / Agent B
    ↓
结果聚合 → 返回用户
```

## 代码结构

### 第1步：导入依赖
- `langchain_openai.ChatOpenAI` + MiMo 模型

### 第2步：定义 Agent Card
- `AgentCard` 类：声明 Agent 的名称、描述、技能列表
- `has_skill()` 方法：检查是否支持某技能
- 定义了 WeatherAgent 和 KnowledgeAgent 两个 Agent Card

### 第3步：定义 Agent 和任务处理
- `Agent` 基类：`handle_task()` 方法根据技能名分发到具体方法
- `WeatherAgent`：实现 `get_weather()` 和 `get_forecast()`（模拟数据）
- `KnowledgeAgent`：实现 `answer_question()` 和 `explain_concept()`（调用 LLM）

### 第4步：Agent 间任务委托
- `AGENT_REGISTRY`：Agent 注册表，存储所有可用 Agent
- `delegate_task()` 函数：通过注册表查找目标 Agent，发送任务请求
- 测试了正常委托和错误处理（不存在的技能）

### 第5步：协调者模式
- `COORDINATOR_PROMPT`：让 LLM 分析用户需求，决定调用哪些 Agent
- `coordinator()` 函数：分析需求 → 生成任务计划 → 执行 → 汇总结果
- 支持单 Agent 和多 Agent 协作场景

## 重难点

### 重点
- **Agent Card 设计**：如何声明能力让其他 Agent 理解
- **任务委托机制**：如何发现和调用其他 Agent
- **协调者模式**：如何让主 Agent 智能地调度子 Agent

### 难点
- **Agent 发现**：如何让 Agent 知道有哪些可用的 Agent
- **错误处理**：目标 Agent 不可用或执行失败时的兜底策略
- **上下文传递**：多个 Agent 之间如何共享上下文

## 三种通信模式对比

| 模式 | 说明 | 复杂度 | 适用场景 |
|------|------|--------|----------|
| 直接调用 | Agent A 直接调用 Agent B | 低 | 简单链式流程 |
| 注册表 | 通过注册表发现 Agent | 中 | 动态 Agent 发现 |
| 协调者 | 主 Agent 统一调度 | 高 | 复杂多 Agent 系统 |

## 与其他协议的关系

| 协议 | 通信对象 | 章节 |
|------|---------|------|
| Tool Use | Agent ↔ 工具 | Ch5 |
| MCP | Agent ↔ 服务 | Ch10 |
| A2A | Agent ↔ Agent | Ch15（本章）|

## 实际应用

- 多 Agent 客服系统（路由 Agent + 专业 Agent）
- 自动化工作流（编排 Agent + 执行 Agent）
- 分布式 Agent 网络（跨服务的 Agent 协作）
