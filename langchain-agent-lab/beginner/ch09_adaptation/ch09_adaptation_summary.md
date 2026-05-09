# 第9章：适应（Adaptation）

## 概念

**适应** = 让 LLM 能够根据反馈进行自我改进

```
用户反馈 → [LLM] → 分析反馈 → 调整策略 → 改进输出
```

## 适应类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **根据反馈调整** | 根据用户反馈改进回答 | 用户说"太长了"→缩短回答 |
| **自我修正** | 发现错误后自动修正 | 检查到语法错误→自动修正 |
| **环境适应** | 根据环境变化调整策略 | 时间变化→调整问候语 |

## 代码演示的任务

使用 LangChain 实现三种适应方式。

## 方式1：根据反馈调整

根据用户反馈改进回答。

### 代码

```python
# 初始回答提示词
initial_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个助手。请简洁回答用户问题。"),
    ("user", "{question}")
])

# 根据反馈调整的提示词
adapt_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个助手。请根据用户反馈调整你的回答。

原始回答：{original_answer}
用户反馈：{feedback}

请根据反馈改进回答。"""),
    ("user", "请改进回答。")
])

# 创建链
initial_chain = initial_prompt | llm | StrOutputParser()
adapt_chain = adapt_prompt | llm | StrOutputParser()

# 定义适应函数
def answer_with_adaptation(question, feedback=None):
    # 生成初始回答
    answer = initial_chain.invoke({"question": question})
    
    # 如果有反馈，进行调整
    if feedback:
        answer = adapt_chain.invoke({
            "original_answer": answer,
            "feedback": feedback
        })
    
    return answer
```

### 关键点
- 根据用户反馈改进回答
- 针对性强
- 需要用户反馈

## 方式2：自我修正

发现错误后自动修正。

### 代码

```python
# 生成回答的提示词
generate_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个助手。请回答用户问题。"),
    ("user", "{question}")
])

# 检查错误的提示词
check_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个质量检查员。请检查回答是否有问题：
1. 语法错误
2. 逻辑错误
3. 事实错误

如果有问题，输出"有问题"和具体问题。
如果没有问题，输出"没有问题"。"""),
    ("user", "问题：{question}\n回答：{answer}")
])

# 修正回答的提示词
fix_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个助手。请根据检查结果修正回答。

原始回答：{original_answer}
检查结果：{check_result}

请修正回答中的问题。"""),
    ("user", "请修正回答。")
])

# 创建链
generate_chain = generate_prompt | llm | StrOutputParser()
check_chain = check_prompt | llm | StrOutputParser()
fix_chain = fix_prompt | llm | StrOutputParser()

# 定义自我修正函数
def answer_with_self_correction(question, max_attempts=2):
    # 生成初始回答
    answer = generate_chain.invoke({"question": question})
    
    for attempt in range(max_attempts):
        # 检查回答
        check_result = check_chain.invoke({
            "question": question,
            "answer": answer
        })
        
        # 如果没有问题，返回回答
        if "没有问题" in check_result:
            return answer
        
        # 修正回答
        answer = fix_chain.invoke({
            "original_answer": answer,
            "check_result": check_result
        })
    
    return answer
```

### 关键点
- 自动检查并修正错误
- 自动改进
- 可能过度修正

## 方式3：环境适应

根据环境变化调整策略。

### 代码

```python
from datetime import datetime

# 获取当前时间
def get_time_context():
    hour = datetime.now().hour
    if hour < 12:
        return "早上"
    elif hour < 18:
        return "下午"
    else:
        return "晚上"

# 根据时间调整的提示词
time_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个助手。请根据当前时间调整你的回答风格。

当前时间：{time_context}

如果是早上，使用活力充沛的语气。
如果是下午，使用专业稳重的语气。
如果是晚上，使用轻松友好的语气。"""),
    ("user", "{question}")
])

# 创建链
time_chain = time_prompt | llm | StrOutputParser()

# 定义环境适应函数
def answer_with_environment(question):
    # 获取环境信息
    time_context = get_time_context()
    
    # 调用 LLM
    response = time_chain.invoke({
        "time_context": time_context,
        "question": question
    })
    
    return response, time_context
```

### 关键点
- 根据环境变化调整策略
- 智能、自然
- 需要环境信息

## 三种适应方式对比

| 方式 | 触发条件 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|----------|
| 根据反馈调整 | 用户反馈 | 针对性强 | 需要用户反馈 | 交互式应用 |
| 自我修正 | 检查到错误 | 自动改进 | 可能过度修正 | 质量要求高 |
| 环境适应 | 环境变化 | 智能、自然 | 需要环境信息 | 上下文相关 |

## 适应的工作原理

```
输入/反馈/环境变化
    ↓
分析输入（理解反馈或环境）
    ↓
调整策略（决定如何改进）
    ↓
生成改进的回答
    ↓
输出改进后的结果
```

## 与其他模式的关系

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 | 第5章 工具使用 | 第6章 规划 | 第7章 多代理 | 第8章 记忆 | 第9章 适应 |
|-------------|-----------|-------------|-----------|--------------|----------|-------------|----------|----------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 | LLM + 外部工具 | 计划→执行 | 多个代理协作 | 保持对话状态 | 根据反馈改进 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 | 扩展能力 | 结构化执行 | 分工合作 | 上下文保持 | 自我优化 |

## 实际应用场景

- **聊天机器人**：根据用户反馈调整回答风格
- **代码生成**：检查并修正生成的代码
- **内容创作**：根据时间、地点调整内容
- **客服系统**：根据用户情绪调整回答方式

## 运行方式

```bash
cd chapter_notebooks/chapter_09_adaptation
jupyter notebook ch09_adaptation_langchain.ipynb
```

---

**一句话总结**：适应让 LLM 能够根据反馈和环境变化进行自我改进，实现更智能的回答。
