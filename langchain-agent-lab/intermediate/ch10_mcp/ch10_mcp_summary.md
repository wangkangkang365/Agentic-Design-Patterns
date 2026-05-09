# 第10章：MCP（Model Context Protocol）

## 概念

**MCP** = 一种开放协议，让 AI 模型与外部工具/数据源标准化连接

```
┌──────────┐    MCP协议    ┌──────────────┐
│  LLM     │ ◄──────────► │  MCP Server  │
│  Client  │              │  (工具/资源)  │
└──────────┘              └──────────────┘
```

## 与 Tool Use 的区别

| | Tool Use | MCP |
|--|----------|-----|
| **定义** | LLM 调用工具的能力 | 工具连接的标准协议 |
| **类比** | 函数调用 | USB 接口标准 |
| **关系** | MCP 实现了 Tool Use | MCP 是协议层 |

## 代码演示的任务

1. 创建 MCP Server（FastMCP）
2. 使用 MCP Client 连接
3. 集成 LangChain Agent

## Step 1：创建 MCP Server

使用 FastMCP 创建一个 MCP Server，暴露 3 个工具。

### 代码

```python
# 创建 MCP Server
mcp_server = FastMCP("demo_server")

@mcp_server.tool()
def greet(name: str) -> str:
    """根据名字生成问候语"""
    return f"你好，{name}！欢迎使用 MCP 工具。"

@mcp_server.tool()
def add(a: int, b: int) -> str:
    """计算两个整数的和"""
    return f"{a} + {b} = {a + b}"

@mcp_server.tool()
def get_weather(city: str) -> str:
    """查询指定城市的天气（模拟数据）"""
    weather_data = {
        "北京": "晴天，25°C",
        "上海": "多云，22°C",
        "深圳": "阵雨，28°C",
        "杭州": "阴天，20°C"
    }
    return weather_data.get(city, f"{city}：暂无数据")
```

### 关键点
- `@mcp_server.tool()` 装饰器将函数注册为 MCP 工具
- 函数的 docstring 成为工具的描述
- 参数类型注解自动生成 JSON Schema

## Step 2：使用 MCP Client 连接

MCP Client 可以列出工具和调用工具。

### 代码

```python
async def demo_mcp_client():
    async with Client(mcp_server) as client:
        # 列出所有工具
        tools = await client.list_tools()
        for tool in tools:
            print(f"  - {tool.name}: {tool.description}")

        # 调用工具
        result = await client.call_tool("greet", {"name": "小明"})
        print(result.data)
```

### 关键点
- `Client(mcp_server)` 使用内存传输连接
- `list_tools()` 动态发现工具
- `call_tool()` 调用指定工具

## Step 3：集成 LangChain Agent

将 MCP 工具包装为 LangChain 工具。

### 代码

```python
# 同步包装器
def call_mcp_tool(tool_name: str, **kwargs) -> str:
    async def _call():
        async with Client(mcp_server) as client:
            result = await client.call_tool(tool_name, kwargs)
            return result.data
    return _loop.run_until_complete(_call())

# 创建 LangChain 工具
greet_tool = StructuredTool.from_function(
    func=lambda name: call_mcp_tool("greet", name=name),
    name="greet",
    description="根据名字生成问候语",
    args_schema=GreetInput
)
```

### 关键点
- `StructuredTool.from_function()` 将 MCP 调用包装为 LangChain 工具
- 使用 Pydantic BaseModel 定义参数 Schema
- Agent 通过 LangChain 工具接口调用 MCP Server

## MCP 架构

```
┌─────────────────────────────────────────────────────────┐
│                    MCP 架构                              │
├─────────────────────────────────────────────────────────┤
│  MiMo LLM                                               │
│     ↓                                                   │
│  LangChain Agent                                        │
│     ↓                                                   │
│  LangChain Tools（包装 MCP Client）                      │
│     ↓                                                   │
│  MCP Client  ←── MCP 协议 ──→  MCP Server              │
│  (fastmcp)                      (FastMCP)               │
│                                 ↓                       │
│                            工具实现                      │
│                         (greet/add/weather)             │
└─────────────────────────────────────────────────────────┘
```

## MCP 核心概念

| 概念 | 说明 | 示例 |
|------|------|------|
| **MCP Server** | 暴露工具/资源的服务 | FastMCP Server |
| **MCP Client** | 连接 Server 的客户端 | `Client(mcp_server)` |
| **Tool** | 可调用的函数 | `greet`, `add`, `get_weather` |
| **Transport** | 通信方式 | stdio / HTTP / 内存 |

## MCP vs 直接 Tool Use

| | 直接 Tool Use | MCP |
|--|--------------|-----|
| **定义方式** | Python 函数 | MCP Server 暴露 |
| **发现** | 代码中硬编码 | Client 动态发现 |
| **跨语言** | 需要适配 | 协议统一 |
| **适用场景** | 单项目 | 多工具/多项目复用 |

## 运行方式

```bash
pip install fastmcp
cd chapter_notebooks/chapter_10_mcp
jupyter notebook ch10_mcp_langchain.ipynb
```

---

**一句话总结**：MCP 是工具连接的标准协议，让 LLM 可以动态发现和调用外部工具，实现 Tool Use 的标准化。
