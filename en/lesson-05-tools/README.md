# Lesson 5: AI Tools

[Home](../README.md)

**⏱️ Duration: 60 minutes**

So far, AI has only used its training knowledge. But what if AI could **call your functions** to get real-time data, perform calculations, or interact with external systems? That's what **tools** enable!

## 🎯 What You'll Learn

- Understand AI function calling
- Create tools with `aiTool()`
- Describe tool arguments
- Let AI use multiple tools
- Handle tool results

---

## 📚 Part 1: What Are AI Tools? (10 mins)

### The Problem

AI knowledge is frozen at training time. It can't:

- Check today's weather
- Look up live data
- Access your database
- Perform real calculations

### The Solution: Tools

**Tools** are functions that AI can call when it needs information:

```
┌─────────────────────────────────────────────────────────────────┐
│                      HOW TOOLS WORK                             │
└─────────────────────────────────────────────────────────────────┘

  User: "What's the weather in Boston?"
         │
         ▼
  ┌──────────────┐
  │     AI       │  "I need weather data..."
  │   thinks     │         │
  └──────────────┘         │
         │                 ▼
         │        ┌───────────────┐
         │        │  TOOL CALL    │
         │        │ get_weather() │
         │        │ city="Boston" │
         │        └───────────────┘
         │                 │
         │                 ▼
         │        ┌───────────────┐
         │        │ Your Function │
         │        │ Returns: 72°F │
         │        └───────────────┘
         │                 │
         ▼                 ▼
  ┌──────────────────────────────┐
  │  AI: "It's 72°F in Boston!" │
  └──────────────────────────────┘
```

### Why Tools Matter

- ✅ **Real-time data** - Weather, stocks, news
- ✅ **Calculations** - Math, conversions, statistics
- ✅ **Database access** - Look up user info, products
- ✅ **External APIs** - Send emails, make reservations
- ✅ **Custom logic** - Your business rules

---

## 💻 Part 2: Creating Your First Tool (15 mins)

### The aiTool() Function

```java
tool = aiTool(
    "tool_name",           // Name AI will use to call it
    "Tool description",    // Explains when to use it
    ( args ) => {          // Function to execute
        // Your code here
        return result
    }
)
```

### Example: Calculator Tool

```java
// calculator-tool.bxs
// Create a calculator tool that AI can use
// Run with: boxlang calculator-tool.bxs

println( "🧮 Calculator Tool Demo" )
println( "─".repeat( 40 ) )
println()

calculatorTool = aiTool(
    "calculator",
    "Performs mathematical calculations. Use for any math operations.",
    ( args ) => {
        // Parse the expression and calculate
        //expression = args.expression
        try {
            println( "Using the tool to calculate: " & args )
            result = evaluate( args )
            return "Result: " & result
        } catch( any e ) {
            return "Error calculating: " & e.message
        }
    }
).describeExpression( "The math expression to calculate, e.g. '2 + 2' or '100 * 0.15'" )

// Test questions that require calculation
questions = [
    "What is 15% of 200?",
    "Calculate 45 + 73",
    "What's 1000 divided by 8?"
]

for( question in questions ) {
    println( "Q: " & question )
    answer = aiChat( question, { tools: [ calculatorTool ] } )
    println( "A: " & answer )
    println()
}
```
Notice the line `println( "Using the tool to calculate: " & args )`. Not every problem the provider uses will have that output indicating that sometimes it can ( or thinks it can ) answer the question with just its existing training.


### Example: Weather Tool

```java
// weather-tool.bxs
// Create a weather lookup tool
// Run with: boxlang weather-tool.bxs

println( "🌤️ Weather Tool Demo" )
println( "─".repeat( 40 ) )
println()

// Simulated weather data (in real app, you'd call a weather API)
weatherDatabase = {
    "Boston": { temp: 72, condition: "Sunny" },
    "New York": { temp: 68, condition: "Cloudy" },
    "Miami": { temp: 85, condition: "Hot and humid" },
    "San Francisco": { temp: 62, condition: "Foggy" },
    "Denver": { temp: 65, condition: "Clear" }
}

weatherTool = aiTool(
    "get_weather",
    "Get current weather for a city. Use when asked about weather conditions.",
    ( city ) => {
       
        if( weatherDatabase.keyExists( city ) ) {
            data = weatherDatabase[ city ]
            return "Weather in #city#: #data.temp#°F, #data.condition#"
        }
        
        return "Weather data not available for #city#. Try: #weatherDatabase.keyList()#"
    }
).describeCity( "The city name to get weather for (e.g., 'Boston', 'Miami')" )

// Test questions
questions = [
    "What's the weather like in Boston today?",
    "Is it warm in Miami?",
    "How's the weather in San Francisco?"
]

for( question in questions ) {
    println( "Q: " & question )
    answer = aiChat( question, { tools: [ weatherTool ] } )
    println( "A: " & answer )
    println()
}
```

---

## 🔧 Part 3: Tool Arguments (10 mins)

### Describing Arguments

Use `describeArg()` or dynamic methods to explain what each argument is for:

```java
// Method 1: describeArg()
tool = aiTool( "search", "Search products", fn )
    .describeArg( "query", "The search terms" )
    .describeArg( "category", "Product category to filter by" )
    .describeArg( "maxResults", "Maximum number of results (default 10)" )

// Method 2: Dynamic methods (cleaner!)
tool = aiTool( "search", "Search products", fn )
    .describeQuery( "The search terms" )
    .describeCategory( "Product category to filter by" )
    .describeMaxResults( "Maximum number of results" )
```

### Why Descriptions Matter

Good descriptions help AI use tools correctly:

```
❌ BAD: .describeCity( "city" )
   AI might pass "NYC", "new york", "New York City"

✅ GOOD: .describeCity( "City name, e.g. 'Boston' or 'New York'" )
   AI understands the expected format!
```

---

## 🔗 Part 4: Multiple Tools (15 mins)

AI can decide which tool to use (or use multiple):

### Example: Smart Assistant

```java
// smart-assistant.bxs
// AI with multiple tools - it chooses the right one!
// Run with: boxlang smart-assistant.bxs

println( "🤖 Smart Assistant Demo" )
println( "═".repeat( 40 ) )
println( "This assistant has 3 tools: weather, calculator, and time" )
println()

// Tool 1: Weather
weatherData = { "Boston": 72, "Miami": 85, "Denver": 65, "Seattle": 58 }
weatherTool = aiTool(
    "get_weather",
    "Get current weather for a city",
    ( city ) => {
        temp = weatherData[ city ] ?: 70
        return "#temp#°F in #city#"
    }
).describeCity( "City name (Boston, Miami, Denver, or Seattle)" )

// Tool 2: Calculator
calcTool = aiTool(
    "calculate",
    "Perform math calculations",
    ( expression ) => {
        try {
            return evaluate( expression )
        } catch( any e ) {
            return "Error: " & e.message
        }
    }
).describeExpression( "Math expression like '2+2' or '100*0.2'" )

// Tool 3: Time
timeTool = aiTool(
    "get_time",
    "Get current date and time",
    ( args ) => {
        return now().format( "EEEE, MMMM d, yyyy 'at' h:mm a" )
    }
)

// All tools available to AI
tools = [ weatherTool, calcTool, timeTool ]

// Different questions that use different tools
questions = [
    "What's the weather in Miami?",
    "What's 25% of 400?",
    "What day is it today?",
    "Is Boston warmer than Denver?",
    "Calculate 15 * 8 + 12"
]

for( question in questions ) {
    println( "Q: " & question )
    answer = aiChat( question, { tools: tools } )
    println( "A: " & answer )
    println( "─".repeat( 40 ) )
}
```

### Tool Selection

AI automatically chooses the right tool based on the question:

```
┌─────────────────────────────────────────────────────────────────┐
│                    TOOL SELECTION                               │
└─────────────────────────────────────────────────────────────────┘

  "Weather in Boston?"    ──▶  get_weather( city="Boston" )

  "Calculate 15% tip"     ──▶  calculate( expression="0.15*50" )

  "What time is it?"      ──▶  get_time()

  "Tell me a joke"        ──▶  (no tool needed, AI answers directly)
```

---

## 🧪 Part 5: Lab - Weather Bot (10 mins)

### The Goal

Build a weather bot that:

1. Has a weather lookup tool
2. Has a temperature converter tool
3. Can answer combined questions

### Solution

```java
// weather-bot.bxs
// Lab: Interactive weather bot with multiple tools
// Run with: boxlang weather-bot.bxs

println( "🌤️ Weather Bot" )
println( "═".repeat( 40 ) )
println()

// Weather data (simulated)
weatherData = {
    "Boston": { temp: 72, condition: "Sunny", humidity: 45 },
    "New York": { temp: 68, condition: "Cloudy", humidity: 60 },
    "Miami": { temp: 85, condition: "Hot", humidity: 80 },
    "San Francisco": { temp: 62, condition: "Foggy", humidity: 70 },
    "Denver": { temp: 58, condition: "Clear", humidity: 30 }
}

// Tool 1: Get weather
weatherTool = aiTool(
    "get_weather",
    "Get current weather for a city. Returns temperature in Fahrenheit.",
    ( city ) => {
       
        if( weatherData.keyExists( city ) ) {
            data = weatherData[ city ]
            return "Weather in #city#: #data.temp#°F, #data.condition#, Humidity: #data.humidity#%"
        }
        
        return "No weather data for #city#. Available cities: #weatherData.keyList()#"
    }
).describeCity( "City name exactly as: Boston, New York, Miami, San Francisco, or Denver" )

// Tool 2: Convert temperature
convertTool = aiTool(
    "convert_temperature",
    "Convert temperature between Fahrenheit and Celsius",
    ( args ) => {
        temp = args.temperature
        from = args.fromUnit.uCase()
        
        if( from == "F" || from == "FAHRENHEIT" ) {
            celsius = ( temp - 32 ) * 5/9
            return "#temp#°F = #numberFormat( celsius, '0.0' )#°C"
        } else {
            fahrenheit = ( temp * 9/5 ) + 32
            return "#temp#°C = #numberFormat( fahrenheit, '0.0' )#°F"
        }
    }
)
.describeTemperature( "The temperature value as a number" )
.describeFromUnit( "The unit to convert FROM: 'F' for Fahrenheit or 'C' for Celsius" )

// Tool 3: Compare cities
compareTool = aiTool(
    "compare_cities",
    "Compare weather between two cities",
    ( args ) => {
        
        var cityArr =args.listtoArray("vs",false,true);
        city1 = cityArr[1].trim()
        city2 = cityArr[2].trim()
        
        if( !weatherData.keyExists( city1 ) || !weatherData.keyExists( city2 ) ) {
            return "Can't compare - need valid cities from: #weatherData.keyList()#"
        }
        
        data1 = weatherData[ city1 ]
        data2 = weatherData[ city2 ]
        diff = data1.temp - data2.temp
        
        if( diff > 0 ) {
            return "#city1# is #abs(diff)#°F warmer than #city2#"
        } else if( diff < 0 ) {
            return "#city2# is #abs(diff)#°F warmer than #city1#"
        } else {
            return "#city1# and #city2# are the same temperature"
        }
    }
)
.describeCity1( "First city to compare" )
.describeCity2( "Second city to compare" )

// All tools
tools = [ weatherTool, convertTool, compareTool ]

// Chat loop
println( "Ask me about weather! Type 'quit' to exit." )
println( "Available cities: #weatherData.keyList()#" )
println( "─".repeat( 40 ) )
println()

running = true
while( running ) {
        question = cliRead( "You: " )
    
    if( question.trim() == "quit" ) {
        running = false
        println( "☀️ Goodbye!" )
    } else {
        try {
            answer = aiChat( question, { tools: tools } )
            println( "Bot: " & answer )
            println()
        } catch( any e ) {
            println( "❌ Error: " & e.message )
            println()
        }
    }
}
```

### Sample Interaction

```
🌤️ Weather Bot
════════════════════════════════════════

Ask me about weather! Type 'quit' to exit.
────────────────────────────────────────

You: What's the weather in Miami?
Bot: The weather in Miami is 85°F and hot with 80% humidity.

You: Convert that to Celsius
Bot: 85°F is 29.4°C.

You: Which is warmer, Boston or Denver?
Bot: Boston is 14°F warmer than Denver.

You: quit
☀️ Goodbye!
```

---

## 🗂️ Part 5: Global Tool Registry (10 mins)

When your app has many tools shared across different AI calls, passing instances everywhere becomes repetitive. The **Tool Registry** lets you register tools once at startup and reference them by name.

### Register Once, Use Anywhere

```java
// At app startup (once)
registry = aiToolRegistry()

registry.register(
    name       : "calculator",
    description: "Performs math: +, -, *, /",
    module     : "myapp",
    callback   : ( args ) => evaluate( args.expression )
)

registry.register(
    name       : "get_weather",
    description: "Returns weather for a city",
    module     : "myapp",
    callback   : ( args ) => fetchWeather( args.city )
)
```

### Reference by Name in aiChat()

```java
// Later, anywhere in your code — reference by "name@module" string
answer = aiChat(
    "What is 15% of 200 and what's the weather in Miami?",
    { tools: [ "calculator@myapp", "get_weather@myapp" ] }
)
```

You can also mix string references with inline tool instances:

```java
inlineTool = aiTool( "get_date", "Today's date", ( args ) => now() )

answer = aiChat(
    "What time is it and what's 2+2?",
    { tools: [ "calculator@myapp", inlineTool ] }
)
```

### Inspect the Registry

```java
// List all registered tools
registry.getAll().each( ( key, tool ) => {
    println( "#key# → #tool.getDescription()#" )
})

// Check if a tool exists
if( registry.has( "calculator@myapp" ) ) {
    println( "Calculator is registered!" )
}
```

### Built-In Tools

`bx-ai` ships `now@bx-ai` — a date/time tool available without registration:

```java
answer = aiChat( "What is today's date?", { tools: [ "now@bx-ai" ] } )
```

### Summary: Inline vs Registry

| Approach | Best For |
|---|---|
| Inline `aiTool()` | One-off tools in a single script |
| Registry `aiToolRegistry()` | Shared tools used across the application |

---

## ✅ Knowledge Check

1. **What do AI tools allow?**
   - [ ] Change AI's personality
   - [x] AI can call your functions
   - [ ] Speed up responses
   - [ ] Reduce costs

2. **How do you create a tool?**
   - [x] aiTool( name, description, function )
   - [ ] createTool()
   - [ ] addFunction()
   - [ ] registerTool()

3. **Why describe tool arguments?**
   - [ ] Required by BoxLang
   - [ ] Makes code faster
   - [x] Helps AI understand how to use the tool
   - [ ] For documentation only

4. **Can AI use multiple tools?**
   - [x] Yes, it chooses the right one
   - [ ] No, only one at a time
   - [ ] Only with special config
   - [ ] Only in pipelines

---

## 📝 Summary

You learned:

| Concept | Description |
|---------|-------------|
| **Tool** | Function AI can call |
| **aiTool()** | Creates a tool object |
| **describe*()** | Explains arguments to AI |
| **tools: []** | Pass tools to aiChat |

### Key Code Patterns

```java
// Create a tool
tool = aiTool( "name", "description", ( args ) => {
    return result
}).describeArg( "what it is" )

// Use tools
answer = aiChat( "question", { tools: [ tool1, tool2 ] } )
```

---

## ⏭️ Next Lesson

Now you can give AI real capabilities! Let's put it all together and build autonomous agents.

👉 **[Lesson 6: Building Agents](../lesson-06-agents/README.md)**

---

## 📁 Lesson Files

```
lesson-05-tools/
├── README.md (this file)
├── examples/
│   ├── calculator-tool.bxs
│   ├── weather-tool.bxs
│   ├── smart-assistant.bxs
│   └── tool-registry.bxs
└── labs/
    └── weather-bot.bxs
```
