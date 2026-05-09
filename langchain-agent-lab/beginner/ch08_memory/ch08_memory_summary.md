# 第8章：记忆（Memory）

## 概念

**记忆** = 让 LLM 能够记住之前的对话内容

```
用户: "我叫小明"
AI: "你好，小明！"
用户: "我叫什么名字？"
AI: "你叫小明。"  ← 记住了之前的对话
```

## 记忆类型

| 类型 | 说明 | 特点 |
|------|------|------|
| **手动管理** | 使用列表保存对话历史 | 简单、灵活 |
| **向量存储记忆** | 将对话存入向量数据库 | 可检索，适合长期记忆 |

## 代码演示的任务

使用 LangChain 实现三种手动管理对话历史的方式。

## 方式1：手动管理对话历史

使用列表保存对话历史，手动传递给 LLM。

### 代码

```python
# 手动管理对话历史
chat_history = []

# 定义提示词
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个友好的助手。请根据对话历史回答问题。"),
    ("placeholder", "{chat_history}"),
    ("user", "{question}")
])

# 创建链
chain = prompt | llm | StrOutputParser()

# 定义对话函数
def chat(question):
    # 调用 LLM
    response = chain.invoke({
        "chat_history": chat_history,
        "question": question
    })
    
    # 保存对话历史
    chat_history.append(HumanMessage(content=question))
    chat_history.append(AIMessage(content=response))
    
    return response
```

### 关键点
- 使用列表保存对话历史
- 手动传递给 LLM
- 简单、灵活

## 方式2：使用字典保存对话历史

使用字典保存对话历史，更结构化。

### 代码

```python
# 使用字典保存对话历史
conversation_log = []

# 定义对话函数
def chat_with_dict(question):
    # 转换为消息格式
    messages = []
    for entry in conversation_log:
        messages.append(HumanMessage(content=entry["user"]))
        messages.append(AIMessage(content=entry["ai"]))
    
    # 调用 LLM
    response = dict_chain.invoke({
        "chat_history": messages,
        "question": question
    })
    
    # 保存到对话日志
    conversation_log.append({"user": question, "ai": response})
    
    return response
```

### 关键点
- 使用字典保存对话历史
- 结构化、可查询
- 适合需要日志记录的场景

## 方式3：限制对话历史长度

只保留最近的几轮对话，节省内存。

### 代码

```python
# 限制对话历史长度
max_history = 3  # 只保留最近3轮对话
limited_history = []

# 定义对话函数
def chat_with_limited_history(question):
    # 调用 LLM
    response = limited_chain.invoke({
        "chat_history": limited_history,
        "question": question
    })
    
    # 保存对话历史
    limited_history.append(HumanMessage(content=question))
    limited_history.append(AIMessage(content=response))
    
    # 限制历史长度
    if len(limited_history) > max_history * 2:
        limited_history.pop(0)
        limited_history.pop(0)
    
    return response
```

### 关键点
- 只保留最近的几轮对话
- 节省内存
- 可能丢失早期对话

## 三种方式对比

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| 手动管理 | 简单、灵活 | 需要自己维护 | 简单应用 |
| 字典保存 | 结构化、可查询 | 代码稍复杂 | 需要日志记录 |
| 限制长度 | 节省内存 | 可能丢失早期对话 | 长对话 |

## 记忆的工作原理

```
用户输入
    ↓
加载记忆（从存储中读取历史）
    ↓
将历史 + 新输入传递给 LLM
    ↓
LLM 生成回答
    ↓
保存记忆（将新对话写入存储）
    ↓
输出回答
```

## 与其他模式的关系

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 | 第5章 工具使用 | 第6章 规划 | 第7章 多代理 | 第8章 记忆 |
|-------------|-----------|-------------|-----------|--------------|----------|-------------|----------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 | LLM + 外部工具 | 计划→执行 | 多个代理协作 | 保持对话状态 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 | 扩展能力 | 结构化执行 | 分工合作 | 上下文保持 |

## 实际应用场景

- **聊天机器人**：记住用户偏好和对话历史
- **客服系统**：记住用户问题和解决方案
- **个人助手**：记住用户习惯和日程
- **教育系统**：记住学生学习进度

## 运行方式

```bash
cd chapter_notebooks/chapter_08_memory
jupyter notebook ch08_memory_langchain.ipynb
```

---

**一句话总结**：记忆让 LLM 能够记住之前的对话，实现真正的连续对话。
