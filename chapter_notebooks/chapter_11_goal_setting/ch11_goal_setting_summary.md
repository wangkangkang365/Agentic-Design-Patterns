# 第11章：目标设定与迭代（Goal Setting & Iteration）

## 概念

**目标设定与迭代** = 设定目标 → 生成 → 评估是否达标 → 不达标则改进 → 重复

```
设定目标 → 生成初稿 → 评估(是否达标?) → 否 → 改进 → 评估...
                              ↓ 是
                           输出结果
```

## 与第4章反思的区别

| | 第4章 反思 | 第11章 目标迭代 |
|--|-----------|----------------|
| **重点** | 找出错误/不足 | 对照目标检查 |
| **标准** | 通用质量 | 明确的目标列表 |
| **终止** | 固定次数 | 目标全部达成 |

## 代码演示的任务

设定写作目标，让 LLM 迭代改进直到所有目标达成。

## Step 1：设定目标

为写作任务设定明确的目标列表。

### 代码

```python
goals = [
    "故事字数在100-200字之间",
    "包含一个意外结局",
    "语言简洁生动",
    "主题围绕'信任'"
]
```

### 关键点
- 目标要明确、可衡量
- 数量适中（3-5个）
- 目标之间不冲突

## Step 2：生成初稿

根据目标生成初始故事。

### 代码

```python
generate_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个作家。请根据以下目标写一个故事：

目标：
{goals}

请直接输出故事内容。"""),
    ("user", "请写一个故事。")
])

generate_chain = generate_prompt | llm | StrOutputParser()
draft = generate_chain.invoke({"goals": goals_text})
```

## Step 3：评估是否达标

检查故事是否满足所有目标。

### 代码

```python
evaluate_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个评审员。请检查故事是否满足以下目标：

目标：
{goals}

故事：
{story}

对每个目标，输出：
- 达成 / 未达成：原因

最后总结：全部达成 / 未达成"""),
    ("user", "请评估故事。")
])

evaluate_chain = evaluate_prompt | llm | StrOutputParser()
```

### 关键点
- 逐个目标检查
- 给出达成/未达成判定
- 最后总结是否全部达成

## Step 4：迭代改进

如果未达成目标，根据评估结果改进故事。

### 代码

```python
improve_prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个作家。请根据评估结果改进故事，确保满足所有目标。

目标：
{goals}

当前故事：
{current_story}

评估结果：
{evaluation}

请输出改进后的故事。"""),
    ("user", "请改进故事。")
])

def iterate_until_goals(goals, max_iterations=3):
    goals_text = "\\n".join([f"- {g}" for g in goals])
    
    # 生成初稿
    story = generate_chain.invoke({"goals": goals_text})
    
    for i in range(max_iterations):
        # 评估
        evaluation = evaluate_chain.invoke({"goals": goals_text, "story": story})
        
        # 检查是否全部达成
        if "全部达成" in evaluation:
            return story
        
        # 改进故事
        story = improve_chain.invoke({
            "goals": goals_text,
            "current_story": story,
            "evaluation": evaluation
        })
    
    return story
```

### 关键点
- 循环：评估 → 检查 → 改进
- 终止条件：全部达成 或 达到最大次数
- 每次改进基于评估结果

## 目标迭代流程

```
┌─────────────────────────────────────────────────────────┐
│                目标设定与迭代流程                         │
├─────────────────────────────────────────────────────────┤
│  设定目标（明确、可衡量）                                 │
│     ↓                                                   │
│  生成初稿                                                │
│     ↓                                                   │
│  评估（对照目标检查）                                     │
│     ↓                                                   │
│  全部达成？ ──是──→ 输出结果                              │
│     │                                                   │
│     否                                                  │
│     ↓                                                   │
│  改进故事                                                │
│     ↓                                                   │
│  回到评估                                                │
└─────────────────────────────────────────────────────────┘
```

## 目标设定原则

| 原则 | 说明 | 示例 |
|------|------|------|
| **明确** | 不模糊 | "100-200字" 而非 "不要太长" |
| **可衡量** | 能判断是否达成 | "包含意外结局" 而非 "有趣" |
| **有限** | 目标数量适中 | 3-5 个目标 |
| **不冲突** | 目标之间兼容 | 不要同时要求 "简短" 和 "详细" |

## 与其他模式的关系

| 第4章 反思 | 第6章 规划 | 第9章 适应 | 第11章 目标迭代 |
|-----------|----------|----------|---------------|
| 生成→批评→改进 | 计划→执行 | 根据反馈改进 | 目标→评估→改进 |
| 通用质量改进 | 结构化执行 | 环境/反馈驱动 | 目标驱动的迭代 |

## 实际应用场景

- **代码生成**：设定功能、性能、可读性目标，迭代改进
- **文案创作**：设定字数、风格、情感目标
- **方案设计**：设定成本、可行性、效果目标
- **翻译**：设定准确度、流畅度、专业术语目标

## 运行方式

```bash
cd chapter_notebooks/chapter_11_goal_setting
jupyter notebook ch11_goal_setting_langchain.ipynb
```

---

**一句话总结**：目标设定与迭代让 LLM 通过「设定目标 → 评估 → 改进」循环，持续优化直到满足所有要求。
