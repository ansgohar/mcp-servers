# Model Context Protocol (MCP) - Architecture Overview

## What is MCP?

MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems. Think of MCP like a USB-C port for AI applications—providing a standardized way to connect AI to data sources, tools, and workflows.

## Core Concepts

### Participants

MCP follows a client-server architecture:

- **MCP Host**: The AI application (like Claude Code, Claude Desktop, VS Code) that coordinates and manages MCP clients
- **MCP Client**: A component that maintains a connection to an MCP server and obtains context
- **MCP Server**: A program that provides context to MCP clients

Example: VS Code acts as an MCP host. When it connects to an MCP server (like Sentry or filesystem), it instantiates an MCP client object to maintain that connection.

### Architecture Layers

MCP consists of two layers:

#### 1. Data Layer
Defines the JSON-RPC based protocol for client-server communication:
- **Lifecycle management**: Connection initialization, capability negotiation, termination
- **Server features**: Tools, resources, and prompts
- **Client features**: Sampling, elicitation, logging
- **Utility features**: Notifications, progress tracking

#### 2. Transport Layer
Manages communication channels between clients and servers:
- **Stdio transport**: Uses standard input/output for local process communication (optimal performance, no network overhead)
- **Streamable HTTP transport**: Uses HTTP POST for remote server communication with optional Server-Sent Events

## Core Primitives

### Server Primitives

MCP servers can expose three core primitives:

1. **Tools**: Executable functions that AI can invoke
   - Example: File operations, API calls, database queries
   - Methods: `tools/list`, `tools/call`

2. **Resources**: Data sources that provide contextual information
   - Example: File contents, database records, API responses
   - Methods: `resources/list`, `resources/read`

3. **Prompts**: Reusable templates for structuring LLM interactions
   - Example: System prompts, few-shot examples
   - Methods: `prompts/list`, `prompts/get`

### Client Primitives

MCP clients can expose:

1. **Sampling**: Request LLM completions from the client's AI application
   - Method: `sampling/complete`

2. **Elicitation**: Request additional information from users
   - Method: `elicitation/request`

3. **Logging**: Send log messages for debugging and monitoring
   - Method: `logging/message`

## Protocol Flow Example

### 1. Initialization (Lifecycle Management)

```json
// Client Request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "elicitation": {}
    },
    "clientInfo": {
      "name": "example-client",
      "version": "1.0.0"
    }
  }
}

// Server Response
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "tools": {"listChanged": true},
      "resources": {}
    },
    "serverInfo": {
      "name": "example-server",
      "version": "1.0.0"
    }
  }
}

// Client Notification
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

### 2. Tool Discovery

```json
// Request
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list"
}

// Response
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "weather_current",
        "title": "Get Current Weather",
        "description": "Retrieves the current weather for a location",
        "inputSchema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City name"
            },
            "units": {
              "type": "string",
              "enum": ["metric", "imperial"],
              "default": "metric"
            }
          },
          "required": ["location"]
        }
      }
    ]
  }
}
```

### 3. Tool Execution

```json
// Request
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "weather_current",
    "arguments": {
      "location": "San Francisco",
      "units": "imperial"
    }
  }
}

// Response
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Current weather in San Francisco: 72°F, Sunny"
      }
    ]
  }
}
```

### 4. Real-time Updates (Notifications)

```json
// Server sends notification when tools change
{
  "jsonrpc": "2.0",
  "method": "notifications/tools/list_changed"
}

// Client responds by requesting updated tool list
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "tools/list"
}
```

## Why Notifications Matter

- **Dynamic Environments**: Tools may change based on server state
- **Efficiency**: Clients don't need to poll for changes
- **Consistency**: Ensures clients have accurate information
- **Real-time Collaboration**: Enables responsive AI applications

## Transport Types

### Stdio Transport (Local)
- Communication via standard input/output streams
- Direct process communication
- Optimal performance
- No network overhead
- Best for local servers

### HTTP Transport (Remote)
- HTTP POST for client-to-server messages
- Server-Sent Events for streaming
- Enables remote server communication
- Supports standard HTTP authentication (OAuth, bearer tokens, API keys)
- Best for hosted services

## Key Design Principles

1. **Protocol Version Negotiation**: Ensures client and server use compatible versions
2. **Capability Discovery**: Each party declares supported features
3. **Identity Exchange**: Provides identification for debugging
4. **Dynamic Updates**: Servers can notify clients of changes
5. **Type Safety**: JSON Schema validation for inputs

## MCP in AI Applications

### Initialization Flow
```python
# Pseudo-code
async with stdio_client(server_config) as (read, write):
    async with ClientSession(read, write) as session:
        init_response = await session.initialize()
        if init_response.capabilities.tools:
            app.register_mcp_server(session, supports_tools=True)
        app.set_server_ready(session)
```

### Tool Discovery
```python
available_tools = []
for session in app.mcp_server_sessions():
    tools_response = await session.list_tools()
    available_tools.extend(tools_response.tools)
conversation.register_available_tools(available_tools)
```

### Tool Execution
```python
async def handle_tool_call(conversation, tool_name, arguments):
    session = app.find_mcp_session_for_tool(tool_name)
    result = await session.call_tool(tool_name, arguments)
    conversation.add_tool_result(result.content)
```

### Notification Handling
```python
async def handle_tools_changed_notification(session):
    tools_response = await session.list_tools()
    app.update_available_tools(session, tools_response.tools)
    if app.conversation.is_active():
        app.conversation.notify_llm_of_new_capabilities()
```

## What MCP Can Enable

- **Personal AI Assistants**: Connect to Google Calendar, Notion, etc.
- **Code Generation**: Generate web apps from Figma designs
- **Enterprise Chatbots**: Connect to multiple databases
- **Creative Tools**: Create 3D designs, control hardware
- **Data Analysis**: Query and analyze data across systems

## Why MCP Matters

- **For Developers**: Reduces complexity when building AI applications
- **For AI Applications**: Provides access to ecosystem of data and tools
- **For End-Users**: Results in more capable AI that can access data and take actions

## Resources

- [MCP Specification](https://modelcontextprotocol.io/specification/latest)
- [MCP SDKs](https://modelcontextprotocol.io/docs/sdk)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [Reference Server Implementations](https://github.com/modelcontextprotocol/servers)
