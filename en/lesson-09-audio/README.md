# Lesson 9: Audio AI — Speech, Transcription & Translation

[Home](../README.md)

**⏱️ Duration: 60 minutes**

In this lesson you'll add **voice capabilities** to your BoxLang AI applications. The bx-ai module ships three audio BIFs: `aiSpeak()` for text-to-speech, `aiTranscribe()` for speech-to-text, and `aiTranslate()` for audio-to-English translation.

## 🎯 What You'll Learn

- Convert text to natural speech with `aiSpeak()`
- Transcribe audio files to text with `aiTranscribe()`
- Translate foreign-language audio directly to English with `aiTranslate()`
- Work with the `AiSpeechResponse` object (raw bytes, content type, save to file)
- Chain `aiSpeak()` → `aiTranscribe()` in a round-trip pipeline
- Configure audio defaults in `config/boxlang.json`

---

## 📚 Part 1: Supported Providers (5 mins)

Not every provider supports all three operations:

| Provider | TTS (`aiSpeak`) | STT (`aiTranscribe`) | Translation (`aiTranslate`) |
|---|---|---|---|
| **OpenAI** | ✅ | ✅ | ✅ |
| **ElevenLabs** | ✅ | ✅ | ❌ |
| **Groq** | ❌ | ✅ (Whisper) | ❌ |

**API keys needed** — add to `.env`:

```bash
OPENAI_API_KEY=your-key
ELEVENLABS_API_KEY=your-key   # optional — for ElevenLabs TTS/STT
GROQ_API_KEY=your-key         # optional — for fast Whisper transcription
```

### Module-Level Audio Defaults

Configure defaults in `config/boxlang.json` so you don't have to repeat them in every call:

```javascript
{
    "modules": {
        "bxai": {
            "settings": {
                "audio": {
                    "defaultVoice"              : "alloy",
                    "defaultOutputFormat"       : "mp3",
                    "defaultSpeechModel"        : "tts-1",
                    "defaultTranscriptionModel" : "whisper-1"
                }
            }
        }
    }
}
```

---

## 🔊 Part 2: Text-to-Speech with `aiSpeak()` (20 mins)

### Signature

```javascript
aiSpeak( text, params, options )
```

| Argument | Type | Description |
|---|---|---|
| `text` | string | The text to synthesise |
| `params` | struct | Provider-level params: `model`, `voice`, `instructions` |
| `options` | struct | Module options: `outputFile`, `provider` |

**Returns:** `AiSpeechResponse` — holds audio bytes, content type, and a `saveToFile()` helper.

### Example 1: Save directly to a file

```javascript
aiSpeak(
    text   : "Hello! Welcome to BoxLang AI.",
    options: { outputFile: expandPath( "./data/hello.mp3" ) }
)
```

### Example 2: Choose voice and model

```javascript
// OpenAI voices: alloy, echo, fable, onyx, nova, shimmer
// Models: tts-1, tts-1-hd, gpt-4o-mini-tts
aiSpeak(
    text   : "BoxLang makes AI integration elegant and fun.",
    params : { model: "tts-1-hd", voice: "nova" },
    options: { outputFile: expandPath( "./data/nova-hd.mp3" ), provider: "openai" }
)
```

### Example 3: Work with `AiSpeechResponse`

```javascript
response = aiSpeak(
    text   : "The quick brown fox jumps over the lazy dog.",
    params : { voice: "alloy" },
    options: { provider: "openai" }
)

println( response.getMimeType() )          // "audio/mpeg"
println( response.getAudioData().length )     // bytes

response.saveToFile( expandPath( "./data/alloy.mp3" ) )
```

### Example 4: Voice style instructions

> ⚠️ Style instructions are only supported by `gpt-4o-mini-tts`.

```javascript
aiSpeak(
    text    : "Ahoy! Welcome to BoxLang AI, the finest treasure on the seven seas!",
    params  : { model: "gpt-4o-mini-tts", voice: "ash" },
    options : {
        outputFile  : expandPath( "./data/pirate.mp3" ),
        instructions: "Speak like a dramatic pirate. Use 'Arrr!' and seafaring vocabulary."
    }
)
```

📂 **Example file:** `examples/tts-basic.bxs`

---

## 🎙️ Part 3: Speech-to-Text with `aiTranscribe()` (20 mins)

### Signature

```javascript
aiTranscribe( audio, params, options )
```

| Argument | Type | Description |
|---|---|---|
| `audio` | string \| byte[] | File path, URL, or raw audio bytes |
| `params` | struct | Provider params: `language`, `model` |
| `options` | struct | Module options: `provider` |

**Returns:** string — the transcribed text.

### Example 1: Simple transcription

```javascript
text = aiTranscribe( expandPath( "./assets/sample.mp3" ) )
println( text )
```

### Example 2: Language hint for better accuracy

```javascript
// Providing 'language' skips auto-detection and improves speed + accuracy
spanishText = aiTranscribe(
    audio  : expandPath( "./assets/sample-es.mp3" ),
    params : { language: "es" }
)
```

### Example 3: Groq Whisper (faster)

```javascript
groqText = aiTranscribe(
    audio  : expandPath( "./assets/sample.mp3" ),
    params : { model: "whisper-large-v3" },
    options: { provider: "groq" }
)
```

### Example 4: TTS → STT round-trip

```javascript
original = "BoxLang is a modern dynamic JVM language."

// Synthesise speech in memory — no file needed
speechResponse = aiSpeak( original )

// Pass raw bytes directly to aiTranscribe()
roundTrip = aiTranscribe( speechResponse.getAudioData() )

println( "Original  : " & original  )
println( "Round-trip: " & roundTrip )
```

📂 **Example file:** `examples/transcription.bxs`

---

## 🌐 Part 4: Audio Translation with `aiTranslate()` (10 mins)

`aiTranslate()` translates audio **in any language** directly to **English text**.

> 💡 Key difference:
> - `aiTranscribe()` → transcribes audio to text **in its original language**
> - `aiTranslate()` → translates audio to **English**, regardless of source language

### Signature

```javascript
aiTranslate( audio, params, options )
```

### Example: Compare transcribe vs translate

```javascript
spanishFile = expandPath( "./assets/sample-es.mp3" )

// Transcribe preserves original language
original = aiTranscribe( audio: spanishFile, params: { language: "es" } )
println( "Spanish text : " & original )

// Translate always outputs English
english = aiTranslate( spanishFile )
println( "English text : " & english )
```

### Example: Explicit provider/model

```javascript
translation = aiTranslate(
    audio  : expandPath( "./assets/french-audio.mp3" ),
    params : { model: "whisper-1" },
    options: { provider: "openai" }
)
```

📂 **Example file:** `examples/translation-audio.bxs`

---

## 🔬 Knowledge Check

1. Which BIF converts text to speech?
   - [ ] `aiTranscribe()`
   - [x] `aiSpeak()`
   - [ ] `aiTranslate()`
   - [ ] `aiVoice()`

2. What does `AiSpeechResponse.getAudioData()` return?
   - [ ] A file path string
   - [ ] A base64 string
   - [x] Raw audio bytes (`byte[]`)
   - [ ] A URL to the audio file

3. What language does `aiTranslate()` always output?
   - [ ] The source language
   - [ ] Spanish
   - [x] English
   - [ ] The system default language

4. Which provider supports **only** speech-to-text (no TTS)?
   - [x] Groq
   - [ ] OpenAI
   - [ ] ElevenLabs
   - [ ] All providers support TTS

---

## 📝 Summary

| BIF | Direction | Output | Key params |
|---|---|---|---|
| `aiSpeak(text, params, options)` | Text → Audio | `AiSpeechResponse` | `voice`, `model`, `instructions` |
| `aiTranscribe(audio, params, options)` | Audio → Text | string (original lang) | `language`, `model` |
| `aiTranslate(audio, params, options)` | Audio → Text | string (English only) | `model` |

### Key Code Patterns

```javascript
// Save TTS directly to file
aiSpeak( text: "Hello", options: { outputFile: "hello.mp3" } )

// Capture bytes for further processing
bytes = aiSpeak( "Hello" ).getAudioData()

// Transcribe with provider choice
text = aiTranscribe( "audio.mp3", {}, { provider: "groq" } )

// Round-trip: TTS → STT
roundTrip = aiTranscribe( aiSpeak( "Hello" ).getAudioData() )
```

---

## 📁 Lesson Files

```
lesson-09-audio/
├── README.md (this file)
├── examples/
│   ├── tts-basic.bxs          # aiSpeak() — save to file, AiSpeechResponse, voice + style
│   ├── transcription.bxs      # aiTranscribe() — local file, language hint, Groq, round-trip
│   └── translation-audio.bxs  # aiTranslate() vs aiTranscribe() comparison
└── labs/
    └── audio-pipeline.bxs     # Full TTS → STT pipeline lab
```

---

## ⏭️ What's Next?

- **Lesson 7**: Document Loaders & RAG — load PDFs, web pages, and databases into AI context
- **Lesson 8**: MCP Servers & Clients — build and consume Model Context Protocol services
- Explore `examples/` in the bx-ai-intro workspace for more advanced audio patterns
