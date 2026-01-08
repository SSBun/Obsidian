# MCPs（模型上下文协议）

MCPs 可能指的是 AI 代理生态系统中的 "Model Context Protocols"（模型上下文协议）或 "Multi-Agent Communication Protocols"（多代理通信协议）。这些是标准化的接口，允许代理：

- 共享上下文和状态信息
- 在多代理系统中协调行动
- 与不同的 AI 模型和框架集成
- 确保安全高效的通信
- 实现不同代理实现之间的互操作性

在现代 AI 框架的背景下，MCPs 促进代理与外部系统的无缝交互。

## 使用 MCPs

### MCP 的架构

MCP（Model Context Protocol）由 Anthropic 开发，是一个开放标准，用于连接 AI 模型到外部工具和数据源。

- **MCP 客户端**：通常是 AI 应用（如 Claude Desktop），连接到 MCP 服务器以访问工具。
- **MCP 服务器**：提供工具、资源和提示的服务器，可以用多种语言实现。
- **传输层**：通常使用 stdio（标准输入输出）进行通信。

### 如何使用 MCP

1. **安装客户端**：如 Claude Desktop 或其他支持 MCP 的应用。
2. **配置服务器**：在客户端配置中指定 MCP 服务器的路径和参数。
3. **连接**：客户端启动时自动连接到服务器，并列出可用工具。

#### 例子：连接到 Python MCP 服务器

```python
import asyncio
from mcp import ClientSession
from mcp.client.stdio import stdio_client

class MCPClient:
    async def connect_to_server(self, server_script_path: str):
        """连接到 MCP 服务器

        Args:
            server_script_path: 服务器脚本路径 (.py 或 .js)
        """
        is_python = server_script_path.endswith('.py')
        is_js = server_script_path.endswith('.js')
        if not (is_python or is_js):
            raise ValueError("服务器脚本必须是 .py 或 .js 文件")

        command = "python" if is_python else "node"
        server_params = StdioServerParameters(
            command=command,
            args=[server_script_path],
            env=None
        )

        stdio_transport = await self.exit_stack.enter_async_context(stdio_client(server_params))
        self.stdio, self.write = stdio_transport
        self.session = await self.exit_stack.enter_async_context(ClientSession(self.stdio, self.write))

        await self.session.initialize()

        # 列出可用工具
        response = await self.session.list_tools()
        tools = response.tools
        print("\n连接到服务器，工具列表:", [tool.name for tool in tools])
```

## 开发 MCPs

### 创建 MCP 服务器

MCP 服务器可以提供三种主要功能：
- **工具 (Tools)**：可调用的函数，AI 可以执行。
- **资源 (Resources)**：数据源，如文件、数据库。
- **提示 (Prompts)**：预定义的提示模板。

#### Python 示例：使用 fastmcp 创建计算器服务器

```python
#!/usr/bin/env python3
import asyncio
from fastmcp import FastMCP

# 创建 FastMCP 服务器
mcp = FastMCP(
    name="计算器 MCP 服务器",
    version="1.0.0"
)

@mcp.tool()
def add(a: float, b: float) -> float:
    """将两个数字相加并返回结果。"""
    return a + b

@mcp.tool()
def multiply(a: float, b: float) -> float:
    """将两个数字相乘并返回结果。"""
    return a / b

if __name__ == "__main__":
    # 使用 stdio 传输启动服务器
    asyncio.run(serve_stdio(mcp))
```

#### Node.js/TypeScript 示例

```bash
# 创建项目
mkdir weather-server
cd weather-server
npm init -y
npm install @modelcontextprotocol/sdk zod@3
npm install -D @types/node typescript
mkdir src
```

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  {
    name: "weather-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// 定义工具
server.setRequestHandler("tools/list", async () => {
  return {
    tools: [
      {
        name: "get_weather",
        description: "获取指定位置的天气",
        inputSchema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "城市名称",
            },
          },
          required: ["location"],
        },
      },
    ],
  };
});

server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "get_weather") {
    const location = args.location;
    // 模拟天气 API 调用
    return {
      content: [
        {
          type: "text",
          text: `${location} 的天气：晴天，温度 25°C`,
        },
      ],
    };
  }

  throw new Error(`未知工具: ${name}`);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("天气 MCP 服务器已启动");
}

main().catch(console.error);
```

### 部署和集成

1. **本地运行**：直接运行服务器脚本，客户端通过 stdio 连接。
2. **配置客户端**：在 Claude Desktop 等应用中配置服务器路径。
3. **测试**：使用客户端测试工具调用。
4. **生产部署**：根据需要部署到服务器，确保安全和可扩展性。

### 最佳实践

- **安全性**：验证输入，限制访问权限。
- **错误处理**：提供清晰的错误消息。
- **文档**：记录工具的参数和返回值。
- **版本控制**：使用语义版本控制。

通过 MCP，AI 代理可以无缝集成外部工具，实现更强大的功能。