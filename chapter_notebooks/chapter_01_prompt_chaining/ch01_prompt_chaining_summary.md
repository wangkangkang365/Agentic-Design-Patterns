# 第1章：提示链（Prompt Chaining）

## 概念

**提示链** = 把复杂任务拆分成多个步骤，上一步的输出作为下一步的输入

```
输入 → [步骤1] → 中间结果 → [步骤2] → 最终输出
```

## 代码演示

### 步骤1：提取信息
```
输入: "这款新笔记本电脑配备 3.5 GHz 八核处理器、16GB 内存和 1TB NVMe 固态硬盘。"
输出: "3.5GHz八核处理器, 16GB内存, 1TB NVMe SSD"
```

### 步骤2：转换为 JSON
```
输入: 上一步的输出
输出: {"cpu": "3.5GHz八核", "memory": "16GB", "storage": "1TB NVMe SSD"}
```

## 核心代码

```python
# 构建链
extraction_chain = prompt_extract | llm | StrOutputParser()
full_chain = {"specifications": extraction_chain} | prompt_transform | llm | StrOutputParser()

# 运行链
result = full_chain.invoke({"text_input": input_text})
```

## 为什么用提示链？

| 对比 | 单次提示 | 提示链 |
|------|---------|--------|
| 准确性 | 复杂任务易出错 | 分步处理更准确 |
| 调试 | 难定位问题 | 可检查每步结果 |
| 灵活性 | 固定流程 | 可替换任意步骤 |

## 应用场景

- **文档处理**：提取 → 翻译 → 格式化
- **数据分析**：清洗 → 分析 → 生成报告
- **代码生成**：需求 → 设计 → 实现 → 测试
- **客服系统**：理解问题 → 查询知识库 → 生成回答

## 运行方式

```bash
cd chapter_notebooks
jupyter notebook Chapter_01_Prompt_Chaining_(Code_Example).ipynb
```

## 关键点

1. 使用 `|` 管道符连接各个步骤
2. `StrOutputParser()` 将 LLM 输出转为字符串
3. `{"specifications": extraction_chain}` 传递中间结果给下一步

---

**一句话总结**：复杂任务分步做，每步输出当下步输入。
