# Lesson 8: MCP Servers and Clients

[Home](../README.md)

**Duration: 60 minutes**

This lesson teaches the practical MCP workflow in BoxLang AI without extra complexity.

## What You'll Learn

- What MCP is and when to use it
- How to call remote MCP tools with `MCP()`
- How to expose local tools with `MCPServer()`
- How to plug MCP servers into agents
- Basic production hardening

---

## Step 1: MCP Mental Model

MCP is a standard protocol for AI tools.

- `MCP()` is your client ( consume remote tools )
- `MCPServer()` is your server ( publish local tools )

Think: JSON-RPC tool calls over HTTP/STDIO.

```text
MCP Mental Model

    [ AI App ]
            |
            |  MCP() client calls
            v
    +-------------------+         JSON-RPC         +-------------------+
    |   MCP Client      |  ---------------------->  |    MCP Server     |
    |   (consumer)      |  <----------------------  |    (publisher)    |
    +-------------------+                           +-------------------+
                                                                                                                |
                                                                                                                | registerTool()
                                                                                                                v
                                                                                                     [ aiTool(...) ]
```

---

## Step 2: MCP Client Basics

```javascript
client = MCP( "http://localhost:3000" )

// Discover
listResponse = client.listTools()
if ( listResponse.isSuccess() ) {
    println( listResponse.getData() )
} else {
    println( listResponse.getError() )
}

// Execute a remote tool
weatherResponse = client.send( "getWeather", { city: "Boston" } )
if ( weatherResponse.isSuccess() ) {
    println( weatherResponse.getData() )
}
```

Useful client methods:

- `listTools()`
- `listResources()`
- `listPrompts()`
- `send( toolName, args )`
- `readResource( uri )`
- `getPrompt( name, args )`

Optional client config:

```javascript
client = MCP( "http://localhost:3000" )
    .withTimeout( 5000 )
    .withBearerToken( "token" )
    .withHeaders( { "X-App": "bootcamp" } )
```

```text
Client Request Lifecycle

    listTools()/send() --> MCPClient.makeRequest() --> HTTP POST --> MCPServer.handleRequest()
                                                                                                                                        |
                                                                                                                                        +--> tools/list
                                                                                                                                        +--> tools/call
                                                                                                                                        +--> resources/list/read
                                                                                                                                        +--> prompts/list/get
```

---

## Step 3: Build a Local MCP Server

```javascript
mcpSrv = MCPServer( "mathServer", "Math tools for demo", "1.0.0" )

calculateTool = aiTool(
    "calculate",
    "Perform basic math operations",
    ( required string operation, required numeric a, required numeric b ) => {
        switch ( operation ) {
            case "add":
                return a + b
            case "subtract":
                return a - b
            case "multiply":
                return a * b
            case "divide":
                return b == 0 ? "Cannot divide by zero" : a / b
            default:
                return "Unknown operation: #operation#"
        }
    }
)
.describeOperation( "add, subtract, multiply, divide" )
.describeA( "First number" )
.describeB( "Second number" )

mcpSrv.registerTool( calculateTool )

println( "Registered tools: #mcpSrv.getToolCount()#" )
println( mcpSrv.listTools() )
```

```text
Server Composition

    MCPServer("mathServer")
                    |
                    +-- registerTool( calculateTool )
                    |
                    +-- listTools()
                    |
                    +-- handleRequest( jsonRpcBody )
```

---

## Step 4: Expose the Server via HTTP

In your endpoint file:

```javascript
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Handles MCP JSON-RPC requests and routes to registered servers
MCPRequestProcessor::startHttp()
</bx:script>
```

---

## Step 5: Agent + MCP Integration

Two good patterns:

1. Pass MCP servers at creation time

```javascript
agent = aiAgent(
    name: "Helper",
    instructions: "Use MCP tools when useful.",
    mcpServers: [ "http://localhost:3000" ]
)
```

2. Add later with fluent API

```javascript
agent = aiAgent( name: "Helper" )
    .withMCPServer( "http://localhost:3000" )

println( agent.run( "Use available tools to answer this question" ) )
```

```text
Agent + MCP Flow

  User Prompt
      |
      v
  aiAgent.run()
      |
      +--> decides a tool is needed
      |
      +--> withMCPServer("http://localhost:3000")
              |
              v
          remote MCP tool call
              |
              v
          tool result -> model -> final response
```

---

## Step 6: Production Hardening

```javascript
mcpSrv = MCPServer( "secureServer" )
    .withBasicAuth( "admin", "secret" )
    .withBodyLimit( 1024 * 1024 )
    .withCors( [ "https://myapp.com" ] )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        return apiKey == "my-api-key"
    } )
```

---

## Quick Recap

- `MCP()` consumes remote MCP capabilities
- `MCPServer()` publishes local tools/resources/prompts
- `aiTool( name, description, callable )` is the correct tool pattern
- Agents can use MCP directly via `mcpServers` or `.withMCPServer()`

---

## References

- https://ai.ortusbooks.com/advanced/mcp-client
- https://ai.ortusbooks.com/advanced/mcp-server
- https://modelcontextprotocol.io/

👉 **[Continue to Lesson 9: Audio](../lesson-09-audio/README.md)**