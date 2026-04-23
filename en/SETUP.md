# 🛠️ Setting up for BoxLang AI (15 mins)

[Back To Start](./README.md)

## Background

Because of BoxLang's versatility and multi runtime ability, this bootcamp can be run via the command line / terminal or as a web based site. All the code can be used nearly interchangably but the setup is slightly different.

## As a Web App

This is easiest when using CommandBox. [Download CommandBox](https://www.ortussolutions.com/products/commandbox)

1. Clone this repo into an empty folder
2. From CommandBox in that folder, type

```bash
    server start
```

This will start a server locally with BoxLang as the engine and using OpenJDK21.

Note: The code examples lean heavily toward using the terminal. The underlying code will be transferrable between the two contexts but there are differences in how anything visual appears on the screen such as using println() for the terminal vs writeoutput() for a browser etc.

## Terminal ( Recommended )

### Step 1: Verify BoxLang

Open your terminal and run:

```bash
boxlang --version
```

You should see something like `BoxLang 1.x.x`. If not, [download BoxLang](https://boxlang.io).

### Step 2: Install bx-ai Module

#### Method 1: Using Boxlang Version Manager [docs](https://boxlang.ortusbooks.com/getting-started/installation/boxlang-version-manager-bvm)

```bash
install-bx-module bx-ai
```

#### Method 2: Using CommandBox

```bash
cd ${BOXLANG_HOME}
box install bx-ai
```

### Step 3: Get an API Key

**Option A: OpenAI (Recommended for beginners)**

1. Go to https://platform.openai.com/api-keys
2. Sign up or log in
3. Click "Create new secret key"
4. Copy the key (starts with `sk-`)
5. Add credits to your account ($5 is plenty to start)

**Option B: Ollama (Free, runs locally)**

1. Download from https://ollama.ai
2. Install and run Ollama
3. Pull a model:
   ```bash
   ollama pull llama3.2
   ```
4. No API key needed!

### Step 4: Set Your API Key

#### For Web Apps

Create a `.env` file in your project:

```bash
# For OpenAI
OPENAI_API_KEY=sk-your-key-here

# For Claude
CLAUDE_API_KEY=sk-ant-your-key-here
```

#### For Terminal / CommandLine

FILL IN HERER?????



> ⚠️ **Never commit API keys to git!** Add `.env` to your `.gitignore`.

---
