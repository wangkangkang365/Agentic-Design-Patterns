# Chapter 14: 知识检索（Knowledge Retrieval）

## 概念

**RAG（Retrieval Augmented Generation，检索增强生成）** 是一种将外部知识与 LLM 结合的技术。

### 为什么需要 RAG？

| 问题 | 说明 |
|------|------|
| 知识截止 | LLM 训练数据有截止日期，无法获取最新信息 |
| 幻觉问题 | LLM 可能编造不存在的事实 |
| 领域知识 | LLM 缺乏企业内部、专业领域的知识 |

### RAG 流程

```
用户提问 → 检索相关文档 → 将文档+问题传给LLM → 生成答案
```

## 三种检索方式

| 方式 | 原理 | 适用场景 |
|------|------|----------|
| 关键词匹配 | TF-IDF / BM25 | 简单问答，关键词明确 |
| 向量检索 | Embedding + 余弦相似度 | 语义理解，同义词多 |
| 混合检索 | 关键词 + 向量结合 | 复杂场景，精度要求高 |

## Notebook 结构

| Cell | 类型 | 内容 |
|------|------|------|
| 1 | Markdown | 概念：什么是 RAG，为什么需要 RAG |
| 2 | Code | 导入包 + 初始化 MiMo 模型 |
| 3 | Markdown | 步骤1：文档准备说明 |
| 4 | Code | 创建中文知识库（3份公司制度文档） |
| 5 | Markdown | 步骤2：文档分块说明 |
| 6 | Code | 文档分块（Text Splitting） |
| 7 | Markdown | 步骤3：检索机制说明 |
| 8 | Code | TF-IDF 索引构建 |
| 9 | Code | 检索函数 + 测试 |
| 10 | Markdown | 步骤4：RAG 生成说明 |
| 11 | Code | RAG 问答链（检索 + 生成） |
| 12 | Markdown | 步骤5：LangGraph RAG 流程说明 |
| 13 | Code | LangGraph RAG 流程构建 |
| 14 | Code | LangGraph RAG 测试 |
| 15-16 | Markdown | 总结 + 优缺点 + 应用场景 |

## 核心代码

### 1. 文档分块

```python
def split_text(text, chunk_size=200, chunk_overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end]
        chunks.append(chunk)
        start = end - chunk_overlap
    return chunks
```

### 2. TF-IDF 检索

```python
def tokenize(text):
    text = re.sub(r'[\s\W]+', '', text)
    return list(text)

def compute_tf(tokens):
    tf = {}
    for token in tokens:
        tf[token] = tf.get(token, 0) + 1
    total = len(tokens)
    return {k: v / total for k, v in tf.items()}

def compute_idf(documents):
    idf = {}
    total_docs = len(documents)
    all_tokens = set()
    for doc in documents:
        all_tokens.update(doc)
    for token in all_tokens:
        doc_count = sum(1 for doc in documents if token in doc)
        idf[token] = math.log(total_docs / (1 + doc_count))
    return idf
```

### 3. RAG 问答链

```python
rag_prompt = ChatPromptTemplate.from_template(...)
rag_chain = rag_prompt | llm | StrOutputParser()

def ask_with_rag(question):
    docs = retrieve(question, top_k=2)  # 检索
    context = "\n".join([doc.page_content for doc in docs])
    answer = rag_chain.invoke({"context": context, "question": question})  # 生成
    return answer, docs
```

### 4. LangGraph RAG

```python
workflow = StateGraph(RAGState)
workflow.add_node("retrieve", retrieve_node)
workflow.add_node("generate", generate_node)
workflow.set_entry_point("retrieve")
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)
app = workflow.compile()
```

## 运行结果

所有测试通过：

| 问题 | 检索来源 | 回答 |
|------|----------|------|
| 年假有几天？ | 休假制度 | 入职满1年5天，满5年10天，满10年15天 |
| 差旅报销标准是什么？ | 报销流程 | 一线500元/晚，二线300元/晚，餐饮100元/天 |
| 如何申请远程办公？ | 远程办公 | 提前1天在OA系统申请，经直属主管审批 |
| 加班费怎么算？ | - | 不知道（知识库中无相关信息） |

## RAG 优缺点

| 优点 | 缺点 |
|------|------|
| 减少幻觉 | 检索质量影响答案质量 |
| 知识可更新 | 需要维护知识库 |
| 可追溯来源 | 增加延迟和成本 |
| 保护数据隐私 | 分块策略影响效果 |

## 应用场景

| 场景 | 说明 |
|------|------|
| 企业知识库 | 内部制度、流程、FAQ |
| 客服系统 | 产品手册、常见问题 |
| 法律咨询 | 法规、案例检索 |
| 医疗问答 | 医学文献、药品说明 |
| 技术文档 | API 文档、开发指南 |

## 关键依赖

- `langchain-openai`：ChatOpenAI
- `langchain-core`：Document, ChatPromptTemplate, StrOutputParser
- `langgraph`：StateGraph, END

## 本章重点

1. **RAG 流程**：文档加载 → 分块 → 索引 → 检索 → 生成
2. **TF-IDF 检索**：词频 × 逆文档频率，简单有效的关键词匹配
3. **文档分块**：chunk_size 和 chunk_overlap 的权衡
4. **LangGraph 编排**：将 RAG 流程组织为状态图
5. **知识边界**：当知识库中无相关信息时，如实回答"不知道"
