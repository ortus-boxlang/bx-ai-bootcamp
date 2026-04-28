# Lección 7: Cargadores y Documentos

**⏱️ Duración: 60 minutos**

¡Bienvenido a los cargadores de documentos! En esta lección, aprenderás cómo importar contenido desde varias fuentes (archivos, directorios, URLs, bases de datos) y prepararlo para procesamiento de IA, incluyendo la construcción de un sistema RAG (Generación Aumentada por Recuperación) simple.

## 🎯 Lo que Aprenderás

- Cargar documentos desde archivos, directorios y URLs
- Trabajar con diferentes formatos de archivo (texto, markdown, PDF, CSV, JSON)
- Dividir documentos grandes para procesamiento de IA
- Ingerir documentos en memoria vectorial
- Construir un sistema RAG simple para IA consciente del contexto

---

## 📚 Parte 1: Entendiendo los Cargadores de Documentos (10 mins)

### ¿Qué es un Cargador de Documentos?

Un cargador de documentos importa contenido desde fuentes externas y lo convierte en un formato `Document` estandarizado que la IA puede procesar.

```
┌─────────────────────────────────────────────────────────────────┐
│              FLUJO DE CARGA DE DOCUMENTOS                       │
└─────────────────────────────────────────────────────────────────┘

  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐
  │ Fuente  │─────▶│ Cargador│─────▶│Objetos  │─────▶│Proceso  │
  │(Archivos│      │         │      │Document │      │   IA    │
  └─────────┘      └─────────┘      └─────────┘      └─────────┘
       │                                  │
       ▼                                  ▼
  • Archivos de texto               • Formato estandarizado
  • PDFs                            • Con metadatos
  • Páginas web                     • Listo para dividir
  • Bases de datos                  • Embebible


  💡 Piensa en los cargadores como "adaptadores de importación" para IA
```

### Estructura del Documento

Cada documento cargado tiene estas propiedades:

```javascript
{
    id: "unique-doc-id",           // Identificador único
    content: "El texto actual...",  // Contenido del documento
    metadata: {                     // Información adicional
        source: "/ruta/al/archivo.txt",
        type: "text",
        createdDate: "2024-01-15",
        custom: "cualquier valor"
    },
    embedding: []                   // Vector embedding (opcional)
}
```

### ¿Por qué Cargadores de Documentos?

- ✅ **Interfaz Unificada** - Carga cualquier fuente de la misma manera
- ✅ **Preservación de Metadatos** - Mantén registro de fuente, fechas, etc.
- ✅ **Listo para RAG** - Perfecto para generación aumentada por recuperación
- ✅ **Procesamiento por Lotes** - Maneja directorios completos a la vez
- ✅ **Integración de Memoria** - Ingesta directa en almacenes vectoriales

---

## 💻 Parte 2: Carga Básica de Archivos (15 mins)

### El BIF aiDocuments()

```javascript
// Cargar un solo archivo de texto
docs = aiDocuments( "/ruta/al/archivo.txt" ).load()

// Cargar con tipo específico
docs = aiDocuments( "/ruta/al/archivo.md", { type: "markdown" } ).load()

// Cargar desde URL
docs = aiDocuments( "https://ejemplo.com/pagina.html" ).load()
```

### Ejemplo 1: Cargar un Archivo de Texto

**Crear un documento de muestra (`conocimiento.txt`):**
```
BoxLang es un lenguaje JVM dinámico moderno.
Se ejecuta en la Máquina Virtual de Java.
BoxLang soporta tanto programación orientada a objetos como funcional.
```

**Cargar y usarlo:**
```javascript
// 01-cargar-archivo-texto.bxs

// Cargar el documento
docs = aiDocuments( "conocimiento.txt" ).load()

println( "Cargado #arrayLen( docs )# documento(s)" )

// Acceder a las propiedades del documento
doc = docs[ 1 ]
println( "Contenido: #doc.content#" )
println( "Fuente: #doc.metadata.source#" )

// Usar con IA
result = aiChat(
    "Basado en este contexto: #doc.content#\n\n¿Qué es BoxLang?",
    {},
    { provider: "openai" }
)

println( "\nRespuesta de IA:" )
println( result )
```

**Ejecutar:**
```bash
boxlang 01-cargar-archivo-texto.bxs
# Cargado 1 documento(s)
# Contenido: BoxLang es un lenguaje JVM dinámico moderno...
# Fuente: conocimiento.txt
# Respuesta de IA: BoxLang es un lenguaje JVM dinámico moderno...
```

### Ejemplo 2: Cargar Diferentes Tipos de Archivos

```javascript
// 02-cargar-diferentes-tipos.bxs

// Cargar un archivo markdown
mdDocs = aiDocuments( "readme.md", { type: "markdown" } ).load()
println( "Cargado #arrayLen( mdDocs )# documento(s) markdown" )

// Cargar un archivo CSV
csvDocs = aiDocuments( "datos.csv", { type: "csv" } ).load()
println( "Cargado #arrayLen( csvDocs )# documento(s) CSV" )

// Cargar JSON
jsonDocs = aiDocuments( "config.json", { type: "json" } ).load()
println( "Cargado #arrayLen( jsonDocs )# documento(s) JSON" )

// Cada fila de CSV y objeto JSON se convierte en un documento separado
for ( doc in csvDocs ) {
    println( "Fila CSV: #doc.content#" )
}
```

### 🧪 Lab 1: Carga tu Primer Documento (5 mins)

**Tarea:** Carga un archivo markdown y pide a la IA que lo resuma.

**Pasos:**
1. Crea un archivo markdown con algo de contenido (receta, tutorial, notas)
2. Cárgalo usando `aiDocuments()`
3. Pasa el contenido a `aiChat()` para resumir
4. Imprime el resumen

**Código de inicio en `labs/lab-01-cargar-y-resumir.bxs`**

---

## 📁 Parte 3: Carga de Directorios (10 mins)

### Cargar Directorios Completos

```javascript
// Cargar todos los archivos de un directorio
docs = aiDocuments( "/ruta/a/carpeta" ).load()

// Cargar recursivamente (subdirectorios)
docs = aiDocuments( "/ruta/a/carpeta" )
    .recursive()
    .load()

// Filtrar por extensiones de archivo
docs = aiDocuments( "/ruta/a/carpeta" )
    .recursive()
    .extensions( [ "md", "txt" ] )
    .load()
```

### Ejemplo 3: Base de Conocimiento desde Directorio

**Crear estructura de directorio:**
```
base-conocimiento/
├── info-producto.md
├── faq.txt
└── guias/
    ├── primeros-pasos.md
    └── avanzado.md
```

**Cargar todo:**
```javascript
// 03-cargar-directorio.bxs

docs = aiDocuments( "base-conocimiento" )
    .recursive()
    .extensions( [ "md", "txt" ] )
    .load()

println( "Cargado #arrayLen( docs )# documentos de la base de conocimiento" )

// Mostrar lo que se cargó
for ( doc in docs ) {
    println( "- #doc.metadata.source#" )
}

// Usar todos los documentos para contexto
todoContenido = docs.map( d => d.content ).toList( "\n\n" )

result = aiChat(
    "Basado en esta base de conocimiento:\n\n#todoContenido#\n\n¿Qué productos ofrecemos?",
    {},
    { provider: "openai" }
)

println( "\n" & result )
```

### Filtrar Documentos

```javascript
// Filtrar durante la carga
docs = aiDocuments( "/docs" )
    .filter( doc => doc.getContentLength() > 100 )
    .load()

// Filtrar después de cargar
docsGrandes = docs.filter( doc => len( doc.content ) > 500 )
```

---

## ✂️ Parte 4: División de Documentos (10 mins)

### ¿Por qué Dividir?

Los documentos grandes necesitan dividirse en fragmentos más pequeños porque:
- ✅ Los modelos de IA tienen límites de tokens
- ✅ Los embeddings funcionan mejor en contenido enfocado
- ✅ Recuperación más precisa en sistemas RAG

```
┌─────────────────────────────────────────────────────────────────┐
│                  ESTRATEGIA DE DIVISIÓN                         │
└─────────────────────────────────────────────────────────────────┘

  Documento Original (5000 palabras)
  ┌────────────────────────────────────────────────────┐
  │ Capítulo 1: Introducción                          │
  │ Capítulo 2: Primeros Pasos                        │
  │ Capítulo 3: Temas Avanzados                       │
  │ Capítulo 4: Mejores Prácticas                     │
  └────────────────────────────────────────────────────┘
                       │
                       ▼ (dividir con superposición)
         ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
         │Frag. 1 │─▶│Frag. 2 │─▶│Frag. 3 │─▶│Frag. 4 │
         │500 pal.│  │500 pal.│  │500 pal.│  │500 pal.│
         └────────┘  └────────┘  └────────┘  └────────┘
              └─────▶superposic.◀─────┘
                    (50 palabras)

  💡 La superposición ayuda a mantener el contexto entre fragmentos
```

### Configuración de División

```javascript
// Cargar con división
docs = aiDocuments( "/doc-grande.txt" )
    .chunkSize( 500 )        // Caracteres por fragmento
    .overlap( 50 )           // Superposición entre fragmentos
    .load()

// O especificar en config
docs = aiDocuments( "/doc-grande.txt", {
    chunkSize: 500,
    overlap: 50
} ).load()
```

### Ejemplo 4: Dividir un Documento Grande

```javascript
// 04-dividir-documento-grande.bxs

// Crear un documento de muestra grande
contenidoGrande = repeatString( "BoxLang es un poderoso lenguaje JVM. ", 100 )
fileWrite( "doc-grande.txt", contenidoGrande )

// Cargar SIN división
docsSinDividir = aiDocuments( "doc-grande.txt" ).load()
println( "Sin dividir: #arrayLen( docsSinDividir )# documento(s)" )
println( "Longitud del contenido: #len( docsSinDividir[ 1 ].content )#" )

// Cargar CON división
docsDivididos = aiDocuments( "doc-grande.txt" )
    .chunkSize( 200 )
    .overlap( 20 )
    .load()

println( "\nCon división: #arrayLen( docsDivididos )# fragmento(s)" )

// Mostrar cada fragmento
for ( i = 1; i <= arrayLen( docsDivididos ); i++ ) {
    println( "Fragmento #i#: #len( docsDivididos[ i ].content )# caracteres" )
}
```

**Salida:**
```
Sin dividir: 1 documento(s)
Longitud del contenido: 3700

Con división: 20 fragmento(s)
Fragmento 1: 200 caracteres
Fragmento 2: 200 caracteres
...
```

---

## 🧠 Parte 5: Integración de Memoria y RAG (15 mins)

### ¿Qué es RAG?

**RAG (Generación Aumentada por Recuperación)** combina:
1. **Recuperación** - Encuentra documentos relevantes de una base de conocimiento
2. **Aumento** - Agrega esos documentos al prompt de IA
3. **Generación** - IA genera una respuesta usando el contexto

```
┌─────────────────────────────────────────────────────────────────┐
│                      FLUJO DE TRABAJO RAG                       │
└─────────────────────────────────────────────────────────────────┘

  Paso 1: Indexar Documentos        Paso 2: Consultar y Recuperar
  ─────────────────────              ──────────────────────
  ┌──────────┐                       Pregunta del Usuario
  │Documentos│                             │
  │(divididos│                             ▼
  └─────┬────┘                       ┌─────────┐
        │                            │Búsqueda │
        ▼                            │Vectorial│
  ┌──────────┐                       └────┬────┘
  │Embeddings│                            │
  └─────┬────┘                            ▼
        │                            Top 3 Fragmentos Relevantes
        ▼
  ┌──────────┐                       Paso 3: Generar Respuesta
  │ Memoria  │                       ─────────────────────
  │ Vectorial│                       ┌─────────────────┐
  └──────────┘                       │Fragmentos+Consult│
                                     └────────┬────────┘
                                              ▼
                                         ┌─────────┐
                                         │   IA    │
                                         │Respuesta│
                                         └─────────┘
```

### Ingesta Directa a Memoria

El método `toMemory()` carga documentos directamente en memoria vectorial:

```javascript
// Crear memoria vectorial
memory = aiMemory(
    memory: "chroma",
    config: { collectionName: "base-conocimiento" }
)

// Cargar documentos e ingerir a memoria
result = aiDocuments( "/docs" )
    .recursive()
    .extensions( [ "md", "txt" ] )
    .toMemory( memory, { chunkSize: 500, overlap: 50 } )

println( "Documentos cargados: #result.documentsIn#" )
println( "Fragmentos creados: #result.chunksOut#" )
println( "Almacenado en memoria: #result.stored#" )
```

### Ejemplo 5: Construir un Sistema RAG Simple

```javascript
// 05-sistema-rag-simple.bxs

// Paso 1: Crear documentos de base de conocimiento
fileWrite( "docs/producto-a.txt", "Producto A es una herramienta de desarrollo web para desarrolladores BoxLang." )
fileWrite( "docs/producto-b.txt", "Producto B es un framework de pruebas para aplicaciones BoxLang." )
fileWrite( "docs/precios.txt", "Producto A cuesta $49/mes. Producto B cuesta $29/mes." )

// Paso 2: Crear memoria vectorial (embeddings de Ollama - ¡gratis!)
memory = aiMemory(
    memory: "chroma",
    config: {
        collectionName: "productos",
        embeddingProvider: "ollama",
        embeddingModel: "nomic-embed-text"
    }
)

// Paso 3: Cargar e ingerir documentos
println( "Cargando documentos en memoria vectorial..." )
result = aiDocuments( "docs" )
    .recursive()
    .toMemory( memory )

println( "✅ Indexados #result.stored# fragmentos de documentos" )

// Paso 4: Consultar el sistema RAG
function preguntarRAG( pregunta ) {
    // Recuperar documentos relevantes
    resultados = memory.search( pregunta, 3 )

    // Construir contexto desde documentos recuperados
    contexto = resultados.map( r => r.content ).toList( "\n\n" )

    // Preguntar a IA con contexto
    return aiChat(
        "Contexto:\n#contexto#\n\nPregunta: #pregunta#",
        { model: "qwen3:0.6b" },
        { provider: "ollama" }
    )
}

// Paso 5: ¡Hacer preguntas!
println( "\n" & preguntarRAG( "¿Qué productos ofrecen?" ) )
println( "\n" & preguntarRAG( "¿Cuánto cuesta el Producto A?" ) )
println( "\n" & preguntarRAG( "¿Para qué se usa el Producto B?" ) )
```

### 🧪 Lab 2: Construir tu Sistema RAG (10 mins)

**Tarea:** Crear un sistema RAG para un dominio específico (recetas, fragmentos de código, documentos de empresa).

**Requisitos:**
1. Crea 3-5 archivos de texto/markdown con contenido relacionado
2. Cárgalos en memoria vectorial con `toMemory()`
3. Crea una función `consultarConocimiento()` que recupere y use contexto
4. Haz al menos 3 preguntas diferentes

**Código de inicio en `labs/lab-02-construir-sistema-rag.bxs`**

---

## 🌐 Parte 6: Carga Avanzada (5 mins)

### Cargar desde URLs

```javascript
// Cargar página web
docs = aiDocuments( "https://boxlang.io/docs" ).load()

// Cargar con encabezados personalizados
docs = aiDocuments( "https://api.ejemplo.com/datos", {
    type: "http",
    headers: { "Authorization": "Bearer token" }
} ).load()
```

### Cargar desde Bases de Datos

```javascript
// Cargar desde consulta SQL
docs = aiDocuments( config: {
    type: "sql",
    datasource: "miDB",
    sql: "SELECT titulo, contenido, fecha_creacion FROM articulos",
    contentColumn: "contenido",
    metadataColumns: [ "titulo", "fecha_creacion" ]
} ).load()
```

### Carga Asíncrona

```javascript
// Cargar asíncronamente para conjuntos de datos grandes
future = aiDocuments( "/conjunto-datos-enorme" ).loadAsync()

// Hacer otro trabajo...
println( "Cargando en segundo plano..." )

// Esperar finalización
docs = future.get()
println( "Cargado #arrayLen( docs )# documentos" )
```

---

## 📊 Parte 7: Ejemplo RAG Completo (5 mins)

### Ejemplo 6: Base de Conocimiento de Soporte al Cliente

```javascript
// 06-rag-soporte-cliente.bxs

// Crear base de conocimiento de soporte completa
fileWrite( "bc/devoluciones.md", "# Devoluciones\nLos clientes pueden devolver artículos dentro de 30 días." )
fileWrite( "bc/envio.md", "# Envío\nOfrecemos envío gratis en pedidos superiores a $50." )
fileWrite( "bc/garantia.md", "# Garantía\nTodos los productos tienen garantía de 1 año." )

// Configurar memoria vectorial
memory = aiMemory(
    memory: "chroma",
    config: {
        collectionName: "soporte-cliente",
        embeddingProvider: "ollama"
    }
)

// Ingerir base de conocimiento
result = aiDocuments( "bc" )
    .recursive()
    .extensions( [ "md" ] )
    .toMemory( memory, { chunkSize: 300 } )

println( "Base de conocimiento lista: #result.stored# fragmentos indexados" )

// Crear agente de IA con RAG
agentesoporte = aiAgent(
    memory: memory
).withInstructions( "Eres un agente de soporte al cliente útil. Usa la base de conocimiento para responder preguntas con precisión." )

// Manejar preguntas de clientes
preguntas = [
    "¿Cuál es su política de devoluciones?",
    "¿Ofrecen envío gratis?",
    "¿Cuánto dura la garantía?"
]

for ( pregunta in preguntas ) {
    println( "\nP: #pregunta#" )
    println( "R: #agentesoporte.run( pregunta )#" )
}
```

---

## 🎯 Resumen

### Lo que Aprendiste

✅ **Cargadores de Documentos** - Cargar archivos, directorios, URLs, bases de datos
✅ **Formatos de Archivo** - Texto, markdown, PDF, CSV, JSON, XML
✅ **Carga de Directorios** - Carga recursiva con filtros
✅ **División** - Dividir documentos grandes inteligentemente
✅ **Integración de Memoria** - Usar `toMemory()` para RAG
✅ **Sistemas RAG** - Flujo de trabajo de Generación Aumentada por Recuperación

### BIFs Clave

| BIF | Propósito |
|-----|-----------|
| `aiDocuments()` | Crear cargador de documentos |
| `aiMemory()` | Crear memoria (incluyendo memoria vectorial) |

### Próximos Pasos

Ahora que puedes cargar y procesar documentos, estás listo para:
- **Lección 8** - Servidores y Clientes MCP
- **Temas Avanzados** - Cargadores personalizados, transformaciones, sistemas multi-memoria

---

## 📝 Recursos

- [Documentación de Cargadores de Documentos](https://ai.ortusbooks.com/rag/document-loaders)
- [Documentación de Memoria Vectorial](https://ai.ortusbooks.com/main-components/vector-memory)
- [Documentación de RAG](https://ai.ortusbooks.com/main-components/rag)

---

**🎉 ¡Felicitaciones!** ¡Ahora puedes construir sistemas RAG con BoxLang AI!

👉 **[Continuar a Lección 8: Servidores y Clientes MCP](../lesson-08-mcp-servers-clients/)**
