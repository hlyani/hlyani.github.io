# MCP And Function Call

# 一、对比

| 对比维度       | MCP（Model Context Protocol）                                | Function Call                                                |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **架构设计**   | 客户端—服务器模式，采用标准的JSON-RPC协议，实现完全解耦的模块化架构 | 集成于模型内部，多为垂直定制，通常直接嵌入模型调用逻辑       |
| **通信模式**   | 同步与异步均支持，灵活调整消息传递方式                       | 主要以同步模式为主，异步支持有限，依赖于平台扩展实现         |
| **标准化程度** | 开放标准，支持跨平台工具调用，一次开发，众多模型共用接口     | 供应商专用，适用范围受限，每个新工具需单独开发接口，标准化程度不高 |
| **部署方式**   | 工具方提供MCP服务器，采用统一协议即可接入所有支持MCP的AI模型 | 每个API或工具都需在各自模型中集成，需定制开发，不具通用性    |
| **安全与权限** | 内嵌多层安全机制（如OAuth认证、动态权限映射和数据加密），上下文持续安全传输 | 安全机制分散，依赖开发者自行实现，缺乏统一认证流程           |
| **扩展性**     | 模块化设计，新增工具只需实现MCP接口即可，不需多次适配        | 每增加一个工具都需要重新设计和实现接口，存在重复开发问题     |

# 二、环境初始化

```
uv init mcp -p 3.10
# uv init -p 3.10
# uv venv
cd mcp
source .venv/Scripts/activate
# uv add mcp[cli] httpx pypinyin # server
# uv add openai lxml # client
```

# 三、编码

```
git clone https://gitee.com/hlyani/mcp.git
```

## 1、stdio

```
cd mcp
uv --directory weather/client run stdio.py
```

```
{
    "mcpServers": {
      "weather_stdio": {
        "disabled": false,
        "timeout": 60,
        "command": "uv",
        "args": [
          "--directory",
          "C:\\harley\\BaiduSyncdisk\\sugon\\vscode_workspace\\mcp\\weather\\server",
          "run",
          "stdio.py"
        ],
        "transportType": "stdio"
      }
    }
  }
```

## 2、sse

```
cd mcp
uv run --directory weather/server sse.py
```

```
uv --directory weather/client run sse.py
```

```
{
    "mcpServers": {
      "weather_sse": {
        "disabled": false,
        "timeout": 60,
        "url": "http://localhost:8000/sse",
        "transportType": "sse"
      }
    }
  }
```

