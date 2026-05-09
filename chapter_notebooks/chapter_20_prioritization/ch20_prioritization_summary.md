# Ch20: Prioritization（优先级排序）总结

## 核心思想

用 Agent 管理任务优先级：创建任务 → 设置优先级 → 分配人员 → 查看状态。

## 代码结构

### 第1步：导入依赖
- `langchain_core.tools.tool` 装饰器（替代已废弃的 `Tool(func=...)`）
- MiMo 模型

### 第2步：任务管理系统
- `Task` 模型：Pydantic 定义任务结构（id/description/priority/assigned_to）
- `SuperSimpleTaskManager` 类：内存任务管理器
  - `create_task()`：创建任务，自动生成 TASK-001 格式的 ID
  - `update_task()`：更新任务属性
  - `list_all_tasks()`：列出所有任务

### 第3步：定义工具
- `@tool` 装饰器定义 4 个工具
- `create_new_task`：创建任务
- `assign_priority`：设置优先级（P0/P1/P2）
- `assign_task_to_worker`：分配任务给人员
- `list_all_tasks`：列出所有任务

### 第4步：PM Agent（手动记忆管理）
- 手动维护 `chat_history` 列表
- 每次调用时将历史消息一起传入 LLM
- LLM 输出 ACTION/INPUT 格式，解析后执行工具

### 第5步：测试
- 创建紧急任务并分配
- 创建普通任务
- 查看所有任务（验证记忆是否生效）

## 废弃 API 修复

| 原始（废弃） | 修复后 |
|-------------|--------|
| `langchain.memory.ConversationBufferMemory` | 手动列表 `chat_history` |
| `create_react_agent` | `create_agent` 或手动实现 |
| `AgentExecutor(memory=...)` | 手动传入历史消息 |
| `Tool(func=...)` | `@tool` 装饰器 |

## 重难点

### 重点
- **手动记忆管理**：用列表存储对话历史，不依赖框架内置组件
- **工具定义**：`@tool` 装饰器自动生成参数 schema
- **优先级语义**：P0（紧急）> P1（重要）> P2（普通）

### 难点
- **工具调用解析**：MiMo 不支持原生 tool calling，需要 prompt 引导输出格式
- **上下文一致性**：手动管理历史时需要注意消息格式

## 与其他模式的关系

| 模式 | 说明 | 章节 |
|------|------|------|
| Tool Use | Agent 调用工具 | Ch5 |
| Memory | 手动记忆管理 | Ch8 |
| Prioritization | 任务优先级排序 | Ch20（本章）|
