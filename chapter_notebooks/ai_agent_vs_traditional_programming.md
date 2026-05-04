# AI Agent 设计模式与传统程序控制流类比

## 核心观点

AI Agent 的设计模式本质上是**用 LLM 实现传统程序的控制流**，只是判断条件从硬编码变成了 LLM 的语义理解。

---

## 传统程序的三种基本控制流

结构化编程定理：**任何程序**都可以用这三种基本结构组合而成。

| 结构 | 说明 | 示例 |
|------|------|------|
| **顺序** | 从上到下依次执行 | A → B → C |
| **选择** | 根据条件走不同分支 | if-else, switch |
| **循环** | 重复执行直到条件不满足 | while, for |

---

## 对应关系

| 程序结构 | AI Agent 模式 | 实现方式 | 章节 |
|----------|--------------|----------|------|
| 顺序 | 提示链 (Chaining) | 管道符 `\|` 串联 | 第1章 |
| 选择 | 路由 (Routing) | `RunnableBranch` | 第2章 |
| 循环 | 反思 (Reflection) | while 循环调用 LLM | 第4章 |

---

## 代码对比

### 顺序结构 → 提示链

```python
# 传统程序
result = step_a(input)
result = step_b(result)
result = step_c(result)

# AI Agent
chain = prompt_a | llm | prompt_b | llm | prompt_c | llm
result = chain.invoke(input)
```

### 选择结构 → 路由

```python
# 传统程序
if "订票" in text:
    booking_handler()
elif "信息" in text:
    info_handler()
else:
    default_handler()

# AI Agent
RunnableBranch(
    (条件A, 处理器A),
    (条件B, 处理器B),
    默认处理器
)
```

### 循环结构 → 反思

```python
# 传统程序
while not 满意:
    result = 执行任务()
    result = 检查改进(result)

# AI Agent
while not 满意:
    result = llm(任务)
    result = llm(检查改进)
```

---

## 核心区别

| 对比 | 传统程序 | AI Agent |
|------|----------|----------|
| 条件判断 | 关键词匹配、硬编码规则 | LLM 语义理解 |
| 灵活性 | 固定，需修改代码 | 动态，理解自然语言 |
| 扩展性 | 添加新条件需改代码 | 修改提示词即可 |

---

## 总结

AI Agent 设计模式 = **传统控制流 + LLM 语义理解**

三种基本结构：
1. **顺序**：提示链，A → B → C
2. **选择**：路由，根据意图选分支
3. **循环**：反思，重复执行直到满意

任何复杂的 AI Agent 系统，都是这三种结构的组合。
