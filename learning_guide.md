# Agent 设计模式学习指南（Ch1-Ch21）

## 一、优先级分类

### ⭐ 核心模式（必须掌握）

| 章节 | 模式 | 重要性说明 |
|------|------|-----------|
| Ch1 | Prompt Chaining（提示链） | 所有复杂 Agent 的基础——任何多步骤任务本质都是链式组合。理解了链式调用，后面的模式都是变体 |
| Ch5 | Tool Use（工具调用） | Agent 的核心定义能力。没有工具调用，LLM 只是一个聊天机器人，不能称为 Agent。LangChain 的 `@tool` 装饰器、MCP 协议集成都是高频使用的技术 |
| Ch14 | Knowledge Retrieval / RAG（知识检索） | 企业落地最广泛的技术，面试高频考点。解决 LLM 幻觉和私有数据问题，是让 Agent "有知识" 的关键 |
| Ch7 | Multi-Agent（多代理协作） | 真实生产环境中的 Agent 系统基本都是多 Agent 协作。掌握编排模式（顺序/并行/循环/协调者）是架构师必备技能 |

### 🔴 难点模式（需要多花时间）

| 章节 | 模式 | 难在哪里 |
|------|------|----------|
| Ch7 | Multi-Agent | 涉及进程间通信、状态管理、编排逻辑，架构复杂度最高。5 种协作模式（顺序/并行/循环/协调者/Agent-as-Tool）各有适用场景，组合使用时调试困难 |
| Ch14 | RAG | 检索质量决定效果上限。需要深入理解：文本分块策略（chunk size/overlap）、向量检索 vs 关键词检索、重排序（re-ranking）、查询改写等细节。生产环境还需要考虑索引更新、多模态数据等 |
| Ch6 | Planning（规划） | 让 LLM 自主分解任务并按步骤执行，失败率较高。需要设计兜底策略、步骤验证、错误恢复机制。比 Chaining 更自主但更不可预测 |
| Ch10 | MCP（模型上下文协议） | 协议层概念多（transport/session/tool）、异步编程模型、Jupyter 环境的事件循环冲突需要额外处理（`nest_asyncio`）。理解协议设计思想比代码实现更重要 |

### 🟢 较易模式（可以快速过）

| 章节 | 模式 | 简单原因 |
|------|------|----------|
| Ch2 | Routing（路由） | 本质就是 if-else 分支，逻辑直观 |
| Ch3 | Parallelization（并行化） | Python ThreadPoolExecutor 基础，并发执行独立任务 |
| Ch4 | Reflection（反思） | 自我评估+修正循环，概念简单，实现直接 |
| Ch8 | Memory（记忆） | 手动管理列表/字典，不依赖框架内置组件（LangChain 已移除 memory 模块） |
| Ch9 | Adaptation（自适应） | try/except + 指数退避重试，常规编程模式 |
| Ch11 | Goal Setting & Iteration（目标设定与迭代） | 设定目标 → 生成草稿 → 评估 → 迭代，流程清晰 |
| Ch12 | Exception Handling（异常处理） | 同 Ch9，标准错误处理模式 |
| Ch13 | Human-in-the-Loop（人机协作） | 暂停等待人工输入，概念直观 |

---

## 二、推荐学习路径

```
第一阶段（核心突破）：Ch5 Tool Use → Ch14 RAG → Ch7 Multi-Agent
第二阶段（理论理解）：Ch6 Planning → Ch10 MCP
第三阶段（快速扫盲）：Ch1 → Ch2 → Ch3 → Ch4 → Ch8 → Ch9 → Ch11 → Ch12 → Ch13
```

**时间分配建议**：
- 第一阶段：每个模式至少 2-3 小时，包括动手写代码、调试、理解原理
- 第二阶段：每个模式 1-2 小时，重点理解设计理念
- 第三阶段：每个模式 30 分钟-1 小时，快速理解核心思想即可

---

## 三、各模式核心要点

### Ch1 — Prompt Chaining（提示链）
- **核心思想**：将复杂任务拆分为多个顺序步骤，每步的输出作为下一步的输入
- **关键代码**：LangChain 的 `|` 管道操作符，`chain = step1 | step2 | step3`
- **适用场景**：多步骤文本处理、表单填写、翻译+摘要等
- **与其他模式的关系**：所有复杂 Agent 模式都是链式调用的变体

### Ch2 — Routing（路由）
- **核心思想**：根据输入内容，将请求分发到不同的处理分支
- **关键代码**：基于 LLM 的分类判断 + 条件分支
- **适用场景**：客服意图识别、邮件分类、多语言处理
- **注意事项**：路由判断的准确性直接影响后续处理质量

### Ch3 — Parallelization（并行化）
- **核心思想**：将独立子任务并发执行，提高效率
- **关键代码**：`ThreadPoolExecutor` + `executor.map()`
- **适用场景**：多文档同时处理、多数据源聚合、批量 API 调用
- **注意事项**：注意线程安全，共享状态需要加锁

### Ch4 — Reflection（反思）
- **核心思想**：让 LLM 评估自己的输出，然后基于评估结果改进
- **关键代码**：Generator + Critic 两个角色交替执行
- **适用场景**：写作优化、代码审查、方案改进建议
- **注意事项**：反思循环需要设置最大次数，避免无限迭代

### Ch5 — Tool Use（工具调用）⭐
- **核心思想**：Agent 通过工具与外部世界交互（搜索、计算、API 调用等）
- **关键代码**：`@tool` 装饰器定义工具、`create_agent` 创建 Agent、`AgentExecutor` 执行
- **核心概念**：
  - 工具定义（名称、描述、参数 schema）
  - LLM 决定调用哪个工具、传什么参数
  - 执行工具并返回结果给 LLM
- **进阶**：MCP 协议实现工具标准化（见 Ch10）
- **实际应用**：几乎所有生产级 Agent 都需要工具调用能力

### Ch6 — Planning（规划）🔴
- **核心思想**：LLM 自主生成执行计划，然后按计划逐步执行
- **关键流程**：分析任务 → 生成步骤列表 → 逐步执行 → 汇总结果
- **与 Chaining 的区别**：Chaining 是预定义的固定流程，Planning 是 LLM 动态生成的
- **难点**：LLM 生成的计划可能不合理，需要验证和兜底机制
- **适用场景**：开放式任务、需要多步推理的复杂问题

### Ch7 — Multi-Agent（多代理协作）⭐🔴
- **5 种协作模式**：
  1. **顺序模式**：Agent A → Agent B → Agent C，类似 Chaining
  2. **并行模式**：多个 Agent 同时执行，汇总结果
  3. **循环模式**：Agent 反复执行直到满足条件
  4. **协调者模式**：一个主 Agent 协调多个子 Agent
  5. **Agent-as-Tool**：将一个 Agent 封装为另一个 Agent 的工具
- **难点**：状态管理、错误传播、Agent 间通信
- **实际应用**：复杂系统通常组合使用多种模式

### Ch8 — Memory（记忆）
- **核心思想**：让 Agent 记住历史对话，实现上下文连续性
- **3 种实现方式**：
  1. 列表存储：最简单，`messages.append()`
  2. 字典存储：按角色分类，便于检索
  3. 长度限制：滑动窗口，避免 token 超限
- **重要变化**：LangChain 1.2.17 已移除 `langchain.memory` 模块，需要手动管理
- **实际应用**：聊天机器人、对话式助手都需要记忆能力

### Ch9 — Adaptation（自适应）
- **3 种自适应模式**：
  1. **反馈自适应**：根据用户反馈调整输出
  2. **自我纠正**：检测错误并自动修复
  3. **环境自适应**：根据环境条件调整行为
- **关键代码**：`try/except` + 指数退避重试（`time.sleep(2 ** retry_count)`）
- **适用场景**：API 调用不稳定、需要容错的场景

### Ch10 — MCP（模型上下文协议）🔴
- **核心思想**：为工具/服务提供标准化的集成协议，类似 USB 协议之于外设
- **关键组件**：
  - **MCP Server**：提供工具（FastMCP 框架，`@mcp.tool()` 注册）
  - **MCP Client**：连接和调用工具
  - **Transport**：通信方式（stdio/HTTP/in-memory）
- **与 Tool Use 的关系**：Tool Use 是"怎么做"，MCP 是"用什么标准做"
- **难点**：异步编程模型、Jupyter 环境兼容性（需要 `nest_asyncio`）

### Ch11 — Goal Setting & Iteration（目标设定与迭代）
- **核心思想**：定义评估标准 → 生成内容 → 评估是否达标 → 不达标则迭代改进
- **关键流程**：设定目标（字数/风格/关键词） → LLM 生成 → LLM 评估 → 循环直到达标
- **适用场景**：文案撰写、方案生成、需要质量保证的输出任务
- **与其他模式的关系**：Reflection 是自评，Goal Setting 是基于明确标准的评估

### Ch12 — Exception Handling（异常处理）
- **3 种处理策略**：
  1. **try/except**：基础异常捕获
  2. **Fallback（降级）**：主方案失败时使用备选方案
  3. **Retry（重试）**：失败后等待重试，配合指数退避
- **适用场景**：调用外部 API、网络请求、任何可能失败的操作
- **最佳实践**：组合使用三种策略，先重试，再降级，最后抛异常

### Ch13 — Human-in-the-Loop（人机协作）
- **3 种协作模式**：
  1. **审批确认**：关键操作前等待人工批准
  2. **内容审核**：AI 生成内容由人工审核修改
  3. **升级处理**：AI 无法处理时转交人工
- **适用场景**：金融交易、医疗诊断、法律文书等需要人工把关的领域
- **设计要点**：平衡自动化效率和人工控制权

### Ch14 — Knowledge Retrieval / RAG（知识检索）⭐🔴
- **核心思想**：检索相关文档 → 增强提示词 → 生成答案（Retrieve → Augment → Generate）
- **关键流程**：
  1. 文档分块（Text Splitter）
  2. 向量化/关键词索引
  3. 相似度检索
  4. 将检索结果注入 Prompt
  5. LLM 基于检索结果生成答案
- **本项目实现**：TF-IDF 关键词匹配模拟 RAG 流程（无需真实向量数据库）
- **生产环境方案**：FAISS/Chroma/Pinecone 等向量数据库 + Embedding 模型
- **进阶方向**：查询改写、混合检索（关键词+向量）、重排序（Re-ranking）
- **LangGraph 集成**：用图结构编排 RAG 流程，支持条件分支和循环

---

## 四、一句话总结

> **Tool Use 和 RAG 是吃饭的本事，Multi-Agent 是进阶天花板，Planning 和 MCP 是设计理念，其余是辅助模式。**

---

## 五、Ch15-Ch21 补充章节

### ⭐ 重要补充（建议学习）

| 章节 | 模式 | 重要性 | 难度 | 说明 |
|------|------|--------|------|------|
| Ch17 | Reasoning（推理） | ⭐⭐⭐ | 🔴 高 | CoT、Self-Correction、Plan-and-Execute。推理能力是 Agent 智能的核心，当前 LLM 进化的主要方向 |
| Ch18 | Guardrails（护栏） | ⭐⭐⭐ | 🟡 中 | 输入验证、内容安全过滤、输出检查。生产环境必须有安全护栏 |
| Ch19 | Evaluation（评估） | ⭐⭐⭐ | 🟡 中 | 基础评估 + LLM-as-Judge。没有评估就没有改进 |
| Ch15 | Inter-Agent（代理间通信） | ⭐⭐ | 🟡 中 | A2A 协议、Agent Card、任务委托、协调者模式 |
| Ch16 | Resource Optimization（资源优化） | ⭐⭐ | 🟢 低 | 模型路由、成本优化策略 |
| Ch20 | Prioritization（优先级排序） | ⭐ | 🟢 低 | 任务管理系统、优先级分配 |
| Ch21 | Exploration（探索） | ⭐ | 🟢 低 | Agent Laboratory、多评审 Agent |

### Ch15 — Inter-Agent（代理间通信）
- **核心思想**：Agent 之间需要通信和协作，A2A 协议定义了标准方式
- **关键组件**：Agent Card（能力声明）、Agent Registry（注册表）、Coordinator（协调者）
- **三种模式**：直接调用、注册表发现、协调者调度
- **与 Ch7 的区别**：Ch7 是内部编排，Ch15 是跨 Agent 通信

### Ch16 — Resource Optimization（资源优化）
- **核心思想**：根据查询复杂度选择不同成本的模型
- **关键流程**：查询分类 → 路由到合适模型 → 执行
- **成本节省**：简单查询用便宜模型，复杂查询用贵模型
- **进阶方向**：多级路由、动态路由、Fallback 机制

### Ch17 — Reasoning（推理）🔴
- **三种推理模式**：
  1. **CoT**：在 prompt 中要求「一步一步思考」，最简单有效
  2. **Self-Correction**：回答 → 检查 → 修正，适合高风险场景
  3. **Plan-and-Execute**：先规划再执行，适合复杂任务
- **进阶方向**：Tree-of-Thought、Graph-of-Thought、Self-Consistency

### Ch18 — Guardrails（护栏）
- **三层护栏体系**：
  1. **输入验证**（Pydantic）：类型检查、格式验证
  2. **内容安全**（LLM）：Prompt Injection 检测、敏感信息过滤
  3. **输出检查**（规则引擎）：格式验证、安全词检查
- **设计原则**：多层防御、快速失败、确定性优先

### Ch19 — Evaluation（评估）
- **两种评估层次**：
  1. **基础评估**：准确率、延迟、Token 使用量（纯 Python）
  2. **LLM-as-Judge**：多维度打分（清晰度/中立性/相关性/完整性）
- **生产建议**：基础评估实时监控 + LLM-as-Judge 定期抽检

### Ch20 — Prioritization（优先级排序）
- **核心思想**：用 Agent 管理任务优先级
- **关键组件**：Task 模型、TaskManager、PM Agent
- **废弃 API 修复**：ConversationBufferMemory → 手动列表管理，create_react_agent → create_agent

### Ch21 — Exploration（探索）
- **核心思想**：多评审 Agent 对研究报告进行多角度评审
- **关键组件**：ReviewersAgent（3 个不同角色的 reviewer）、结构化 JSON 评审
- **实际应用**：论文评审、代码 Review、方案评估

### 推荐学习路径（Ch15-Ch21）

```
第1批（简单）：Ch19 评估 → Ch21 探索 → Ch16 资源优化
第2批（中等）：Ch17 推理 → Ch18 护栏 → Ch15 代理间通信
第3批（修复）：Ch20 优先级排序
```

### 完整一句话总结

> **Tool Use 和 RAG 是吃饭的本事，Multi-Agent 是进阶天花板，Reasoning 和 Guardrails 是生产必备，Planning 和 MCP 是设计理念，其余是辅助模式。**
