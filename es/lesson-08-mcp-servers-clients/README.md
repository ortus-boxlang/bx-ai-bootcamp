# Leccion 8: Servidores y Clientes MCP

**Duracion: 60 minutos**

Esta leccion cubre MCP en BoxLang AI con un flujo practico y directo.

## Que Aprenderas

- Que es MCP y cuando usarlo
- Como consumir herramientas remotas con `MCP()`
- Como publicar herramientas locales con `MCPServer()`
- Como integrar servidores MCP en agentes
- Endurecimiento basico para produccion

---

## Paso 1: Modelo Mental MCP

MCP es un protocolo estandar para herramientas de IA.

- `MCP()` es el cliente (consume herramientas remotas)
- `MCPServer()` es el servidor (publica herramientas locales)

Piensalo como JSON-RPC para herramientas de IA.

```text
Modelo Mental MCP

  [ App de IA ]
      |
      | llamadas con MCP()
      v
  +-------------------+         JSON-RPC         +-------------------+
  |   Cliente MCP     |  ---------------------->  |   Servidor MCP    |
  |   (consumidor)    |  <----------------------  |   (publicador)    |
  +-------------------+                           +-------------------+
                                              |
                                              | registerTool()
                                              v
                                          [ aiTool(...) ]
```

---

## Paso 2: Cliente MCP Basico

```javascript
client = MCP( "http://localhost:3000" )

// Descubrir herramientas
listResponse = client.listTools()
if ( listResponse.isSuccess() ) {
    println( listResponse.getData() )
} else {
    println( listResponse.getError() )
}

// Ejecutar herramienta remota
weatherResponse = client.send( "getWeather", { city: "Bogota" } )
if ( weatherResponse.isSuccess() ) {
    println( weatherResponse.getData() )
}
```

Metodos utiles del cliente:

- `listTools()`
- `listResources()`
- `listPrompts()`
- `send( toolName, args )`
- `readResource( uri )`
- `getPrompt( name, args )`

Configuracion opcional:

```javascript
client = MCP( "http://localhost:3000" )
    .withTimeout( 5000 )
    .withBearerToken( "token" )
    .withHeaders( { "X-App": "bootcamp" } )
```

```text
Ciclo de Peticion del Cliente

    listTools()/send() --> MCPClient.makeRequest() --> HTTP POST --> MCPServer.handleRequest()
                                                                                                                                         |
                                                                                                                                         +--> tools/list
                                                                                                                                         +--> tools/call
                                                                                                                                         +--> resources/list/read
                                                                                                                                         +--> prompts/list/get
```

---

## Paso 3: Crear un Servidor MCP Local

```javascript
mcpSrv = MCPServer( "mathServer", "Herramientas matematicas", "1.0.0" )

calculateTool = aiTool(
    "calculate",
    "Realiza operaciones matematicas basicas",
    ( required string operation, required numeric a, required numeric b ) => {
        switch ( operation ) {
            case "add":
                return a + b
            case "subtract":
                return a - b
            case "multiply":
                return a * b
            case "divide":
                return b == 0 ? "No se puede dividir por cero" : a / b
            default:
                return "Operacion desconocida: #operation#"
        }
    }
)
.describeOperation( "add, subtract, multiply, divide" )
.describeA( "Primer numero" )
.describeB( "Segundo numero" )

mcpSrv.registerTool( calculateTool )

println( "Herramientas registradas: #mcpSrv.getToolCount()#" )
println( mcpSrv.listTools() )
```

```text
Composicion del Servidor

    MCPServer("mathServer")
                    |
                    +-- registerTool( calculateTool )
                    |
                    +-- listTools()
                    |
                    +-- handleRequest( jsonRpcBody )
```

---

## Paso 4: Exponer el Servidor por HTTP

En tu endpoint:

```javascript
<bx:script>
import bxModules.bxai.models.mcp.MCPRequestProcessor

// Procesa solicitudes MCP JSON-RPC y enruta al servidor registrado
MCPRequestProcessor::startHttp()
</bx:script>
```

---

## Paso 5: Integracion Agente + MCP

Dos patrones recomendados:

1. Pasar servidores MCP al crear el agente

```javascript
agent = aiAgent(
    name: "Asistente",
    instructions: "Usa herramientas MCP cuando sea util.",
    mcpServers: [ "http://localhost:3000" ]
)
```

2. Agregar despues con API fluida

```javascript
agent = aiAgent( name: "Asistente" )
    .withMCPServer( "http://localhost:3000" )

println( agent.run( "Usa las herramientas disponibles para responder" ) )
```

```text
Flujo Agente + MCP

  Prompt del usuario
       |
       v
  aiAgent.run()
       |
       +--> decide que necesita una herramienta
       |
       +--> withMCPServer("http://localhost:3000")
               |
               v
           llamada a herramienta MCP remota
               |
               v
           resultado -> modelo -> respuesta final
```

---

## Paso 6: Endurecimiento para Produccion

```javascript
mcpSrv = MCPServer( "secureServer" )
    .withBasicAuth( "admin", "secret" )
    .withBodyLimit( 1024 * 1024 )
    .withCors( [ "https://miapp.com" ] )
    .withApiKeyProvider( ( apiKey, requestData ) => {
        return apiKey == "mi-api-key"
    } )
```

---

## Resumen Rapido

- `MCP()` consume capacidades MCP remotas
- `MCPServer()` publica herramientas/recursos/prompts locales
- `aiTool( name, description, callable )` es el patron correcto
- Los agentes usan MCP con `mcpServers` o `.withMCPServer()`

---

## Referencias

- https://ai.ortusbooks.com/advanced/mcp-client
- https://ai.ortusbooks.com/advanced/mcp-server
- https://modelcontextprotocol.io/
