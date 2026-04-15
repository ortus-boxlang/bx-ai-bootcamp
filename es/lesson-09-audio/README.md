# Lección 9: Audio con IA — Voz, Transcripción y Traducción

**⏱️ Duración: 60 minutos**

En esta lección agregarás **capacidades de voz** a tus aplicaciones BoxLang AI. El módulo bx-ai incluye tres BIFs de audio: `aiSpeak()` para texto a voz, `aiTranscribe()` para voz a texto, y `aiTranslate()` para traducir audio al inglés directamente.

## 🎯 Lo que aprenderás

- Convertir texto en voz natural con `aiSpeak()`
- Transcribir archivos de audio a texto con `aiTranscribe()`
- Traducir audio en otro idioma directamente al inglés con `aiTranslate()`
- Trabajar con el objeto `AiSpeechResponse` (bytes, tipo de contenido, guardar en archivo)
- Encadenar `aiSpeak()` → `aiTranscribe()` en un pipeline de ida y vuelta
- Configurar valores predeterminados de audio en `config/boxlang.json`

---

## 📚 Parte 1: Proveedores compatibles (5 min)

No todos los proveedores admiten las tres operaciones:

| Proveedor | TTS (`aiSpeak`) | STT (`aiTranscribe`) | Traducción (`aiTranslate`) |
|---|---|---|---|
| **OpenAI** | ✅ | ✅ | ✅ |
| **ElevenLabs** | ✅ | ✅ | ❌ |
| **Groq** | ❌ | ✅ (Whisper) | ❌ |

**Claves de API necesarias** — agrega en `.env`:

```bash
OPENAI_API_KEY=tu-clave
ELEVENLABS_API_KEY=tu-clave   # opcional — para TTS/STT de ElevenLabs
GROQ_API_KEY=tu-clave         # opcional — para transcripción rápida con Whisper
```

### Valores predeterminados de audio en el módulo

Configura los valores por defecto en `config/boxlang.json` para no repetirlos en cada llamada:

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

## 🔊 Parte 2: Texto a voz con `aiSpeak()` (20 min)

### Firma del BIF

```javascript
aiSpeak( text, params, options )
```

| Argumento | Tipo | Descripción |
|---|---|---|
| `text` | string | El texto a sintetizar |
| `params` | struct | Parámetros del proveedor: `model`, `voice`, `instructions` |
| `options` | struct | Opciones del módulo: `outputFile`, `provider` |

**Retorna:** `AiSpeechResponse` — tiene los bytes de audio, el tipo de contenido y el método `saveToFile()`.

### Ejemplo 1: Guardar directamente en un archivo

```javascript
aiSpeak(
    text   : "¡Hola! Bienvenido a BoxLang AI.",
    options: { outputFile: expandPath( "./data/hola.mp3" ) }
)
```

### Ejemplo 2: Elegir voz y modelo

```javascript
// Voces de OpenAI: alloy, echo, fable, onyx, nova, shimmer
// Modelos: tts-1, tts-1-hd, gpt-4o-mini-tts
aiSpeak(
    text   : "BoxLang hace la integración de IA elegante y divertida.",
    params : { model: "tts-1-hd", voice: "nova" },
    options: { outputFile: expandPath( "./data/nova-hd.mp3" ), provider: "openai" }
)
```

### Ejemplo 3: Trabajar con `AiSpeechResponse`

```javascript
response = aiSpeak(
    text   : "El rápido zorro marrón salta sobre el perro perezoso.",
    params : { voice: "alloy" },
    options: { provider: "openai" }
)

println( response.getContentType() )          // "audio/mpeg"
println( response.getAudioData().length )     // bytes

response.saveToFile( expandPath( "./data/alloy.mp3" ) )
```

### Ejemplo 4: Instrucciones de estilo de voz

> ⚠️ Las instrucciones de estilo solo son compatibles con `gpt-4o-mini-tts`.

```javascript
aiSpeak(
    text    : "¡Ahoy! ¡Bienvenido a BoxLang AI, el mejor tesoro de los siete mares!",
    params  : { model: "gpt-4o-mini-tts", voice: "ash" },
    options : {
        outputFile  : expandPath( "./data/pirata.mp3" ),
        instructions: "Habla como un dramático pirata. Usa 'Arrr!' y vocabulario marinero."
    }
)
```

📂 **Archivo de ejemplo:** `examples/tts-basico.bxs`

---

## 🎙️ Parte 3: Voz a texto con `aiTranscribe()` (20 min)

### Firma del BIF

```javascript
aiTranscribe( audio, params, options )
```

| Argumento | Tipo | Descripción |
|---|---|---|
| `audio` | string \| byte[] | Ruta de archivo, URL o bytes de audio |
| `params` | struct | `language`, `model` |
| `options` | struct | `provider` |

**Retorna:** string — el texto transcrito.

### Ejemplo 1: Transcripción simple

```javascript
texto = aiTranscribe( expandPath( "./assets/sample.mp3" ) )
println( texto )
```

### Ejemplo 2: Indicar idioma para mayor precisión

```javascript
textoEspanol = aiTranscribe(
    audio  : expandPath( "./assets/sample-es.mp3" ),
    params : { language: "es" }
)
```

### Ejemplo 3: Groq Whisper (más rápido)

```javascript
textoGroq = aiTranscribe(
    audio  : expandPath( "./assets/sample.mp3" ),
    params : { model: "whisper-large-v3" },
    options: { provider: "groq" }
)
```

### Ejemplo 4: Ida y vuelta TTS → STT

```javascript
original = "BoxLang es un lenguaje JVM moderno y dinámico."

respuestaVoz = aiSpeak( original )
idaVuelta    = aiTranscribe( respuestaVoz.getAudioData() )

println( "Original  : " & original   )
println( "Ida-vuelta: " & idaVuelta  )
```

📂 **Archivo de ejemplo:** `examples/transcripcion.bxs`

---

## 🌐 Parte 4: Traducción de audio con `aiTranslate()` (10 min)

`aiTranslate()` traduce audio **en cualquier idioma** directamente a **texto en inglés**.

> 💡 Diferencia clave:
> - `aiTranscribe()` → transcribe el audio al **idioma original**
> - `aiTranslate()` → traduce el audio **siempre al inglés**

### Firma del BIF

```javascript
aiTranslate( audio, params, options )
```

### Ejemplo: Comparar transcripción vs traducción

```javascript
archivoEspanol = expandPath( "./assets/sample-es.mp3" )

// Transcripción — conserva el idioma original
original = aiTranscribe( audio: archivoEspanol, params: { language: "es" } )
println( "Texto en español: " & original )

// Traducción — siempre genera inglés
ingles = aiTranslate( archivoEspanol )
println( "Texto en inglés : " & ingles )
```

📂 **Archivo de ejemplo:** `examples/traduccion-audio.bxs`

---

## 🔬 Comprobación de conocimientos

1. ¿Qué BIF convierte texto a voz?
   - [ ] `aiTranscribe()`
   - [x] `aiSpeak()`
   - [ ] `aiTranslate()`
   - [ ] `aiVoice()`

2. ¿Qué retorna `AiSpeechResponse.getAudioData()`?
   - [ ] Una ruta de archivo string
   - [ ] Un string en base64
   - [x] Bytes de audio crudos (`byte[]`)
   - [ ] Una URL al archivo de audio

3. ¿En qué idioma genera siempre la salida `aiTranslate()`?
   - [ ] El idioma de origen
   - [ ] Español
   - [x] Inglés
   - [ ] El idioma predeterminado del sistema

4. ¿Qué proveedor solo admite voz a texto (sin TTS)?
   - [x] Groq
   - [ ] OpenAI
   - [ ] ElevenLabs
   - [ ] Todos admiten TTS

---

## 📝 Resumen

| BIF | Dirección | Salida | Parámetros clave |
|---|---|---|---|
| `aiSpeak(text, params, options)` | Texto → Audio | `AiSpeechResponse` | `voice`, `model`, `instructions` |
| `aiTranscribe(audio, params, options)` | Audio → Texto | string (idioma original) | `language`, `model` |
| `aiTranslate(audio, params, options)` | Audio → Texto | string (solo inglés) | `model` |

### Patrones de código clave

```javascript
// Guardar TTS directo en archivo
aiSpeak( text: "Hola", options: { outputFile: "hola.mp3" } )

// Capturar bytes para procesamiento posterior
bytes = aiSpeak( "Hola" ).getAudioData()

// Transcribir con proveedor específico
texto = aiTranscribe( "audio.mp3", {}, { provider: "groq" } )

// Ida y vuelta: TTS → STT
idaVuelta = aiTranscribe( aiSpeak( "Hola" ).getAudioData() )
```

---

## 📁 Archivos de la lección

```
lesson-09-audio/
├── README.md (este archivo)
├── examples/
│   ├── tts-basico.bxs         # aiSpeak() — guardar en archivo, AiSpeechResponse, voz + estilo
│   ├── transcripcion.bxs      # aiTranscribe() — archivo local, idioma, Groq, ida y vuelta
│   └── traduccion-audio.bxs   # aiTranslate() vs aiTranscribe() comparación
└── labs/
    └── pipeline-audio.bxs     # Lab: pipeline completo TTS → STT
```

---

## ⏭️ ¿Qué sigue?

- **Lección 7**: Cargadores de documentos y RAG — carga PDFs, páginas web y bases de datos en el contexto de IA
- **Lección 8**: Servidores y clientes MCP — crea y consume servicios con el Protocolo de Contexto de Modelo
- Explora `examples/` en el workspace bx-ai-intro para patrones de audio más avanzados
