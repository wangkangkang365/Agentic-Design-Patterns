# 第13章：人机协作（Human-in-the-Loop）

## 概念

**人机协作** = 在 LLM 工作流程中引入人类决策，确保关键操作可控

```
LLM 分析 → 生成方案 → 需要人工确认？ → 是 → 等待人类审批 → 执行
                                    ↓ 否
                                  直接执行
```

## 人机协作模式

| 模式 | 说明 | 示例 |
|------|------|------|
| **审批确认** | 关键操作前需人类批准 | 发送邮件前确认 |
| **内容审核** | 人类审核 LLM 输出 | 审核生成的文章 |
| **升级处理** | 复杂问题转交人类 | 客服转人工 |

## 代码演示的任务

使用 LangChain 实现三种人机协作模式。

## 模式1：审批确认

关键操作前需要人类批准。

### 代码

```python
def execute_with_approval(action_name, action_func, auto_approve=False, **kwargs):
    """执行操作前需要审批"""
    print(f"[系统] 准备执行: {action_name}")
    
    if auto_approve:
        print("[系统] 自动审批通过")
        return action_func(**kwargs)
    
    approval = input(f"[审批] 是否批准执行 '{action_name}'？(y/n): ")
    
    if approval.lower() == 'y':
        return action_func(**kwargs)
    else:
        return {"status": "rejected"}
```

### 关键点
- 关键操作前暂停
- 等待人类确认
- 可设置自动审批模式

## 模式2：内容审核

人类审核 LLM 生成的内容。

### 代码

```python
def generate_with_review(product, auto_approve=False):
    """生成内容并审核"""
    # 生成内容
    content = generate_chain.invoke({"product": product})
    
    # 审核
    review = review_chain.invoke({"content": content})
    
    if "通过" in review:
        return content
    else:
        return None
```

### 关键点
- LLM 生成内容
- 人类审核质量
- 可自动审批（开发模式）

## 模式3：升级处理

复杂问题自动转交人类处理。

### 代码

```python
def customer_support(question, auto_approve=False):
    """带升级处理的客服系统"""
    # 分类
    category = classify_chain.invoke({"question": question})
    
    if "简单" in category:
        # 自动回答
        answer = answer_chain.invoke({"question": question})
        return {"type": "auto", "answer": answer}
    
    else:
        # 升级到人工
        result = escalate_to_human(question, category)
        return result
```

### 关键点
- 自动分类问题
- 简单问题自动回答
- 复杂问题升级人工

## 完整的人机协作客服系统

组合三种模式：分类 → 自动/人工 → 审核 → 执行

### 代码

```python
def full_customer_support(question, auto_approve=False):
    """完整的人机协作客服系统"""
    # Step 1: 分类
    category = classify_chain.invoke({"question": question})
    
    if "简单" in category:
        # Step 2: 自动回答
        answer = answer_chain.invoke({"question": question})
        
        # Step 3: 审核回答
        review = review_chain.invoke({"content": answer})
        
        if "通过" in review:
            return {"status": "sent", "answer": answer}
        else:
            return {"status": "needs_revision"}
    
    else:
        # 升级到人工
        result = escalate_to_human(question, category)
        return result
```

## 人机协作流程

```
┌─────────────────────────────────────────────────────────┐
│                人机协作流程                               │
├─────────────────────────────────────────────────────────┤
│  用户请求                                                │
│    ↓                                                    │
│  LLM 分类（简单/复杂/投诉）                               │
│    ↓                                                    │
│  ┌─────────────────────────────────────┐                │
│  │ 简单 → LLM 自动生成回答             │                │
│  │ 复杂 → 升级到人工处理               │                │
│  │ 投诉 → 升级到人工处理               │                │
│  └─────────────────────────────────────┘                │
│    ↓                                                    │
│  人类审核（可选）                                         │
│    ↓                                                    │
│  执行 / 返回修改                                          │
└─────────────────────────────────────────────────────────┘
```

## 三种模式对比

| 模式 | 触发条件 | 人类参与点 | 适用场景 |
|------|----------|------------|----------|
| **审批确认** | 关键操作前 | 执行前审批 | 发邮件、删数据 |
| **内容审核** | 生成内容后 | 审核内容质量 | 文案、代码 |
| **升级处理** | 问题复杂时 | 接管处理 | 客服、决策 |

## 实际应用场景

- **客服系统**：简单问题自动回答，复杂问题转人工
- **内容生成**：AI 生成，人类审核后发布
- **代码审查**：AI 写代码，人类 review 后合并
- **决策支持**：AI 分析数据，人类做最终决策

## 运行方式

```bash
cd chapter_notebooks/chapter_13_human_in_the_loop
jupyter notebook ch13_human_in_the_loop_langchain.ipynb
```

---

**一句话总结**：人机协作让 LLM 在关键节点引入人类决策，平衡自动化效率与可控性。
