# Ch16: Resource Optimization（资源优化）总结

## 核心思想

通过「模型路由」优化资源使用：根据查询复杂度选择不同成本的模型，在质量和成本之间取得平衡。

```
用户查询 → 分类器 → simple → 快速模型（低成本）
                    → reasoning → 推理模型（高质量）
```

## 代码结构

### 第1步：导入依赖
- 定义两个 LLM：`fast_llm`（低 temperature）和 `reasoning_llm`（高 temperature）
- 实际场景中对应 gpt-4o-mini vs gpt-4o vs o1 等不同成本的模型

### 第2步：查询分类器
- `CLASSIFIER_PROMPT`：定义分类规则（simple vs reasoning）
- `classify_query()` 函数：发送分类请求 → 解析 JSON 结果
- 容错处理：JSON 解析失败时根据关键词判断

### 第3步：路由器 + 执行
- `route_and_execute()` 函数：分类 → 选择模型 → 执行 → 返回结果
- simple 查询用 fast_llm + 简洁 prompt
- reasoning 查询用 reasoning_llm + 思维链 prompt

### 第4步：测试路由器
- 4 个不同复杂度的测试查询
- 展示分类结果和路由效果

### 第5步：成本分析
- 模拟 token 成本（fast: $0.0001/1K tokens, reasoning: $0.005/1K tokens）
- 对比「全部用贵模型」vs「路由策略」的成本差异

## 重难点

### 重点
- **分类器设计**：准确率直接影响质量和成本
- **模型选择策略**：不同场景下 fast vs reasoning 的权衡
- **成本量化**：用数字说话，让决策有据可依

### 难点
- **分类边界模糊**：很多查询介于 simple 和 reasoning 之间
- **模型能力差异**：不同模型的能力差异不仅仅是 temperature
- **动态调整**：生产环境需要根据实时指标动态调整路由策略

## 与其他模式的关系

| 模式 | 目的 | 资源优化角度 |
|------|------|-------------|
| Ch2 Routing | 功能分流 | 不同功能不同处理 |
| Ch16 Resource Optimization | 成本优化 | 不同复杂度不同模型 |
| Ch12 Exception Handling | 容错 | Fallback 降级 |

## 实际应用场景

| 场景 | 简单模型 | 复杂模型 |
|------|---------|----------|
| 客服系统 | FAQ 回答 | 投诉处理 |
| 代码助手 | 语法问题 | 架构设计 |
| 内容生成 | 标题/摘要 | 长文写作 |
| 数据分析 | 数据查询 | 趋势预测 |

## 进阶方向

- **多级路由**：3-4 级（mini → standard → pro → thinking）
- **动态路由**：根据实时延迟/错误率调整
- **Fallback 机制**：贵模型超时时降级到便宜模型
