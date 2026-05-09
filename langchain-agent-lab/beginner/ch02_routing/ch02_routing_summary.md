# 第2章：路由（Routing）

## 概念

**路由** = 根据输入内容，自动选择不同的处理路径

```
用户输入 → [路由器] → 判断类型 → 走哪条路
               ↓
         ┌─────┼─────┐
         ↓     ↓     ↓
       路径A  路径B  路径C
```

## 代码演示的任务

模拟一个**客服协调员**，根据用户问题类型分配给不同专家：

```
"订机票" → 路由 → 预订专家
"首都问题" → 路由 → 信息专家  
"其他问题" → 路由 → 默认处理
```

## 核心代码解析

### 1. 定义子处理器
```python
def booking_handler(request):  # 预订专家
def info_handler(request):     # 信息专家
def unclear_handler(request):  # 默认处理
```

### 2. 路由提示词
```python
"分析用户请求，判断由哪个专家处理：
- 订票相关 → 输出 'booker'
- 其他信息问题 → 输出 'info'
- 不明确 → 输出 'unclear'"
```

### 3. 路由逻辑
```python
RunnableBranch(
    (判断条件1, 处理器A),
    (判断条件2, 处理器B),
    默认处理器
)
```

## 代码逻辑详解

### 整体架构

```
用户输入 → [路由链] → 分类标签 → [分支逻辑] → 对应处理器 → 返回结果
```

### 第一部分：初始化模型

```python
llm = ChatOpenAI(
    model="mimo-v2.5-pro",
    base_url="https://token-plan-cn.xiaomimimo.com/v1",
    api_key=os.environ.get("MIMO_API_KEY"),
    temperature=0
)
```

**作用**：创建 LLM 实例，`temperature=0` 确保输出稳定（分类任务需要确定性）

### 第二部分：定义处理器

```python
def booking_handler(request: str) -> str:
    """预订处理器"""
    return f"预订处理器已处理请求: '{request}'. 结果: 模拟预订操作完成."

def info_handler(request: str) -> str:
    """信息处理器"""
    return f"信息处理器已处理请求: '{request}'. 结果: 模拟信息检索完成."

def unclear_handler(request: str) -> str:
    """默认处理器"""
    return f"协调员无法处理请求: '{request}'. 请明确您的需求."
```

**作用**：三个独立的处理函数，分别处理不同类型的请求

### 第三部分：路由提示词（核心）

```python
coordinator_router_prompt = ChatPromptTemplate.from_messages([
    ("system", """分析用户请求，判断应由哪个专家处理：
     - 如果请求与预订航班或酒店相关，输出 'booker'
     - 对于其他一般信息问题，输出 'info'
     - 如果请求不明确或不属于以上类别，输出 'unclear'
     只输出一个单词: 'booker', 'info', 或 'unclear'."""),
    ("user", "{request}")
])
```

**核心作用**：
1. 定义分类规则（3种类别）
2. 限制输出格式（只输出一个单词）
3. 指导 LLM 如何理解用户意图

### 第四部分：构建路由链

```python
coordinator_router_chain = coordinator_router_prompt | llm | StrOutputParser()
```

**执行流程**：
```
{request: "帮我订机票"} 
    → coordinator_router_prompt（组装提示词）
    → llm（调用模型）
    → StrOutputParser（提取字符串）
    → 输出: "booker"
```

**关键点**：`|` 是管道符，数据从左往右流动

### 第五部分：定义分支

```python
branches = {
    "booker": RunnablePassthrough.assign(output=lambda x: booking_handler(x['request']['request'])),
    "info": RunnablePassthrough.assign(output=lambda x: info_handler(x['request']['request'])),
    "unclear": RunnablePassthrough.assign(output=lambda x: unclear_handler(x['request']['request'])),
}
```

**作用**：创建三个分支，每个分支调用对应的处理器

**数据结构**：`x` 的结构是 `{'request': {'request': '用户输入'}, 'decision': 'booker'}`

### 第六部分：分支路由器

```python
delegation_branch = RunnableBranch(
    (lambda x: x['decision'].strip() == 'booker', branches["booker"]),
    (lambda x: x['decision'].strip() == 'info', branches["info"]),
    branches["unclear"]  # 默认分支
)
```

**核心作用**：
1. 检查 `decision` 字段的值
2. 匹配到对应分支后执行
3. 没有匹配到则执行默认分支

**执行逻辑**：
```python
if decision == 'booker':
    执行 branches["booker"]
elif decision == 'info':
    执行 branches["info"]
else:
    执行 branches["unclear"]
```

### 第七部分：组合完整链

```python
coordinator_agent = {
    "decision": coordinator_router_chain,  # 路由链输出分类标签
    "request": RunnablePassthrough()       # 原样传递用户输入
} | delegation_branch | (lambda x: x['output'])
```

**数据流**：
```
输入: {"request": "帮我订机票"}
    ↓
{
    "decision": "booker",           ← 路由链的输出
    "request": {"request": "帮我订机票"}  ← 原样传递
}
    ↓
delegation_branch 匹配 "booker"
    ↓
调用 booking_handler("帮我订机票")
    ↓
{"output": "预订处理器已处理请求..."}
    ↓
lambda x: x['output']  ← 提取最终结果
    ↓
输出: "预订处理器已处理请求..."
```

### 第八部分：运行示例

```python
request_a = "帮我订一张去伦敦的机票"
result_a = coordinator_agent.invoke({"request": request_a})
```

**完整执行过程**：
1. `{"request": "帮我订一张去伦敦的机票"}` 进入链
2. 路由链分析意图 → 输出 "booker"
3. RunnableBranch 匹配到 "booker" 分支
4. 调用 `booking_handler("帮我订一张去伦敦的机票")`
5. 返回结果

### 核心代码总结

| 代码 | 作用 |
|------|------|
| `coordinator_router_prompt` | 定义分类规则的提示词 |
| `coordinator_router_chain` | 调用LLM进行意图分类 |
| `branches` | 定义三个处理器分支 |
| `RunnableBranch` | 根据分类标签选择分支 |
| `coordinator_agent` | 组合完整路由链 |

## 与第1章的区别

| 第1章 提示链 | 第2章 路由 |
|-------------|-----------|
| 固定顺序执行 | 动态选择路径 |
| 步骤A → 步骤B → 步骤C | 输入 → 选择A/B/C |
| 流程确定 | 流程不确定 |

## 实际应用场景

- **客服系统**：按问题类型分配工单
- **邮件分类**：垃圾/重要/普通分别处理
- **代码审查**：按语言选择不同检查规则
- **内容审核**：文字/图片/视频走不同流程

## 运行方式

```bash
cd chapter_notebooks/chapter_02_routing
jupyter notebook ch02_routing_langgraph.ipynb
```

## 关键点

1. `RunnableBranch` 实现条件分支
2. 路由链先判断类型，再分发到对应处理器
3. 每个分支可以是独立的处理逻辑

---

**一句话总结**：根据输入选不同路，走对路才能办对事。

---

## 补充：LangGraph 是什么

**LangGraph** 是 LangChain 团队开发的一个库，用于构建**有状态的、多步骤的 AI 应用**。

### 核心概念

```
普通 LangChain:  A → B → C（线性流程）

LangGraph:       ┌→ B ─┐
                 A      → D（支持循环、条件分支、状态管理）
                 └→ C ─┘
```

### 与 LangChain 的区别

| 特性 | LangChain | LangGraph |
|------|-----------|-----------|
| 流程 | 线性为主 | 支持复杂图结构 |
| 状态 | 无状态 | 有状态 |
| 循环 | 不支持 | 支持 |
| 人机交互 | 简单 | 支持中断/恢复 |

### 关于本章文件名的说明

本章文件名 `ch02_routing_langgraph` 有误导性。实际代码用的是 LangChain 的 `RunnableBranch` 实现路由，**没有用 LangGraph 库**。

可能作者想表达："这是用 LangChain 实现类似 LangGraph 的路由概念"。

### 总结

- **LangChain** = 基础链式调用
- **LangGraph** = 复杂有状态图（本章未使用）
