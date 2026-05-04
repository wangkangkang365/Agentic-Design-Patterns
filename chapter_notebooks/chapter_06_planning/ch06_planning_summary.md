# 第6章：规划（Planning）

## 概念

**规划** = 让 LLM 先制定计划，再按计划执行任务

```
用户任务 → [LLM] → 制定计划 → 按计划执行 → 输出结果
```

## 核心思路

1. **先规划**：把复杂任务拆解成步骤
2. **再执行**：按照计划逐步完成

## 代码演示的任务

让 LLM 先为"强化学习"主题制定计划，再根据计划撰写摘要：

```
"写一篇关于强化学习的摘要"
    ↓
第一步: 生成计划
  - LLM 分析主题
  - 生成3-5个步骤的计划
    ↓
第二步: 解析计划
  - 提取每个步骤
    ↓
第三步: 按计划执行
  - 对每个步骤分别执行
  - 生成每个步骤的内容
    ↓
第四步: 汇总结果
  - 将所有步骤的内容汇总
  - 生成最终文章
    ↓
输出: 完整的文章
```

## 核心代码解析

### 1. 初始化模型

```python
llm = ChatOpenAI(
    model="mimo-v2.5-pro",
    base_url="https://token-plan-cn.xiaomimimo.com/v1",
    api_key=os.environ.get("MIMO_API_KEY"),
    temperature=0
)
```

### 2. 第一步：生成计划

```python
planning_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个专业的规划专家。
你的任务是为给定主题创建一个执行计划。

请输出一个包含3-5个步骤的计划，每个步骤用数字编号。
格式要求：
1. 步骤一：xxx
2. 步骤二：xxx
3. 步骤三：xxx

只输出计划，不要添加其他内容。"""),
    ("user", "请为主题 '{topic}' 创建一个执行计划。")
])

planning_chain = planning_prompt | llm | StrOutputParser()
plan = planning_chain.invoke({"topic": topic})
```

**关键点**：
- 使用 `ChatPromptTemplate` 定义规划提示词
- 使用 LCEL（`|`）组合提示词、模型和输出解析器
- LLM 分析主题并生成结构化计划

### 3. 第二步：解析计划

```python
def parse_plan_steps(plan_text):
    """从计划文本中提取步骤"""
    # 匹配数字开头的步骤
    steps = re.findall(r'\d+\.\s*(.+)', plan_text)
    return steps

steps = parse_plan_steps(plan)
```

**关键点**：
- 使用正则表达式提取计划中的步骤
- 将计划文本转换为步骤列表

### 4. 第三步：按计划执行

```python
execution_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个专业的技术作家。
你的任务是根据给定的步骤，撰写该步骤对应的内容。

要求：
1. 只撰写该步骤的内容，不要扩展
2. 保持简洁，控制在50-100字
3. 使用清晰、专业的语言"""),
    ("user", """主题：{topic}

当前步骤：{step}

请撰写该步骤的内容。""")
])

execution_chain = execution_prompt | llm | StrOutputParser()

# 按计划执行每个步骤
step_results = []
for i, step in enumerate(steps, 1):
    result = execution_chain.invoke({"topic": topic, "step": step})
    step_results.append(result)
```

**关键点**：
- 对每个步骤分别执行
- 生成每个步骤的内容
- 保存每个步骤的结果

### 5. 第四步：汇总结果

```python
summary_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个专业的编辑。
你的任务是将多个步骤的内容汇总成一篇完整的文章。

要求：
1. 按照步骤顺序组织内容
2. 确保逻辑连贯
3. 添加标题和小标题
4. 使用 Markdown 格式"""),
    ("user", """主题：{topic}

步骤内容：
{step_contents}

请将以上内容汇总成一篇完整的文章。""")
])

# 格式化步骤内容
step_contents = ""
for i, (step, result) in enumerate(zip(steps, step_results), 1):
    step_contents += f"\n步骤 {i}: {step}\n{result}\n"

summary_chain = summary_prompt | llm | StrOutputParser()
final_output = summary_chain.invoke({"topic": topic, "step_contents": step_contents})
```

**关键点**：
- 将所有步骤的内容格式化
- 使用汇总链生成最终文章
- 输出结构化的完整文章

## 数据流

```
输入: "写一篇关于强化学习的摘要"
    ↓
第一步: 规划链
  - 输入: 主题
  - 处理: LLM 分析主题，生成计划
  - 输出: 结构化的执行计划
    ↓
第二步: 解析
  - 输入: 计划文本
  - 处理: 正则表达式提取步骤
  - 输出: 步骤列表
    ↓
第三步: 执行链
  - 输入: 主题 + 每个步骤
  - 处理: LLM 为每个步骤生成内容
  - 输出: 每个步骤的结果
    ↓
第四步: 汇总链
  - 输入: 主题 + 所有步骤内容
  - 处理: LLM 汇总成完整文章
  - 输出: 最终文章
    ↓
输出: 完整的文章
```

## 规划模式的工作原理

```
1. 用户提供任务描述
2. 规划链：LLM 分析任务，制定计划
3. 解析：提取计划中的步骤
4. 执行链：LLM 按计划的每个步骤执行
5. 汇总链：将所有步骤汇总成最终输出
```

## 规划 vs 直接执行

| | 直接执行 | 先规划再执行 |
|---|---|---|
| **方式** | LLM 直接生成回答 | LLM 先制定计划再执行 |
| **优点** | 快速 | 结构清晰、逻辑完整 |
| **缺点** | 可能遗漏要点 | 需要更多步骤 |
| **适用** | 简单任务 | 复杂任务、长文本 |

## 与其他模式的关系

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 | 第5章 工具使用 | 第6章 规划 |
|-------------|-----------|-------------|-----------|--------------|----------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 | LLM + 外部工具 | 计划→执行 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 | 扩展能力 | 结构化执行 |

## 实际应用场景

- **文章写作**：先列大纲再写正文
- **项目规划**：先制定项目计划再执行
- **代码开发**：先设计架构再编码
- **研究报告**：先确定研究框架再填充内容
- **产品设计**：先规划功能再开发

## 运行方式

```bash
cd chapter_notebooks/chapter_06_planning
jupyter notebook ch06_planning_code_example.ipynb
```

---

**一句话总结**：复杂任务先规划再执行，就像写文章先列大纲一样。规划让 LLM 的输出更有结构、更完整。
