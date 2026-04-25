# Lesson 1: Getting Started

[Home](../README.md)

**⏱️ Duration: 45 minutes**

Welcome to your first step into AI development with BoxLang! In this lesson, you'll set up your environment and make your first AI call.

## 🎯 What You'll Learn

- Install and configure the bx-ai module
- Understand API keys and providers
- Make your first AI call
- Understand tokens (how AI "sees" text)

---

## 📚 Part 1: Understanding AI Basics (10 mins)

Before writing code, let's understand what we're working with.

### What is a Large Language Model (LLM)?

An LLM is an AI system that:

- **Reads** and understands text
- **Generates** human-like responses
- **Helps** with coding, writing, analysis, and more

```
┌─────────────────────────────────────────────────────────────────┐
│                        HOW AI WORKS                             │
└─────────────────────────────────────────────────────────────────┘

  Your Question          AI Processing           AI Response
  ──────────────        ───────────────         ──────────────
       │                      │                       │
       ▼                      ▼                       ▼
  ┌─────────┐           ┌──────────┐            ┌─────────┐
  │"What is │  ──────▶  │ Neural   │  ──────▶  │"BoxLang │
  │BoxLang?"│           │ Network  │            │is a JVM │
  │         │           │(billions │            │language │
  │         │           │of params)│            │that..." │
  └─────────┘           └──────────┘            └─────────┘
```

### What is a Token?

AI doesn't see words like we do. It breaks text into **tokens** - small pieces of text.

```
┌─────────────────────────────────────────────────────────────────┐
│                      TOKEN EXAMPLES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   "Hello"           =  1 token                                  │
│   "Hello world"     =  2 tokens                                 │
│   "BoxLang is cool" =  4 tokens                                 │
│                                                                 │
│   💡 Rule of thumb: ~4 characters = 1 token                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Why tokens matter:**

- ✅ You pay per token (input + output)
- ✅ Models have token limits (context window)
- ✅ More tokens = longer responses

### What is an AI Provider?

A provider is a company that runs AI models in the cloud (or locally):

| Provider | Models | Pricing | Best For |
|----------|--------|---------|----------|
| **OpenAI** | GPT-4, GPT-4o-mini | Paid | Most capable |
| **Claude** | Claude 3.5 Sonnet | Paid | Long context, reasoning |
| **Ollama** | Llama 3.2, Mistral | **Free** | Local/private |

---


---

## 💻 Part 2: Your First AI Call (10 mins)

### The aiChat() Function

The simplest way to talk to AI:

```java
result = aiChat( "Your message here" )
```

That's it! Let's try it.

### Example 1: Hello AI

Create a file called `hello-ai.bxs`:

```java
// hello-ai.bxs
// Your first AI call!

answer = aiChat( "Say hello to someone learning BoxLang AI!" )
println( answer )
```

Run it:

Note: Running scripts from the root of the project, even to lower folders, will ensure your API keys are read with minimal configuration. We will look at ways of mitigating this later in the course.

```bash
boxlang bootcamp\lesson-01-getting-started\examples\hello-ai.bxs
```

**Expected output:** ( or equivalent )

```
Hello! Welcome to your BoxLang AI journey! I'm excited to help you 
learn how to build amazing AI-powered applications. Let's get started! 🚀
```

### Example 2: Ask a Question

```java
// ask-question.bxs
question = "What is BoxLang in one sentence?"
answer = aiChat( question )

println( "Q: " & question )
println( "A: " & answer )
```

### Example 3: Using Ollama (Free/Local)

If you installed Ollama, make sure it is running and try the next example:

```java
// local-ai.bxs
answer = aiChat(
    "What is 2 + 2? Just give me the number.",
    { model: "qwen3:0.6b", provider: "ollama" },
    { provider: "ollama"}
)
println( answer )
```

### Example 4: Error Handling

Always wrap AI calls in try/catch:

```java
// safe-call.bxs
try {
    answer = aiChat( "Tell me a programming joke" )
    println( answer )
} catch( any e ) {
    println( "❌ Error: " & e.message )
    println( "💡 Check your API key!" )
}
```

---

## 🧪 Part 4: Lab - Magic 8-Ball (10 mins)

Let's build your first AI application: a Magic 8-Ball!

### The Goal

Create an AI-powered fortune teller that answers yes/no questions.

### Instructions

1. Create a file `magic-8-ball.bxs`
2. Ask the user for a question
3. Send it to the AI with special instructions
4. Display the mystical answer

### One more concept: Prompts

A prompt is context and background given the AI engine to better frame how it answers. In this lab, we are giving the model a context from which to answer. We are telling the model that it is a Magic 8 ball and it can only answer with a list of predetermined responses which we are providing. The lab goes into prompts in more detail.

### Starter Code

```java
// magic-8-ball.bxs

println( "🎱 Welcome to the AI Magic 8-Ball! 🎱" )
println( "Ask me a yes/no question..." )
println( "" )

// Get user's question
question = cliRead( "Your question: " )

// Create the magic prompt
prompt = "
You are a mystical Magic 8-Ball. 
Answer the following yes/no question with ONE of these classic responses:
- It is certain
- Without a doubt
- Yes definitely
- You may rely on it
- Most likely
- Outlook good
- Signs point to yes
- Reply hazy, try again
- Ask again later
- Cannot predict now
- Don't count on it
- My sources say no
- Outlook not so good
- Very doubtful

Question: #question#

Respond with ONLY the Magic 8-Ball phrase, nothing else.
"

// Get the mystical answer
try {
    answer = aiChat( prompt, { temperature: 1.0 } )
    println( "" )
    println( "🔮 The Magic 8-Ball says..." )
    println( "   " & answer )
} catch( any e ) {
    println( "❌ The spirits are unclear: " & e.message )
}
```

### Run It

```bash
boxlang magic-8-ball.bxs
```

### Sample Output

```
🎱 Welcome to the AI Magic 8-Ball! 🎱
Ask me a yes/no question...

Your question: Will I learn BoxLang AI today?

🔮 The Magic 8-Ball says...
   It is certain
```

### Challenge

Modify the Magic 8-Ball to:
1. Keep asking questions in a loop
2. Type "quit" to exit
3. Count how many questions were asked

---

## ✅ Knowledge Check

Test your understanding:

1. **What does an LLM do?**
   - [ ] Only writes code
   - [x] Understands and generates text
   - [ ] Stores databases
   - [ ] Runs servers

2. **What is a token?**
   - [ ] A password
   - [x] A piece of text the AI processes
   - [ ] A type of variable
   - [ ] An error message

3. **Which provider is free and runs locally?**
   - [ ] OpenAI
   - [ ] Claude
   - [x] Ollama
   - [ ] Gemini

4. **What function makes a simple AI call?**
   - [ ] ai()
   - [x] aiChat()
   - [ ] sendAI()
   - [ ] chatBot()

---

## 📝 Summary

You learned:

| Concept | What It Means |
|---------|---------------|
| **LLM** | AI that understands and generates text |
| **Token** | A piece of text (~4 characters) |
| **Provider** | Company running AI models |
| **aiChat()** | Function to send messages to AI |

### Key Code

```java
// Basic AI call
answer = aiChat( "Your message" )

// With options
answer = aiChat( 
    "Message",
    { temperature: 0.7 },         // Parameters
    { provider: "openai" }        // Options
)
```

---

## ⏭️ Next Lesson

You've made your first AI call! Now let's learn to have real conversations.

👉 **[Lesson 2: Conversations & Messages](../lesson-02-conversations/)**

---

## 📁 Lesson Files

```
lesson-01-getting-started/
├── README.md (this file)
├── examples/
│   ├── hello-ai.bxs
│   ├── ask-question.bxs
│   ├── local-ai.bxs
│   └── safe-call.bxs
└── labs/
    └── magic-8-ball.bxs
```
