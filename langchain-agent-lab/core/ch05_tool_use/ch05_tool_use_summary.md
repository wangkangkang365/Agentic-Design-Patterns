# 第5章：工具使用（Tool Use）

## 概念

**工具使用** = 让 LLM 能够调用外部工具来完成任务

```
用户提问 → [LLM] → 判断需要工具 → 调用工具 → 获取结果 → 生成回答
```

## 代码演示的任务

让 LLM 能够调用搜索工具获取信息：

```
"伦敦天气怎么样？" → LLM → 调用搜索工具 → 获取天气信息 → 生成回答
```

## 核心代码解析

### 1. 定义工具

```python
@tool
def search_information(query: str) -> str:
    """搜索信息的工具，用于查找问题的答案，如天气、首都等。"""
    simulated_results = {
        "伦敦天气": "伦敦目前天气多云，温度约15°C",
        "法国首都": "法国的首都是巴黎",
    }
    return simulated_results.get(query.lower(), "未找到信息")
```

### 2. 创建代理

```python
agent = create_agent(
    model=llm,
    tools=tools,
    system_prompt="你是一个有用的助手。可以使用工具来查找信息。",
)
```

### 3. 执行代理

```python
inputs = {"messages": [{"role": "user", "content": "伦敦天气怎么样？"}]}
for chunk in agent.stream(inputs, stream_mode="updates"):
    print(chunk)
```

## 数据流

```
输入: "伦敦天气怎么样？"
    ↓
LLM 分析问题
    ↓
判断需要调用工具
    ↓
调用 search_information("伦敦天气")
    ↓
工具返回: "伦敦目前天气多云，温度约15°C"
    ↓
LLM 生成最终回答
    ↓
输出: "根据查询，伦敦目前天气多云，温度约15°C。"
```

## 与前四章的区别

| 第1章 提示链 | 第2章 路由 | 第3章 并行化 | 第4章 反思 | 第5章 工具使用 |
|-------------|-----------|-------------|-----------|--------------|
| A → B → C | 选一条路 | A、B、C 同时执行 | 生成→批评→改进 | LLM + 外部工具 |
| 顺序执行 | 选择执行 | 并发执行 | 循环改进 | 扩展能力 |

## 实际应用场景

- **信息检索**：搜索天气、新闻、知识库
- **数据查询**：查数据库、API
- **执行操作**：发邮件、创建文件
- **计算任务**：数学计算、数据分析
- **代码执行**：运行Python代码

## 运行方式

```bash
cd chapter_notebooks/chapter_05_tool_use
jupyter notebook ch05_tool_use_langchain.ipynb
```

## 代码逻辑详解

### 整体架构

```
用户输入 → [代理] → 判断是否需要工具 → 调用工具 → 获取结果 → 生成回答
```

### 第一部分：初始化模型

```python
llm = ChatOpenAI(
    model="mimo-v2.5-pro",
    base_url="https://token-plan-cn.xiaomimimo.com/v1",
    api_key=os.environ.get("MIMO_API_KEY"),
    temperature=0
)
```

**作用**：创建 LLM 实例，`temperature=0` 确保输出稳定

### 第二部分：定义工具

```python
@tool
def search_information(query: str) -> str:
    """搜索信息的工具"""
    simulated_results = {
        "伦敦天气": "伦敦目前天气多云，温度约15°C",
        "法国首都": "法国的首都是巴黎",
    }
    return simulated_results.get(query.lower(), "未找到信息")
```

**关键点**：
1. `@tool` 装饰器将函数标记为工具
2. 文档字符串描述工具的用途（LLM会读取这个）
3. 返回工具执行结果

### 第三部分：创建代理

```python
agent = create_agent(
    model=llm,
    tools=tools,
    system_prompt="你是一个有用的助手。可以使用工具来查找信息。",
)
```

**关键点**：
1. `model` 指定使用的 LLM
2. `tools` 传入工具列表
3. `system_prompt` 定义代理的行为

### 第四部分：运行示例

```python
inputs = {"messages": [{"role": "user", "content": "伦敦天气怎么样？"}]}
for chunk in agent.stream(inputs, stream_mode="updates"):
    print(chunk)
```

**执行过程**：
1. 用户输入问题
2. LLM 分析问题，判断需要调用工具
3. 调用 `search_information("伦敦天气")`
4. 工具返回结果
5. LLM 根据工具结果生成最终回答

## 装饰器详解

### 什么是装饰器

装饰器是 Python 的一种语法糖，用 `@` 符号标记函数，为函数添加额外功能。

```python
@decorator
def function():
    pass
```

等价于：

```python
def function():
    pass
function = decorator(function)
```

### `@tool` 装饰器的作用

| 作用 | 说明 |
|------|------|
| **标记为工具** | 告诉 LLM 这个函数可以被调用 |
| **提取文档字符串** | LLM 读取文档字符串了解工具用途 |
| **自动生成参数描述** | 根据类型注解生成参数说明 |

### 文档字符串（Docstring）

文档字符串就是函数体内的**第一段字符串**，用三引号包裹：

```python
@tool
def search_information(query: str) -> str:
    """搜索信息的工具，用于查找问题的答案，如天气、首都等。"""
    return "结果"
```

其中 `"""搜索信息的工具，用于查找问题的答案，如天气、首都等。"""` 就是文档字符串。

**作用**：
- 对人：说明函数用途，帮助理解代码
- 对 LLM：LLM 读取它来了解工具功能，决定何时调用

**关键点**：
- 必须是函数体内**第一行**
- 用三引号 `"""` 包裹
- `@tool` 会自动提取它作为工具描述

### 为什么文档字符串重要

```
LLM 读取文档字符串 → 了解工具用途 → 判断何时调用 → 生成正确参数
```

如果文档字符串写得不好，LLM 可能：
- 不知道这个工具能做什么
- 在不该调用时调用
- 传入错误的参数

### 好的文档字符串示例

```python
@tool
def search_weather(city: str) -> str:
    """查询指定城市的天气信息。参数city是城市名称，如"北京"、"伦敦"。"""
    ...
```

**要素**：
1. 说明工具用途（查询天气）
2. 说明参数含义（city是城市名称）
3. 给出示例（如"北京"、"伦敦"）

## 核心代码总结

| 代码 | 作用 |
|------|------|
| `@tool` | 装饰器，将函数标记为工具 |
| `"""文档字符串"""` | 描述工具用途，LLM 读取它来了解工具 |
| `create_agent` | 创建可以调用工具的代理 |
| `agent.stream()` | 执行代理并获取结果 |

## 工具的工作原理

```
1. LLM 读取工具的文档字符串，了解工具用途
2. 用户提问时，LLM 判断是否需要调用工具
3. 如果需要，LLM 生成工具调用参数
4. 代理执行器调用工具，获取结果
5. LLM 根据工具结果生成最终回答
```

## 扩展：多个工具

可以定义多个工具，让 LLM 根据需要选择：

```python
@tool
def search_weather(city: str) -> str:
    """查询指定城市的天气信息。参数city是城市名称，如'北京'、'伦敦'。"""
    ...

@tool
def search_capital(country: str) -> str:
    """查询指定国家的首都。参数country是国家名称，如'中国'、'法国'。"""
    ...

@tool
def calculate(expression: str) -> str:
    """计算数学表达式。参数expression是数学表达式，如'2+3'、'10*5'。"""
    ...

tools = [search_weather, search_capital, calculate]
```

### 多工具代理的执行流程

```
用户: "北京天气怎么样？"
    ↓
LLM 分析问题，判断需要调用 search_weather
    ↓
调用 search_weather("北京")
    ↓
返回: "北京今天晴天，温度25°C"
    ↓
LLM 生成最终回答

用户: "日本的首都是什么？"
    ↓
LLM 分析问题，判断需要调用 search_capital
    ↓
调用 search_capital("日本")
    ↓
返回: "日本的首都是东京"
    ↓
LLM 生成最终回答
```

## 模拟工具 vs 真实工具

### 当前工具是模拟的

本章示例中的工具都是**模拟的**（写死数据）：

```python
@tool
def search_weather(city: str) -> str:
    """查询天气"""
    # 写死的数据，不是真实API
    weather_data = {
        "北京": "北京今天晴天，温度25°C",
        "上海": "上海今天多云，温度28°C",
    }
    return weather_data.get(city, "未找到天气信息")
```

**特点**：
- 数据固定，不会变化
- 不需要网络请求
- 适合学习和测试

### 真实工具示例

实际应用中，工具会调用真实的API：

```python
import requests

@tool
def search_weather(city: str) -> str:
    """查询指定城市的天气信息"""
    api_key = os.environ.get("WEATHER_API_KEY")
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(url)
    data = response.json()
    return f"{city}天气: {data['weather'][0]['description']}, 温度: {data['main']['temp']}K"
```

**特点**：
- 数据实时更新
- 需要网络请求和API密钥
- 适合生产环境

### 对比

| 对比 | 模拟工具 | 真实工具 |
|------|----------|----------|
| 数据 | 固定写死 | 实时获取 |
| 网络 | 不需要 | 需要 |
| API密钥 | 不需要 | 需要 |
| 用途 | 学习、测试 | 生产环境 |

---

**一句话总结**：LLM 不仅能思考，还能使用工具行动。`@tool` 装饰器 + 文档字符串 = 给 LLM 看的"工具使用说明书"。

---

## 扩展：Tool Use 与 MCP 的关系

### 什么是 MCP

**MCP (Model Context Protocol)** 是 Anthropic 在 2024 年底提出的开放协议，旨在标准化 AI 模型与外部工具/数据源的交互方式。

### 对比

| | Chapter 5 Tool Use | MCP |
|---|---|---|
| **层级** | 框架实现（LangChain） | 标准化协议 |
| **时间** | 2023 年已有 | 2024 年底标准化 |
| **目的** | 让 LLM 调用函数 | 统一所有 AI 的工具调用方式 |
| **关系** | 具体实现 | 抽象标准 |

### 类比关系

```
Tool Use（概念）
    ↓
LangChain @tool（实现）
    ↓
MCP（标准化协议）
```

就像：
- **HTTP** 是协议标准
- **requests**、**axios** 是具体实现库

### MCP 想解决的问题

1. **不同框架的工具定义不统一** — LangChain、LlamaIndex、CrewAI 各自为政
2. **工具复用困难** — 为 LangChain 写的工具不能直接给 CrewAI 用
3. **缺乏标准接口** — 工具的发现、调用、错误处理没有统一规范

### 结论

- Chapter 5 的 Tool Use 是**具体实现**
- MCP 是**标准化协议**
- 两者是**实现**与**标准**的关系，不是"前身"
