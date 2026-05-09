# Ch21: Exploration（探索）— Agent Laboratory 总结

## 核心思想

参考 [AgentLaboratory](https://github.com/SamuelSchmidgall/AgentLaboratory) 项目，实现「多评审 Agent」模式：
- 3 个不同角色的评审 Agent 独立评审研究报告
- 每个 reviewer 有不同的关注点（实验严谨性/影响力/创新性）
- 汇总所有评审意见，投票决定 Accept/Reject

## 代码结构

### 第1步：导入依赖
- `langchain_openai.ChatOpenAI` + MiMo 模型

### 第2步：定义评审 Agent
- **Reviewer #1（严格实验评审）**：关注实验设计和数据质量
- **Reviewer #2（影响力评审）**：关注研究的影响力和实用性
- **Reviewer #3（创新性评审）**：关注研究的新颖性和原创性

### 第3步：定义评审函数
- `review_report()` 函数：发送评审 prompt → 解析 JSON 输出
- 评审模板包含 15+ 维度（summary/strengths/weaknesses/originality/quality/clarity/significance/overall/decision 等）

### 第4步：执行多 Agent 评审
- 用一篇 RAG 相关的研究报告作为示例
- 3 个评审 Agent 依次独立评审
- 每个 Agent 输出结构化 JSON 评审意见

### 第5步：汇总评审意见
- 统计 Accept/Reject 票数
- 计算平均分
- 汇总各 Agent 的优缺点

## 重难点

### 重点
- **多角色设计**：不同 reviewer persona 的 prompt 设计是关键
- **结构化输出**：JSON 格式便于自动化汇总
- **投票机制**：简单但有效的决策方式

### 难点
- **评审一致性**：同一报告多次评审可能结果不同（temperature > 0）
- **JSON 解析容错**：LLM 输出的 JSON 可能不规范
- **角色区分度**：不同 reviewer 可能给出相似的评审意见

## 与其他模式的关系

| 模式 | 评审方式 | 适用场景 |
|------|---------|----------|
| Ch4 Reflection | Agent 自评 | 个人写作优化 |
| Ch19 LLM-as-Judge | 单 Agent 多维度 | 质量监控 |
| Ch21 Agent Lab | 多 Agent 多角色 | 学术评审、方案评估 |

## 实际应用

- 学术论文自动评审
- 代码 Review（不同角色审查安全性/性能/可读性）
- 方案评估（不同利益相关者视角）
- 产品需求评审（PM/开发/测试不同视角）
