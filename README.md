# 🚀 BoxLang AI Bootcamp

**Get Started with AI Development in One Day**

## 🌍 Select Your Language / Selecciona tu Idioma

| Language | Idioma | Link |
|----------|--------|------|
| 🇺🇸 English | Inglés | **[Start Bootcamp →](en/)** |
| 🇪🇸 Español | Spanish | **[Iniciar Bootcamp →](es/)** |

---

## ⚡ Quick Setup

Run the interactive setup wizard to get your bootcamp environment ready in seconds:

```bash
boxlang Setup.bx
```

The wizard will:

1. Copy `.env.example` → `.env` so you can add your API keys
2. Install the `bx-ai` module locally via `install-bx-module`
3. Ask whether you prefer **English** or **Español**
4. Create a `bootcamp/` folder with all lessons for your chosen language

> The `bootcamp/` folder is git-ignored — it's your personal workspace. Hack away!

---

## 🛠️ Setup Commands

| Command | Description |
|---------|-------------|
| `boxlang Setup.bx` | Run the interactive setup wizard (default) |
| `boxlang Setup.bx --help` | Show all available commands and options |
| `boxlang Setup.bx --reset` | Reset the `bootcamp/` folder back to its original state |

### `boxlang Setup.bx` (default)

Runs the full setup wizard:

- Creates `.env` from `.env.example` if it doesn't already exist
- Installs `bx-ai` locally (`install-bx-module bx-ai --local`)
- Prompts you to choose English 🇺🇸 or Spanish 🇪🇸
- If a `bootcamp/` folder already exists, asks if you want to start fresh before overwriting

### `boxlang Setup.bx --reset`

Prompts for confirmation, then re-copies the original lesson content into `bootcamp/` for the language of your choice.
Useful when you want to start a lesson over from a clean slate.

### `boxlang Setup.bx --help`

Prints the above usage summary directly in your terminal.

---

## 📝 Adding Your API Keys

After setup, open the generated `.env` file and fill in at least one AI provider key:

```dotenv
OPENAI_API_KEY=sk-...
CLAUDE_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-...
# etc.
```

---

## 📋 About This Bootcamp

This intensive, hands-on course takes you from zero AI experience to building intelligent applications with BoxLang in just 8-9 hours (8 lessons).

### What You'll Learn / Lo que Aprenderás

- ✅ Understand how AI and Large Language Models work
- ✅ Make your first AI API call in minutes
- ✅ Build multi-turn conversations with context
- ✅ Switch between AI providers (OpenAI, Claude, Ollama)
- ✅ Extract structured data from AI responses
- ✅ Create tools that AI can use
- ✅ Build an autonomous AI agent
- ✅ Load and process documents for RAG systems
- ✅ Create MCP servers and clients for remote tools
- ✅ Add voice — text-to-speech, transcription and audio translation

---

## 📜 License

This bootcamp is released under the Apache 2.0 license, same as the bx-ai module.
