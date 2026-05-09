# 第12章：异常处理（Exception Handling）

## 概念

**异常处理** = 当 LLM 调用失败时，使用备用策略保证系统稳定

```
请求 → 主处理 → 失败？ → 是 → 备用处理 → 返回结果
                ↓ 否                        ↑
              返回结果 ─────────────────────┘
```

## 异常处理策略

| 策略 | 说明 | 示例 |
|------|------|------|
| **try/except** | 捕获异常，返回兜底结果 | API 超时返回默认回答 |
| **Fallback** | 主方案失败，切换备用方案 | 主模型失败换备用模型 |
| **Retry** | 失败后重试，可设置次数 | 网络波动自动重试 |

## 代码演示的任务

使用 LangChain 实现三种异常处理策略。

## 策略1：try/except 捕获异常

捕获 LLM 调用异常，返回兜底结果。

### 代码

```python
def safe_call(question: str, default: str = "抱歉，暂时无法回答。") -> str:
    """带 try/except 的安全调用"""
    try:
        return chain.invoke({"question": question})
    except Exception as e:
        print(f"调用失败: {e}")
        return default
```

### 关键点
- 防止程序崩溃
- 返回兜底结果
- 最简单的异常处理

## 策略2：Fallback 备用方案

主方案失败时，自动切换备用方案。

### 代码

```python
def call_with_fallback(question: str) -> str:
    """主方案失败时切换备用方案"""
    try:
        result = primary_chain.invoke({"question": question})
        return result
    except Exception as e:
        print(f"主方案失败，切换备用方案")
        try:
            result = fallback_chain.invoke({"question": question})
            return result
        except Exception as e2:
            return "所有方案均失败，请稍后重试。"
```

### 关键点
- 保证系统可用性
- 可以有多个备用方案
- 增加系统复杂度

## 策略3：Retry 重试机制

失败后自动重试，支持指数退避。

### 代码

```python
def call_with_retry(question: str, max_retries: int = 3, base_delay: float = 1.0) -> str:
    """带重试的调用，支持指数退避"""
    for attempt in range(max_retries):
        try:
            result = chain.invoke({"question": question})
            return result
        except Exception as e:
            delay = base_delay * (2 ** attempt)  # 指数退避
            time.sleep(delay)
    
    return f"{max_retries}次重试均失败。"
```

### 关键点
- 适合临时故障
- 指数退避避免频繁请求
- 增加响应延迟

## 完整的异常处理链

组合三种策略：try/except → Fallback → Retry

### 代码

```python
def robust_call(question: str) -> str:
    """完整的异常处理链"""
    # 第1层：Retry
    for attempt in range(2):
        try:
            # 第2层：Fallback
            try:
                result = primary_chain.invoke({"question": question})
                return f"[主方案] {result}"
            except Exception:
                result = fallback_chain.invoke({"question": question})
                return f"[备用方案] {result}"
        except Exception as e:
            if attempt < 1:
                time.sleep(1)
    
    # 第3层：兜底
    return "[兜底] 抱歉，暂时无法回答，请稍后重试。"
```

## 异常处理流程

```
┌─────────────────────────────────────────────────────────┐
│                 异常处理流程                              │
├─────────────────────────────────────────────────────────┤
│  请求                                                   │
│    ↓                                                    │
│  主方案调用 ──成功──→ 返回结果                            │
│    │                                                    │
│    失败                                                 │
│    ↓                                                    │
│  备用方案调用 ──成功──→ 返回结果                          │
│    │                                                    │
│    失败                                                 │
│    ↓                                                    │
│  重试（最多N次）──成功──→ 返回结果                        │
│    │                                                    │
│    全部失败                                              │
│    ↓                                                    │
│  返回兜底结果                                            │
└─────────────────────────────────────────────────────────┘
```

## 三种策略对比

| 策略 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| **try/except** | 任何调用 | 简单、防止崩溃 | 静默失败 |
| **Fallback** | 有备用方案 | 保证可用性 | 增加复杂度 |
| **Retry** | 临时故障 | 自动恢复 | 增加延迟 |

## 实际应用场景

- **API 超时**：Retry + 指数退避
- **模型不可用**：Fallback 到备用模型
- **格式错误**：重试 + 修正提示词
- **速率限制**：Retry + 延迟

## 运行方式

```bash
cd chapter_notebooks/chapter_12_exception_handling
jupyter notebook ch12_exception_handling_langchain.ipynb
```

---

**一句话总结**：异常处理让 LLM 调用更健壮，通过 try/except、Fallback、Retry 三层策略保证系统稳定可用。
