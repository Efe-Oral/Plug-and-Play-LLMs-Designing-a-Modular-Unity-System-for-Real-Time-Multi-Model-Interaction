# Plug-and-Play LLMs: Modular Unity system for real-time multi-model interaction

Unity **2022.3.51f1** project that wires a single UI flow to **several local LLMs** (via [Ollama](https://github.com/ollama/ollama)), **speech input** (Microsoft Azure Speech), and **speech output** (ElevenLabs). The main idea is to swap models and services without rewriting the whole scene: prompting, streaming replies, logging, and TTS/STT are split across scripts under [`Assets/Scripts/`](Assets/Scripts/). The repo also ships a written report as [`Scientific Paper About the Project and Contributions.pdf`](Scientific%20Paper%20About%20the%20Project%20and%20Contributions.pdf) and notes in [`Babamın suggestionları.docx`](Babam%C4%B1n%20suggestionlar%C4%B1.docx).

Upstream repository: [Plug-and-Play-LLMs… on GitHub](https://github.com/Efe-Oral/Plug-and-Play-LLMs-Designing-a-Modular-Unity-System-for-Real-Time-Multi-Model-Interaction).

**Security:** Several scripts still contain embedded API keys. Before you fork or publish, treat those as placeholders and move secrets to environment variables, a secrets manager, or Unity inspector fields that are not committed.

## Techniques used in the code

- **HTTP POST with a JSON body** — Requests use JSON payloads and JSON response handling, aligned with how the web documents [POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST) and [`Content-Type: application/json`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type).
- **UTF-8 byte encoding for the body** — Text is turned into bytes the same way the web describes [UTF-8](https://developer.mozilla.org/en-US/docs/Glossary/UTF-8) for interchange.
- **Streaming-style consumption of a long response** — While the connection is open, new bytes are read from the download buffer, split on newlines, and parsed per chunk. That mirrors the idea of reading a [ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) in small pieces instead of waiting for the full body (implemented with Unity’s `UnityWebRequest` and a coroutine loop in [`OllamaAPIClient.cs`](Assets/Scripts/OllamaAPIClient.cs)).
- **NDJSON-style lines** — Ollama can stream multiple JSON objects separated by newlines; each line is parsed as its own JSON document, matching the mental model of [JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) as a data interchange format.
- **Coroutines for network + UI** — `IEnumerator` / `yield return` keep the main thread responsive while waiting on I/O. See [Unity: Coroutines](https://docs.unity3d.com/Manual/Coroutines.html).
- **Async/await with `Task.Yield()`** — Waits on `UnityWebRequest` completion without blocking the frame loop. See [C# async scenarios](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios).
- **Append-only file logging** — Final prompt/response pairs are written with `StreamWriter` in append mode. See [`StreamWriter`](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamwriter).
- **Scene wiring without a serialized reference** — `FindObjectOfType` locates [`OllamaAPIClient`](Assets/Scripts/OllamaAPIClient.cs) from [`SpeechRecognitionEfe.cs`](Assets/Scripts/SpeechRecognitionEfe.cs) when needed. See [`Object.FindObjectOfType`](https://docs.unity3d.com/ScriptReference/Object.FindObjectOfType.html).

## Libraries and services

| Piece | What it is | Link |
|--------|------------|------|
| **Ollama** | Local LLM server; default URL in code is `http://localhost:11434/api/generate`. | [Ollama](https://github.com/ollama/ollama) · [Generate API](https://github.com/ollama/ollama/blob/main/docs/api.md) |
| **ElevenLabs (UPM)** | REST client package for voices and TTS (RageAgainstThePixel). | [com.rest.elevenlabs](https://github.com/RageAgainstThePixel/com.rest.elevenlabs) |
| **OpenUPM** | Scoped registry used in [`Packages/manifest.json`](Packages/manifest.json). | [OpenUPM](https://openupm.com/) |
| **ElevenLabs REST (hand-rolled)** | [`ElevenLabsAudioSaver.cs`](Assets/Scripts/ElevenLabsAudioSaver.cs) calls the HTTP API with `xi-api-key`. | [ElevenLabs API](https://elevenlabs.io/docs/api-reference/text-to-speech) |
| **Azure Speech** | Native Speech SDK under [`Assets/SpeechSDK/`](Assets/SpeechSDK/); used from [`SpeechRecognitionEfe.cs`](Assets/Scripts/SpeechRecognitionEfe.cs). | [Azure Speech service](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/) |
| **Unity** | `UnityWebRequest`, **uGUI**, **TextMesh Pro**. | [UnityWebRequest](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html) · [TextMesh Pro](https://docs.unity3d.com/Manual/com.unity.textmeshpro.html) |
| **TMP emoji sprite** | EmojiOne assets under TextMesh Pro. | [JoyPixels / EmojiOne](https://github.com/joypixels/emojione) (check license for your use case) |

## Fonts

- **Liberation Sans** — Bundled with TMP; OFL text in [`Assets/TextMesh Pro/Fonts/LiberationSans - OFL.txt`](Assets/TextMesh%20Pro/Fonts/LiberationSans%20-%20OFL.txt). Source project: [liberation-fonts](https://github.com/liberationfonts/liberation-fonts).
- **Valty DEMO** — OTF + SDF under [`Assets/TextMesh Pro/Fonts/`](Assets/TextMesh%20Pro/Fonts/). Confirm license for your use case; example listing: [Valty DEMO on 1001 Fonts](https://www.1001fonts.com/valty-demo-font.html).

## Project structure

```text
.vscode/
Assets/
  Materials/
  Scenes/
  Scripts/
  Sounds/
  SpeechSDK/
    Plugins/
      Android/
      iOS/
      Linux/
      MacOS/
      WSA/
      x86/
      x86_64/
  TextMesh Pro/
    Documentation/
    Fonts/
    Resources/
      Fonts & Materials/
      Sprite Assets/
      Style Sheets/
    Shaders/
    Sprites/
Packages/
ProjectSettings/
```

- **[`Assets/Scripts/`](Assets/Scripts/)** — Ollama client, TTS/STT glue, ElevenLabs saver, [`TestOllama.cs`](Assets/Scripts/TestOllama.cs).
- **[`Assets/Scenes/`](Assets/Scenes/)** — Scene wiring (e.g. [`MainScene.unity`](Assets/Scenes/MainScene.unity)).
- **[`Assets/SpeechSDK/`](Assets/SpeechSDK/)** — Azure Speech binaries and legal files ([`LICENSE.md`](Assets/SpeechSDK/LICENSE.md), [`ThirdPartyNotices.md`](Assets/SpeechSDK/ThirdPartyNotices.md)).
- **[`Assets/TextMesh Pro/`](Assets/TextMesh%20Pro/)** — Fonts, SDF materials, shaders, **[`Sprites/`](Assets/TextMesh%20Pro/Sprites/)** (emoji atlas).
- **[`Packages/`](Packages/)** / **[`ProjectSettings/`](ProjectSettings/)** — UPM dependencies and Unity version.
- **[`.vscode/`](.vscode/)** — Workspace editor settings.

**Root:** [`.gitignore`](.gitignore), [`Babamın suggestionları.docx`](Babam%C4%B1n%20suggestionlar%C4%B1.docx), [`Scientific Paper About the Project and Contributions.pdf`](Scientific%20Paper%20About%20the%20Project%20and%20Contributions.pdf).
