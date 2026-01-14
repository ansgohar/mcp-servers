# MCP Python SDK - Quick Reference

## Installation

```bash
# Using uv (recommended)
uv add "mcp[cli]"

# Using pip
pip install "mcp[cli]"
```

## Basic Server Structure

```python
import asyncio
from mcp.server import Server
from mcp.types import Tool, TextContent
import mcp.server.stdio

# Create server instance
server = Server("my-server-name")

# Define tools
@server.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="tool_name",
            description="What the tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "param": {
                        "type": "string",
                        "description": "Parameter description"
                    }
                },
                "required": ["param"]
            }
        )
    ]

# Handle tool calls
@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "tool_name":
        result = do_something(arguments["param"])
        return [TextContent(type="text", text=result)]
    return [TextContent(type="text", text=f"Unknown tool: {name}")]

# Run server
async def main():
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            server.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

## Defining Resources

```python
from mcp.types import Resource

@server.list_resources()
async def list_resources() -> list[Resource]:
    return [
        Resource(
            uri="file:///path/to/resource",
            name="Resource Name",
            mimeType="text/plain",
            description="Resource description"
        )
    ]

@server.read_resource()
async def read_resource(uri: str) -> str:
    # Return resource content
    return "Resource content"
```

## Defining Prompts

```python
from mcp.types import Prompt, PromptMessage

@server.list_prompts()
async def list_prompts() -> list[Prompt]:
    return [
        Prompt(
            name="prompt_name",
            description="Prompt description",
            arguments=[
                {
                    "name": "arg_name",
                    "description": "Argument description",
                    "required": True
                }
            ]
        )
    ]

@server.get_prompt()
async def get_prompt(name: str, arguments: dict) -> list[PromptMessage]:
    if name == "prompt_name":
        return [
            PromptMessage(
                role="user",
                content={"type": "text", "text": f"Prompt with {arguments['arg_name']}"}
            )
        ]
```

## Error Handling

```python
from mcp.types import McpError

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    try:
        result = dangerous_operation()
        return [TextContent(type="text", text=result)]
    except ValueError as e:
        raise McpError(f"Invalid input: {e}")
    except Exception as e:
        raise McpError(f"Unexpected error: {e}")
```

## Async Patterns

```python
import asyncio

# Concurrent operations
@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    # Run multiple async operations concurrently
    results = await asyncio.gather(
        fetch_data_1(),
        fetch_data_2(),
        fetch_data_3()
    )
    return [TextContent(type="text", text=str(results))]

# Async context manager
async def fetch_with_timeout():
    async with asyncio.timeout(30):
        return await slow_operation()
```

## Transport Options

### Stdio Transport (Local)

```python
import mcp.server.stdio

async def main():
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream, server.create_initialization_options())
```

### SSE Transport (Remote - HTTP)

```python
from mcp.server.sse import SseServerTransport
from starlette.applications import Starlette
from starlette.routing import Route

app = Starlette(
    routes=[
        Route("/sse", endpoint=sse_server, methods=["GET"]),
        Route("/messages", endpoint=handle_messages, methods=["POST"])
    ]
)

async def sse_server(request):
    async with SseServerTransport("/messages") as streams:
        await server.run(
            streams[0],
            streams[1],
            server.create_initialization_options()
        )
```

## Testing Your Server

### Using MCP Inspector

```bash
# Run inspector in Docker
docker run --rm \
  --name gohar-mcp \
  -p 127.0.0.1:6274:6274 \
  -p 127.0.0.1:6277:6277 \
  -e HOST=0.0.0.0 \
  -e MCP_AUTO_OPEN_ENABLED=false \
  ghcr.io/modelcontextprotocol/inspector:latest
```

Connect to inspector at http://127.0.0.1:6274 with:
```bash
uv --directory /path/to/project run server.py
```

### Using MCP CLI

```bash
# Install MCP CLI
uv tool install mcp

# Test server
mcp dev server.py
```

## Configuration for Claude Desktop

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "my-server": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/project",
        "run",
        "server.py"
      ]
    }
  }
}
```

## Configuration for VS Code

Add to `mcp.json`:

```json
{
  "servers": {
    "my-server": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/project",
        "run",
        "server.py"
      ]
    }
  }
}
```

## Best Practices

### 1. Type Hints
Always use type hints for better IDE support:
```python
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    pass
```

### 2. Input Validation
Validate inputs using JSON Schema in inputSchema:
```python
Tool(
    name="calculate",
    inputSchema={
        "type": "object",
        "properties": {
            "operation": {
                "type": "string",
                "enum": ["add", "subtract", "multiply", "divide"]
            },
            "a": {"type": "number"},
            "b": {"type": "number"}
        },
        "required": ["operation", "a", "b"]
    }
)
```

### 3. Error Handling
Provide meaningful error messages:
```python
try:
    result = risky_operation()
except SpecificError as e:
    return [TextContent(type="text", text=f"Error: {e}")]
```

### 4. Timeouts
Always set timeouts for long operations:
```python
async with asyncio.timeout(30):
    result = await slow_api_call()
```

### 5. Logging
Add logging for debugging:
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    logger.info(f"Tool called: {name} with {arguments}")
```

## Common Patterns

### File Operations Tool
```python
@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "read_file":
        path = Path(arguments["path"])
        content = path.read_text()
        return [TextContent(type="text", text=content)]
```

### API Call Tool
```python
import httpx

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "api_call":
        async with httpx.AsyncClient() as client:
            response = await client.get(arguments["url"])
            return [TextContent(type="text", text=response.text)]
```

### Database Query Tool
```python
import asyncpg

@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "query_db":
        conn = await asyncpg.connect(database_url)
        results = await conn.fetch(arguments["query"])
        await conn.close()
        return [TextContent(type="text", text=str(results))]
```

## Resources

- [MCP Python SDK on GitHub](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Specification](https://modelcontextprotocol.io/specification/latest)
- [Example Servers](https://github.com/modelcontextprotocol/servers)
