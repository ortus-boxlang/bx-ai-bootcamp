# Lección 4: Salida Estructurada

**⏱️ Duración: 60 minutos**

Obtener texto de la IA es genial, pero ¿qué pasa si necesitas **datos estructurados**? En esta lección, aprenderás a extraer objetos, arrays y estructuras de datos complejas con tipos seguros de las respuestas de IA.

## 🎯 Lo que Aprenderás

- Extraer datos estructurados de las respuestas de IA
- Usar clases de BoxLang para salida con tipos seguros
- Usar plantillas de struct para extracción rápida
- Extraer arrays de objetos
- Validar y transformar salida de IA

---

## 📚 Parte 1: ¿Por Qué Salida Estructurada? (10 mins)

### El Problema

La IA devuelve texto, pero a menudo necesitas datos:

```
┌─────────────────────────────────────────────────────────────────┐
│                   EL PROBLEMA                                   │
└─────────────────────────────────────────────────────────────────┘

  Tu Pregunta:
  "Extrae el nombre y edad de: Juan tiene 30 años"

  Respuesta de IA (Texto):
  "El nombre de la persona es Juan y su edad es 30."

  Lo que Realmente Necesitas:
  { name: "Juan", age: 30 }    ← ¡DATOS ESTRUCTURADOS!
```

### La Solución

La **salida estructurada** de BoxLang AI convierte las respuestas de IA en datos apropiados:

```
┌─────────────────────────────────────────────────────────────────┐
│                   LA SOLUCIÓN                                   │
└─────────────────────────────────────────────────────────────────┘

  Entrada de Texto                Salida Estructurada
  ───────────────                 ─────────────────
       │                              │
       ▼                              ▼
  ┌─────────┐     ┌─────────┐   ┌─────────────┐
  │ "Juan   │ ──▶ │   IA    │──▶│ { name:     │
  │ tiene   │     │ + Schema│   │   "Juan",   │
  │ 30 años"│     │         │   │   age: 30 } │
  └─────────┘     └─────────┘   └─────────────┘
```

### Beneficios

- ✅ **Tipos Seguros** - Obtén tipos de datos apropiados (strings, números, arrays)
- ✅ **Validación** - El schema asegura que los campos requeridos existan
- ✅ **Sin Parsing** - No se necesita regex ni manipulación de strings
- ✅ **Confiabilidad** - Formato de salida consistente

---

## 💻 Parte 2: Usando Plantillas de Struct (15 mins)

La forma más fácil de obtener salida estructurada es con una **plantilla de struct**.

### Plantilla de Struct Básica

```java
// plantilla-struct.bxs
// Define la estructura que quieres
plantilla = {
    "name": "",
    "age": 0,
    "email": ""
}

// Pide a la IA que extraiga en esta estructura
// Pasa la plantilla como returnFormat para que la IA la rellene directamente
resultado = aiChat(
    "Extract person info: John Doe is 30 years old, email john@example.com",
    {},
    { returnFormat: plantilla }
)

// ¡Usa el resultado como un struct normal!
println( "Nombre: " & resultado.name )
println( "Edad: " & resultado.age )
println( "Email: " & resultado.email )
```

### Cómo Funciona

```
┌─────────────────────────────────────────────────────────────────┐
│              FLUJO DE PLANTILLA DE STRUCT                       │
└─────────────────────────────────────────────────────────────────┘

  1. Plantilla         2. IA Extrae          3. Obtienes Datos
  ───────────          ──────────────        ───────────────

  {                    "John Doe tiene 30     {
    name: "",          años..."                 name: "John Doe",
    age: 0,                 │                   age: 30,
    email: ""               ▼                   email: "john@..."
  }                    ¡La IA entiende       }
                       la estructura!
```

### Más Ejemplos de Plantillas

**Información de Producto:**
```java
plantillaProducto = {
    "name": "",
    "price": 0.0,
    "inStock": true,
    "category": ""
}

producto = aiChat(
    "Product: MacBook Pro, $2499, available, category: Electronics",
    {},
    { returnFormat: plantillaProducto }
)

println( "#producto.name# - $#producto.price#" )
```

**Dirección:**
```java
plantillaDireccion = {
    "street": "",
    "city": "",
    "state": "",
    "zip": "",
    "country": ""
}

direccion = aiChat(
    "Parse: 123 Main St, Boston, MA 02101, USA",
    {},
    { returnFormat: plantillaDireccion }
)
```

---

## 🏗️ Parte 3: Usando Clases (15 mins)

Para salida estructurada reutilizable, define una **clase de BoxLang**.

### Define una Clase

```java
// Persona.bx
class {
    property name="name" type="string";
    property name="age" type="numeric";
    property name="email" type="string";
}
```

### Usa la Clase

```java
// extraccion-clase.bxs
// Extrae en la clase Persona
// Pasa la instancia de clase como returnFormat para que la IA la rellene directamente
resultado = aiChat(
    "Extract: Sarah Connor, 35, sarah@resistance.org",
    {},
    { returnFormat: new Persona() }
)

// ¡Usa getters!
println( "Nombre: " & resultado.getName() )
println( "Edad: " & resultado.getAge() )
println( "Email: " & resultado.getEmail() )
```

### Ejemplo de Clase Compleja

```java
// Orden.bx
class {
    property name="orderId" type="string";
    property name="customerName" type="string";
    property name="total" type="numeric";
    property name="items" type="array";
    property name="status" type="string";
}
```

```java
// Extraer orden del texto
orden = aiChat(
    "
    Order #12345 for Jane Smith
    Total: $156.99
    Items: Widget (x2), Gadget (x1)
    Status: Shipped
    ",
    {},
    { returnFormat: new Orden() }
)

println( "Orden #orden.getOrderId()# - $#orden.getTotal()#" )
```

### Clase vs Struct: Cuándo Usar

| Usar Plantillas de Struct | Usar Clases |
|---------------------------|-------------|
| Extracción rápida, única | Estructuras de datos reutilizables |
| Datos simples y planos | Datos anidados complejos |
| Prototipado | Código de producción |
| Consultas ad-hoc | Entidades definidas |

---

## 📚 Parte 4: Extrayendo Arrays (10 mins)

¿Necesitas extraer **múltiples elementos**? ¡Usa sintaxis de array!

### Array de Structs

```java
// extraccion-array.bxs
// Plantilla para UNA tarea
plantillaTarea = {
    "title": "",
    "priority": "",
    "done": false
}

// Envuelve en array para extraer MÚLTIPLES tareas
tareas = aiChat(
    "
    Extract tasks:
    1. Buy groceries (high priority, not done)
    2. Call mom (medium, done)
    3. Exercise (low, not done)
    ",
    {},
    { returnFormat: [ plantillaTarea ] }
)

// Recorre los resultados
for( tarea in tareas ) {
    estado = tarea.done ? "✅" : "⬜"
    println( "#estado# #tarea.title# [#tarea.priority#]" )
}
```

**Salida:**
```
⬜ Buy groceries [high]
✅ Call mom [medium]
⬜ Exercise [low]
```

### Array de Clases

```java
// Producto.bx
class {
    property name="name" type="string";
    property name="price" type="numeric";
}
```

```java
// Extraer múltiples productos
productos = aiChat(
    "
    Parse products:
    - iPhone 15: $999
    - AirPods Pro: $249
    - MacBook Air: $1299
    ",
    {},
    { returnFormat: [ new Producto() ] }
)

total = 0
for( producto in productos ) {
    println( "#producto.getName()#: $#producto.getPrice()#" )
    total += producto.getPrice()
}
println( "─".repeat( 30 ) )
println( "Total: $#total#" )
```

---

## 🧪 Parte 5: Laboratorio - Parser de Factura (10 mins)

### El Objetivo

Construir un parser de facturas que extraiga:
- Número de factura
- Fecha
- Nombre del cliente
- Líneas de artículos (array)
- Monto total

### Texto de Factura

```
FACTURA #INV-2024-001

Fecha: 15 de Noviembre, 2024
Facturar a: Acme Corporation

Artículos:
  - Desarrollo Web: $5,000
  - Servicios de Diseño: $2,500
  - Hosting (1 año): $500

Total: $8,000
```

### Solución

```java
// parser-factura.bxs

// Definir estructura de línea de artículo
plantillaLinea = {
    "description": "",
    "amount": 0.0
}

// Definir estructura de factura
plantillaFactura = {
    "invoiceNumber": "",
    "date": "",
    "customerName": "",
    "lineItems": [ plantillaLinea ],
    "total": 0.0
}

// Texto de factura de ejemplo
textoFactura = "
INVOICE #INV-2024-001

Date: November 15, 2024
Bill To: Acme Corporation

Items:
  - Web Development: $5,000
  - Design Services: $2,500
  - Hosting (1 year): $500

Total: $8,000
"

// Extraer datos estructurados
factura = aiChat(
    "Parse this invoice: " & textoFactura,
    {},
    { returnFormat: plantillaFactura }
)

// Mostrar resultados
println( "📄 Resultados del Parser de Facturas" )
println( "═".repeat( 40 ) )
println()
println( "Factura #: " & factura.invoiceNumber )
println( "Fecha: " & factura.date )
println( "Cliente: " & factura.customerName )
println()
println( "Líneas de Artículos:" )

totalCalculado = 0
for( item in factura.lineItems ) {
    println( "  • #item.description#: $#numberFormat( item.amount, ',.00' )#" )
    totalCalculado += item.amount
}

println()
println( "─".repeat( 40 ) )
println( "Total: $#numberFormat( factura.total, ',.00' )#" )

// Verificar total (con precisión de centavos para comparaciones monetarias)
if( abs( totalCalculado - factura.total ) < 0.01 ) {
    println( "✅ ¡Total verificado!" )
} else {
    println( "⚠️ ¡Discrepancia de total! Calculado: $#numberFormat( totalCalculado, ',.00' )#" )
}
```

### Salida Esperada

```
📄 Resultados del Parser de Facturas
════════════════════════════════════════

Factura #: INV-2024-001
Fecha: November 15, 2024
Cliente: Acme Corporation

Líneas de Artículos:
  • Web Development: $5,000.00
  • Design Services: $2,500.00
  • Hosting (1 year): $500.00

────────────────────────────────────────
Total: $8,000.00
✅ ¡Total verificado!
```

---

## ✅ Verificación de Conocimientos

1. **¿Qué devuelve `returnFormat` con un struct/clase?**
   - [ ] Un string
   - [x] Un struct u objeto que coincide con tu plantilla
   - [ ] Texto JSON
   - [ ] Un error

2. **¿Cómo extraes un array de elementos?**
   - [ ] Usar extractArray()
   - [x] Envolver tu plantilla en [ ] y pasarla como returnFormat
   - [ ] Llamar aiChat() múltiples veces
   - [ ] Usar un bucle

3. **¿Cuándo deberías usar una clase en lugar de struct?**
   - [ ] Nunca
   - [x] Para estructuras de datos reutilizables y complejas
   - [ ] Solo para arrays
   - [ ] Cuando uses Ollama

4. **¿Qué hace confiable la salida estructurada?**
   - [x] La IA sigue un schema para asegurar formato correcto
   - [ ] Usa servidores especiales
   - [ ] Requiere Claude
   - [ ] Solo funciona con números

---

## 📝 Resumen

Aprendiste:

| Concepto | Descripción |
|----------|-------------|
| **Plantilla de Struct** | Extracción estructurada rápida con `{}` |
| **Clase** | Extracción reutilizable con tipos seguros |
| **Arrays** | Extraer múltiples elementos con `[ ]` |
| **returnFormat** | Opción para extraer datos estructurados (struct, clase o array) |

### Patrones de Código Clave

```java
// Plantilla de struct
resultado = aiChat( "texto", {}, { returnFormat: { name: "", age: 0 } } )

// Clase
resultado = aiChat( "texto", {}, { returnFormat: new Persona() } )

// Array de structs
resultados = aiChat( "texto", {}, { returnFormat: [ { item: "" } ] } )

// Array de clases
resultados = aiChat( "texto", {}, { returnFormat: [ new Tarea() ] } )
```

---

## ⏭️ Siguiente Lección

¡Ahora puedes extraer datos estructurados! Aprendamos a darle a la IA acceso a funciones del mundo real.

👉 **[Lección 5: Herramientas de IA](../lesson-05-tools/)**

---

## 📁 Archivos de la Lección

```
lesson-04-structured-output/
├── README.md (este archivo)
├── examples/
│   ├── plantilla-struct.bxs
│   ├── extraccion-clase.bxs
│   ├── extraccion-array.bxs
│   └── Persona.bx
└── labs/
    └── parser-factura.bxs
```
