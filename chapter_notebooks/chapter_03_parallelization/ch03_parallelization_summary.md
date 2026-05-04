# 第3章：并行化（Parallelization）

## 概念

**并行化** = 多个任务同时执行，节省时间

```
        ┌→ 任务A ─┐
输入 →  ├→ 任务B ─┤ → 合并结果
        └→ 任务C ─┘
```

## 代码演示的任务

对一个主题同时执行三个任务：

```
主题: "AI Agent 设计模式"
        ↓
    ┌───────────────────────────────────┐
    │ 并行执行（同时进行）                │
    │  ├── 生成摘要                      │
    │  ├── 生成问题                      │
    │  └── 提取关键词                    │
    └───────────────────────────────────┘
        ↓
    合并三个结果
```

## 核心代码解析

### 1. 定义三个独立的链

```python
# 总结链
summarize_chain = prompt | llm | StrOutputParser()

# 问题链
questions_chain = prompt | llm | StrOutputParser()

# 关键词链
terms_chain = prompt | llm | StrOutputParser()
```

### 2. 并行执行

```python
map_chain = RunnableParallel({
    "summary": summarize_chain,
    "questions": questions_chain,
    "terms": terms_chain
})
```

**关键**：`RunnableParallel` 让三个任务同时执行

### 3. 合成结果

```python
full_chain = map_chain | synthesis_prompt | llm | StrOutputParser()
```

## 数据流

```
输入: {"topic": "AI Agent 设计模式"}
    ↓
RunnableParallel（并行执行）
    ├── summarize_chain → "摘要内容..."
    ├── questions_chain → "问题1, 问题2, 问题3"
    └── terms_chain → "术语1, 术语2, 术语3"
    ↓
{
    "summary": "摘要内容...",
    "questions": "问题1, 问题2, 问题3",
    "terms": "术语1, 术语2, 术语3"
}
    ↓
synthesis_prompt（合成提示词）
    ↓
llm（生成最终回答）
    ↓
输出: "综合分析..."
```

## 与前两章的区别

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 |
|-------------|-----------|-------------|
| A → B → C | 选一条路 | A、B、C 同时执行 |
| 顺序执行 | 选择执行 | 并发执行 |
| 节省逻辑 | 节省判断 | 节省时间 |

## 实际应用场景

- **内容生成**：同时生成标题、摘要、标签
- **数据分析**：同时计算多个指标
- **多模型对比**：同时调用多个模型
- **文档处理**：同时翻译、摘要、提取关键词

## 运行方式

```bash
cd chapter_notebooks/chapter_03_parallelization
jupyter notebook ch03_parallelization_langchain.ipynb
```

## 关键点

1. `RunnableParallel` 实现并行执行
2. 每个任务独立，互不依赖
3. 最后合成结果时需要合并提示词

## 代码逻辑详解

### 整体架构

```
用户输入 → [并行执行多个任务] → 合并结果 → [合成链] → 最终输出
```

### 第一部分：初始化模型

```python
llm = ChatOpenAI(
    model="mimo-v2.5-pro",
    base_url="https://token-plan-cn.xiaomimimo.com/v1",
    api_key=os.environ.get("MIMO_API_KEY"),
    temperature=0.7
)
```

**作用**：创建 LLM 实例，`temperature=0.7` 允许一定创造性

### 第二部分：定义三个独立的链

```python
# 1. 总结链：生成主题摘要
summarize_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "简洁地总结以下主题:"),
        ("user", "{topic}")
    ])
    | llm
    | StrOutputParser()
)

# 2. 问题链：生成相关问题
questions_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "生成三个关于以下主题的有趣问题:"),
        ("user", "{topic}")
    ])
    | llm
    | StrOutputParser()
)

# 3. 关键词链：提取关键术语
terms_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "从以下主题中识别5-10个关键术语，用逗号分隔:"),
        ("user", "{topic}")
    ])
    | llm
    | StrOutputParser()
)
```

**作用**：三个独立的任务，互不依赖，可以并行执行

### 第三部分：并行执行

```python
map_chain = RunnableParallel({
    "summary": summarize_chain,
    "questions": questions_chain,
    "terms": terms_chain
})
```

**核心作用**：
1. 同时执行三个链
2. 将结果汇总到一个字典
3. 键名对应后续合成提示词中的变量

### 第四部分：合成提示词

```python
synthesis_prompt = ChatPromptTemplate.from_messages([
    ("system", """根据以下信息，生成一个全面的回答：
     摘要：{summary}
     问题：{questions}
     关键术语：{terms}"""),
    ("user", "请综合以上内容，提供一个完整的分析.")
])
```

**作用**：将三个并行任务的结果整合到一个提示词中

### 第五部分：组合完整链

```python
full_chain = map_chain | synthesis_prompt | llm | StrOutputParser()
```

**执行流程**：
```
输入: {"topic": "AI Agent"}
    ↓
map_chain（并行执行）
    ↓
{"summary": "...", "questions": "...", "terms": "..."}
    ↓
synthesis_prompt（组装合成提示词）
    ↓
llm（生成最终回答）
    ↓
StrOutputParser（提取字符串）
    ↓
输出: "综合分析..."
```

### 第六部分：运行示例

```python
topic = "AI Agent 设计模式"
result = full_chain.invoke({"topic": topic})
```

**完整执行过程**：
1. 输入主题进入链
2. 三个任务并行执行（同时进行）
3. 结果汇总到字典
4. 合成提示词整合结果
5. LLM 生成最终回答

## 核心代码总结

| 代码 | 作用 |
|------|------|
| `summarize_chain` | 生成摘要的独立链 |
| `questions_chain` | 生成问题的独立链 |
| `terms_chain` | 提取关键词的独立链 |
| `RunnableParallel` | 并行执行多个任务 |
| `synthesis_prompt` | 整合结果的合成提示词 |
| `full_chain` | 组合完整并行链 |

---

**一句话总结**：多个任务同时跑，省时省力效率高。
