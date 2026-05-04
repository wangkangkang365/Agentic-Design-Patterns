# 第7章：多代理协作（Multi-Agent Collaboration）

## 概念

**多代理协作** = 多个代理协同工作，共同完成任务

## 五种协作模式

| 模式 | 说明 | 示例 |
|------|------|------|
| **顺序执行** | 代理 A → 代理 B → 代理 C | 研究 → 写作 → 编辑 |
| **并行执行** | 代理 A、B、C 同时执行 | 同时收集天气和新闻 |
| **循环执行** | 代理反复执行直到满足条件 | 生成→检查→修改→检查... |
| **协调器** | 一个代理协调其他代理 | 项目经理分配任务 |
| **代理作为工具** | 一个代理将另一个代理作为工具调用 | 专家代理作为工具 |

## 代码演示的任务

使用 LangChain 实现五种多代理协作模式。

## 模式1：顺序执行

```
研究代理 → 写作代理 → 编辑代理
```

### 代码

```python
# 定义三个代理的提示词
researcher_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个研究分析师。请为给定主题收集关键信息，输出3-5个要点。"),
    ("user", "研究主题：{topic}")
])

writer_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个技术作家。请根据研究资料撰写一篇200字的文章。"),
    ("user", "主题：{topic}\n研究资料：{research}")
])

editor_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个编辑。请对文章进行润色和优化。"),
    ("user", "主题：{topic}\n原始文章：{article}")
])

# 创建三个链
researcher_chain = researcher_prompt | llm | StrOutputParser()
writer_chain = writer_prompt | llm | StrOutputParser()
editor_chain = editor_prompt | llm | StrOutputParser()

# 顺序执行
research = researcher_chain.invoke({"topic": topic})
article = writer_chain.invoke({"topic": topic, "research": research})
final = editor_chain.invoke({"topic": topic, "article": article})
```

### 关键点
- 前一个代理的输出作为后一个代理的输入
- 简单、易理解
- 速度较慢

## 模式2：并行执行

```
        ┌→ 天气代理 →┐
用户 ──→├→ 新闻代理 →├→ 汇总
        └→ 股票代理 →┘
```

### 代码

```python
# 定义三个独立的代理
weather_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个天气分析师。请简要分析指定城市的天气情况。"),
    ("user", "分析城市：{city}的天气")
])

news_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个新闻分析师。请简要分析指定主题的最新新闻。"),
    ("user", "分析主题：{topic}的最新新闻")
])

stock_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个股票分析师。请简要分析指定股票的走势。"),
    ("user", "分析股票：{stock}的走势")
])

# 创建三个链
weather_chain = weather_prompt | llm | StrOutputParser()
news_chain = news_prompt | llm | StrOutputParser()
stock_chain = stock_prompt | llm | StrOutputParser()

# 并行执行
def run_weather():
    return weather_chain.invoke({"city": "北京"})

def run_news():
    return news_chain.invoke({"topic": "人工智能"})

def run_stock():
    return stock_chain.invoke({"stock": "小米集团"})

with ThreadPoolExecutor(max_workers=3) as executor:
    weather_future = executor.submit(run_weather)
    news_future = executor.submit(run_news)
    stock_future = executor.submit(run_stock)
    
    weather_result = weather_future.result()
    news_result = news_future.result()
    stock_result = stock_future.result()
```

### 关键点
- 多个代理同时执行
- 速度快
- 需要任务之间相互独立

## 模式3：循环执行

```
生成 → 检查 → 不满足 → 生成 → 检查 → 满足 → 输出
```

### 代码

```python
# 定义生成代理和检查代理
generator_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个创意作家。请根据主题写一个简短的口号（10字以内）。"),
    ("user", "主题：{topic}\n请写一个口号。")
])

checker_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个质量检查员。请检查口号是否符合要求：
1. 长度在10字以内
2. 朗朗上口
3. 与主题相关

如果符合要求，输出'通过'
如果不符合要求，输出'不通过'和改进建议。"""),
    ("user", "主题：{topic}\n口号：{slogan}")
])

# 创建两个链
generator_chain = generator_prompt | llm | StrOutputParser()
checker_chain = checker_prompt | llm | StrOutputParser()

# 循环执行
max_attempts = 3
attempt = 0

while attempt < max_attempts:
    attempt += 1
    
    # 生成口号
    slogan = generator_chain.invoke({"topic": topic})
    
    # 检查口号
    check_result = checker_chain.invoke({"topic": topic, "slogan": slogan})
    
    # 判断是否通过
    if "通过" in check_result and "不通过" not in check_result:
        break
```

### 关键点
- 代理反复执行直到满足条件
- 质量高
- 可能无限循环（需要设置最大尝试次数）

## 模式4：协调器模式

```
        ┌→ 研究代理 →┐
用户 ──→│  协调器   │→ 汇总
        └→ 写作代理 →┘
```

### 代码

```python
# 定义协调器代理
coordinator_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个项目经理。你的任务是分析用户需求，决定需要哪些专家来完成任务。

可用的专家：
1. 研究分析师 - 负责收集和整理信息
2. 技术作家 - 负责撰写文章

请输出需要调用的专家列表，用逗号分隔。例如：研究分析师,技术作家"""),
    ("user", "用户需求：{request}")
])

# 创建协调器链
coordinator_chain = coordinator_prompt | llm | StrOutputParser()

# 定义专家代理
expert_researcher_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个研究分析师。请为给定主题收集关键信息。"),
    ("user", "研究主题：{topic}")
])

expert_writer_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个技术作家。请根据资料撰写文章。"),
    ("user", "主题：{topic}\n资料：{research}")
])

# 创建专家链
expert_researcher_chain = expert_researcher_prompt | llm | StrOutputParser()
expert_writer_chain = expert_writer_prompt | llm | StrOutputParser()

# 协调器模式执行
experts = coordinator_chain.invoke({"request": request})

# 根据协调器的决定调用专家
if "研究分析师" in experts:
    research = expert_researcher_chain.invoke({"topic": topic})

if "技术作家" in experts:
    article = expert_writer_chain.invoke({"topic": topic, "research": research})
```

### 关键点
- 一个代理协调分配任务给其他代理
- 灵活、智能
- 复杂

## 模式5：代理作为工具

```
用户 → 主代理 → 调用专家代理（作为工具） → 返回结果
```

### 代码

```python
# 定义专家代理（作为工具）
expert_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个数学专家。请计算给定的数学表达式。"),
    ("user", "计算：{expression}")
])

expert_chain = expert_prompt | llm | StrOutputParser()

# 定义主代理
main_agent_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个智能助手。你可以使用以下工具：

工具：数学计算器
功能：计算数学表达式
使用方法：当用户问到数学问题时，调用此工具

请根据用户的问题，决定是否需要调用工具。如果需要，输出'调用工具：'和表达式。"""),
    ("user", "{question}")
])

main_agent_chain = main_agent_prompt | llm | StrOutputParser()

# 代理作为工具执行
decision = main_agent_chain.invoke({"question": question})

# 如果需要调用工具
if "调用工具" in decision:
    expression = decision.split("：")[-1] if "：" in decision else "(15 + 27) * 3 - 18"
    result = expert_chain.invoke({"expression": expression})
```

### 关键点
- 一个代理将另一个代理作为工具调用
- 专业化
- 需要工具定义

## 五种模式对比

| 模式 | 执行方式 | 优点 | 缺点 | 适用场景 |
|------|----------|------|------|----------|
| 顺序执行 | A → B → C | 简单、易理解 | 速度慢 | 流水线任务 |
| 并行执行 | A、B、C 同时 | 速度快 | 需要独立任务 | 多维度分析 |
| 循环执行 | 生成→检查→修改 | 质量高 | 可能无限循环 | 质量要求高 |
| 协调器 | 协调器分配 | 灵活、智能 | 复杂 | 复杂任务 |
| 代理作为工具 | 主代理调用专家 | 专业化 | 需要工具定义 | 专业领域 |

## 与其他模式的关系

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 | 第5章 工具使用 | 第6章 规划 | 第7章 多代理 |
|-------------|-----------|-------------|-----------|--------------|----------|-------------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 | LLM + 外部工具 | 计划→执行 | 多个代理协作 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 | 扩展能力 | 结构化执行 | 分工合作 |

## 实际应用场景

- **内容创作**：研究 → 写作 → 编辑（顺序执行）
- **数据分析**：同时分析多个维度（并行执行）
- **代码生成**：生成 → 测试 → 修改 → 测试（循环执行）
- **项目管理**：协调器分配任务（协调器模式）
- **专业咨询**：调用专家代理（代理作为工具）

## 运行方式

```bash
cd chapter_notebooks/chapter_07_multi_agent
jupyter notebook ch07_multi_agent_langchain.ipynb
```

---

**一句话总结**：多代理协作就像一个团队，每个代理负责自己的专业领域，共同完成高质量的任务。
