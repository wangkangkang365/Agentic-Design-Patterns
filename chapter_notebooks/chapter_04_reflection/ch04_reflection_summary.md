# 第4章：反思（Reflection）

## 概念

**反思** = 让 LLM 对自己的输出进行评估和改进

```
输入 → [生成] → 初稿 → [批评] → 反馈 → [改进] → 最终结果
```

## 代码演示的任务

为智能咖啡杯生成产品描述，并通过反思改进：

```
产品信息 → 生成描述 → 批评评估 → 改进描述 → 最终结果
```

## 核心代码解析

### 1. 生成链：创建初稿

```python
generation_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "为智能咖啡杯写一个简短的产品描述"),
        ("user", "{product_details}")
    ])
    | llm
    | StrOutputParser()
)
```

### 2. 批评链：评估描述

```python
critique_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "批评这个产品描述，提供改进建议"),
        ("user", "产品描述：{initial_description}")
    ])
    | llm
    | StrOutputParser()
)
```

### 3. 改进链：根据批评重写

```python
refinement_chain = (
    ChatPromptTemplate.from_messages([
        ("system", "根据批评重写产品描述"),
        ("user", "原始描述：{initial_description}\n批评：{critique}")
    ])
    | llm
    | StrOutputParser()
)
```

## 数据流

```
输入: "智能咖啡杯，保温功能"
    ↓
generation_chain → "这款智能咖啡杯具有保温功能..."
    ↓
critique_chain → "描述太简单，缺少具体参数..."
    ↓
refinement_chain → "这款智能咖啡杯采用先进保温技术..."
    ↓
输出: 最终改进后的描述
```

## 与前三章的区别

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 |
|-------------|-----------|-------------|-----------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 |
| 节省逻辑 | 节省判断 | 节省时间 | 提高质量 |

## 实际应用场景

- **内容创作**：写文章→审稿→修改
- **代码生成**：写代码→代码审查→优化
- **翻译**：翻译→校对→润色
- **设计**：初稿→评审→改进

## 运行方式

```bash
cd chapter_notebooks/chapter_04_reflection
jupyter notebook ch04_reflection_langchain.ipynb
```

## 代码逻辑详解

### 整体架构

```
用户输入 → [生成初稿] → [批评评估] → [改进重写] → 最终输出
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

### 第二部分：定义三个链

```python
# 1. 生成链：创建初稿
generation_chain = prompt | llm | StrOutputParser()

# 2. 批评链：评估描述
critique_chain = prompt | llm | StrOutputParser()

# 3. 改进链：根据批评重写
refinement_chain = prompt | llm | StrOutputParser()
```

**作用**：三个独立的链，分别负责生成、批评、改进

### 第三部分：执行反思流程

```python
# 步骤1: 生成初始描述
initial_description = generation_chain.invoke({"product_details": product_details})

# 步骤2: 批评评估
critique = critique_chain.invoke({"initial_description": initial_description})

# 步骤3: 改进描述
refined_description = refinement_chain.invoke({
    "initial_description": initial_description,
    "critique": critique
})
```

**执行逻辑**：
1. 先生成初稿
2. 对初稿进行批评
3. 根据批评改进描述
4. 输出最终结果

### 数据流详解

```
输入: {"product_details": "智能咖啡杯，保温功能"}
    ↓
generation_chain.invoke()
    ↓
initial_description = "这款智能咖啡杯具有保温功能..."
    ↓
critique_chain.invoke({"initial_description": initial_description})
    ↓
critique = "描述太简单，缺少具体参数..."
    ↓
refinement_chain.invoke({
    "initial_description": initial_description,
    "critique": critique
})
    ↓
refined_description = "这款智能咖啡杯采用先进保温技术..."
    ↓
输出: 最终改进后的描述
```

## 核心代码总结

| 代码 | 作用 |
|------|------|
| `generation_chain` | 生成初始描述的链 |
| `critique_chain` | 评估描述的批评链 |
| `refinement_chain` | 根据批评改进的链 |
| `invoke()` | 执行链并获取结果 |

## 扩展：循环反思

可以多次执行反思流程，持续改进：

```python
for i in range(3):  # 循环3次
    initial_description = generation_chain.invoke(...)
    critique = critique_chain.invoke(...)
    refined_description = refinement_chain.invoke(...)
    # 用 refined_description 替换 initial_description 继续改进
```

### 循环反思的执行流程

```
第1轮: 生成初稿 → 批评 → 改进
第2轮: 改进稿 → 批评 → 进一步改进
第3轮: 进一步改进稿 → 批评 → 最终改进
```

### 循环反思的优势

1. **持续提升**：每轮都有改进，质量越来越高
2. **多角度优化**：不同轮次关注不同方面
3. **收敛到最优**：经过多轮迭代，接近最佳状态

### 代码实现

```python
def iterative_reflection(product_details, iterations=3):
    """执行多次反思迭代"""
    current_description = None
    
    for i in range(iterations):
        # 第一轮生成初稿，后续基于上一轮改进
        if i == 0:
            current_description = generation_chain.invoke(...)
        else:
            current_description = refinement_chain.invoke(...)
        
        # 批评评估
        critique = critique_chain.invoke(...)
    
    # 最终改进
    final_description = refinement_chain.invoke(...)
    return final_description
```

---

**一句话总结**：先写再改，越改越好。
