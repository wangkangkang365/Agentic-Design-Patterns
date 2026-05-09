# LangChain 智能体设计模式学习资料

基于 [Agentic Design Patterns](https://github.com/evoiz/Agentic-Design-Patterns) 书籍，使用 **LangChain + MiMo 模型** 重新实现的 21 章完整学习资料。

## 环境要求

```bash
pip install langchain langchain-openai langchain-core python-dotenv jupyter notebook
```

## API 配置

```bash
# Windows PowerShell
$env:MIMO_API_KEY="your-api-key-here"

# 或创建 .env 文件
echo MIMO_API_KEY=your-api-key-here > .env
```

模型配置（所有 notebook 通用）：

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="mimo-v2.5-pro",
    base_url="https://token-plan-cn.xiaomimimo.com/v1",
    api_key=os.getenv("MIMO_API_KEY"),
)
```

## 学习路径

按难度分为三个阶段，建议按顺序学习：

### ★ 核心模式（core/）

掌握这 4 个模式，就能覆盖 80% 的 Agent 开发场景。

| 章节 | 模式 | 说明 | 文件 |
|------|------|------|------|
| Ch01 | Prompt Chaining | LLM 调用链，最基础的 Agent 模式 | `core/ch01_prompt_chaining/` |
| Ch05 | Tool Use | 工具调用，Agent 与外部世界交互的核心 | `core/ch05_tool_use/` |
| Ch07 | Multi-Agent | 多代理协作，5 种架构模式 | `core/ch07_multi_agent/` |
| Ch14 | Knowledge Retrieval | RAG 检索增强，知识库问答 | `core/ch14_knowledge_retrieval/` |

### ★★ 中等难度（intermediate/）

进阶模式，涉及复杂架构和工程化实践。

| 章节 | 模式 | 说明 | 文件 |
|------|------|------|------|
| Ch03 | Parallelization | 并行执行，提高效率 | `intermediate/ch03_parallelization/` |
| Ch04 | Reflection | 反思自检，提升输出质量 | `intermediate/ch04_reflection/` |
| Ch06 | Planning | 任务规划与分解 | `intermediate/ch06_planning/` |
| Ch10 | MCP | Model Context Protocol，标准化工具接口 | `intermediate/ch10_mcp/` |
| Ch15 | Inter-Agent | Agent 间通信，A2A 协议 | `intermediate/ch15_inter_agent/` |
| Ch16 | Resource Optimization | 资源优化，模型路由与成本控制 | `intermediate/ch16_resource_optimization/` |
| Ch17 | Reasoning | 推理增强，CoT/自纠正/规划执行 | `intermediate/ch17_reasoning/` |
| Ch18 | Guardrails | 安全护栏，输入验证+内容安全+输出检查 | `intermediate/ch18_guardrails/` |
| Ch19 | Evaluation | 评估体系，基础指标+LLM-as-Judge | `intermediate/ch19_evaluation/` |
| Ch20 | Prioritization | 任务优先级管理 | `intermediate/ch20_prioritization/` |
| Ch21 | Exploration | 探索实验，多评审 Agent 协作 | `intermediate/ch21_exploration/` |

### ★★★ 较易模式（beginner/）

基础概念，快速了解即可。

| 章节 | 模式 | 说明 | 文件 |
|------|------|------|------|
| Ch02 | Routing | 输入分类路由 | `beginner/ch02_routing/` |
| Ch08 | Memory | 记忆管理，上下文维护 | `beginner/ch08_memory/` |
| Ch09 | Adaptation | 自适应调整 | `beginner/ch09_adaptation/` |
| Ch11 | Goal Setting | 目标设定与迭代 | `beginner/ch11_goal_setting/` |
| Ch12 | Exception Handling | 异常处理与恢复 | `beginner/ch12_exception_handling/` |
| Ch13 | Human-in-the-Loop | 人机协作 | `beginner/ch13_human_in_the_loop/` |

## 目录结构

```
langchain-agent-lab/
├── README.md                              ← 本文件
├── learning_guide.md                      ← 详细学习指南
├── ai_agent_vs_traditional_programming.md ← Agent 模式 vs 传统控制流
├── demo.py                                ← 快速验证脚本
│
├── core/                                  ★ 核心模式（4 章）
│   ├── ch01_prompt_chaining/
│   ├── ch05_tool_use/
│   ├── ch07_multi_agent/
│   └── ch14_knowledge_retrieval/
│
├── intermediate/                          ★★ 中等难度（11 章）
│   ├── ch03_parallelization/
│   ├── ch04_reflection/
│   ├── ch06_planning/
│   ├── ch10_mcp/
│   └── ch15~ch21 ...
│
└── beginner/                              ★★★ 较易模式（6 章）
    ├── ch02_routing/
    ├── ch08_memory/
    └── ch09~ch13 ...
```

每个章节目录包含：
- `ch{NN}_*.ipynb` — Jupyter Notebook（可直接运行）
- `ch{NN}_*_summary.md` — 章节总结（中文）

## 快速验证

```bash
cd langchain
python demo.py
```

## 运行 Notebook

```bash
jupyter notebook core/ch05_tool_use/ch05_tool_use_langchain.ipynb
```

## 注意事项

- **MiMo 模型不支持原生 tool calling**，部分章节使用 prompt-based 方式模拟
- **Ch01/Ch02/Ch06** 使用原始版本（LangGraph / AgentExecutor），非纯 LangChain 实现
- 所有 notebook 已在 `langchain==1.2.17` + `langchain-openai>=1.0` 环境下测试通过
- 遇到 `Proactor event loop` 警告可忽略（Windows 已知问题）
