# Azure AI Speech

## Introduction

Azure AI Speech provides speech-to-text (STT), text-to-speech (TTS), speech translation, and speaker recognition capabilities.

### Core Capabilities

| Capability | Description |
|------------|-------------|
| **Speech-to-Text (STT)** | Transcribe audio to text |
| **Text-to-Speech (TTS)** | Convert text to natural-sounding audio |
| **Speech Translation** | Translate spoken language in real-time |
| **Speaker Recognition** | Identify or verify speakers |
| **Pronunciation Assessment** | Evaluate pronunciation accuracy |

## Use Cases

- Real-time meeting transcription
- Voice assistants and IVR systems
- Accessibility features (read aloud)
- Multi-language customer support
- Podcast/video captioning
- Language learning applications

---

## SDK Example (Python)

```bash
pip install azure-cognitiveservices-speech
```

### Speech-to-Text (Basic)

```python
import azure.cognitiveservices.speech as speechsdk

# Configuration
speech_config = speechsdk.SpeechConfig(
    subscription="<your-key>",
    region="<your-region>"  # e.g., "eastus"
)
speech_config.speech_recognition_language = "en-US"

# Audio input (microphone or file)
audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
# OR from file:
# audio_config = speechsdk.audio.AudioConfig(filename="audio.wav")

# Create recognizer
recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config
)

# Single recognition (waits for one phrase)
result = recognizer.recognize_once()

if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print(f"Recognized: {result.text}")
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("No speech could be recognized")
```

### Speech-to-Text with Phrase List (Improve Accuracy)

```python
# Add domain-specific words to improve recognition
phrase_list = speechsdk.PhraseListGrammar.from_recognizer(recognizer)
phrase_list.addPhrase("Contoso")
phrase_list.addPhrase("Azure")
phrase_list.addPhrase("Kubernetes")

# Now recognition will be better for these words
result = recognizer.recognize_once()
```

### Continuous Recognition (For Long Audio)

```python
def recognized_callback(evt):
    print(f"RECOGNIZED: {evt.result.text}")

def recognizing_callback(evt):
    print(f"RECOGNIZING: {evt.result.text}")  # Partial results

recognizer.recognized.connect(recognized_callback)
recognizer.recognizing.connect(recognizing_callback)

# Start continuous recognition
recognizer.start_continuous_recognition()

# ... do other things ...

# Stop when done
recognizer.stop_continuous_recognition()
```

### Text-to-Speech (Basic)

```python
speech_config = speechsdk.SpeechConfig(subscription="<key>", region="<region>")

# Set the voice
speech_config.speech_synthesis_voice_name = "en-US-JennyNeural"

synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)

# Speak text
result = synthesizer.speak_text_async("Hello, world!").get()

# Or save to file
audio_config = speechsdk.audio.AudioOutputConfig(filename="output.wav")
synthesizer = speechsdk.SpeechSynthesizer(
    speech_config=speech_config,
    audio_config=audio_config
)
synthesizer.speak_text_async("Hello, world!").get()
```

### Text-to-Speech with SSML (Advanced Control)

```python
ssml = """
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        <mstts:express-as style="cheerful">
            Hello! I'm so happy to help you today!
        </mstts:express-as>
        <break time="500ms"/>
        <prosody rate="+10%" pitch="high">
            This is faster and higher pitched.
        </prosody>
    </voice>
</speak>
"""

result = synthesizer.speak_ssml_async(ssml).get()
```

### Speech Translation

```python
# Translation config (different from SpeechConfig!)
translation_config = speechsdk.translation.SpeechTranslationConfig(
    subscription="<key>",
    region="<region>"
)

# Source language (what you're speaking)
translation_config.speech_recognition_language = "en-US"

# Target languages (can add multiple!)
translation_config.add_target_language("es")
translation_config.add_target_language("fr")
translation_config.add_target_language("de")

# Create translator
translator = speechsdk.translation.TranslationRecognizer(
    translation_config=translation_config,
    audio_config=audio_config
)

result = translator.recognize_once()

if result.reason == speechsdk.ResultReason.TranslatedSpeech:
    print(f"Original: {result.text}")
    print(f"Spanish: {result.translations['es']}")
    print(f"French: {result.translations['fr']}")
```

---

## REST API Example

### Speech-to-Text

```http
POST https://<region>.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
Content-Type: audio/wav
Ocp-Apim-Subscription-Key: <key>

[Binary audio data]
```

### Text-to-Speech

```http
POST https://<region>.tts.speech.microsoft.com/cognitiveservices/v1
Content-Type: application/ssml+xml
Ocp-Apim-Subscription-Key: <key>
X-Microsoft-OutputFormat: audio-16khz-128kbitrate-mono-mp3

<speak version="1.0" xml:lang="en-US">
    <voice name="en-US-JennyNeural">
        Hello, world!
    </voice>
</speak>
```

---

## SSML Cheatsheet (Speech Synthesis Markup Language)

### The 4 Categories of SSML

| Category | Tag | Use Case |
|----------|-----|----------|
| **DJ/Mixer** (Physical) | `<prosody>` | Speed, pitch, volume |
| **Actor** (Emotional) | `<mstts:express-as>` | Style, role |
| **Corrector** (Pronunciation) | `<phoneme>`, `<say-as>`, `<sub>` | Fix pronunciation |
| **Silence** | `<break>` | Pauses |

### Prosody (Physical Audio Properties)

```xml
<prosody rate="+20%">Faster speech</prosody>
<prosody rate="-20%">Slower speech</prosody>
<prosody pitch="high">Higher pitch</prosody>
<prosody pitch="low">Lower pitch</prosody>
<prosody volume="loud">Louder</prosody>
```

### Express-As (Emotions and Roles)

```xml
<!-- Emotions (Styles) -->
<mstts:express-as style="cheerful">I'm happy!</mstts:express-as>
<mstts:express-as style="sad">I'm sad.</mstts:express-as>
<mstts:express-as style="angry">I'm angry!</mstts:express-as>
<mstts:express-as style="empathetic">I understand.</mstts:express-as>
<mstts:express-as style="newscast">Breaking news...</mstts:express-as>

<!-- Roles (Different personas) -->
<mstts:express-as role="SeniorMale">Older male voice</mstts:express-as>
<mstts:express-as role="Girl">Young girl voice</mstts:express-as>
```

### Pronunciation Correction

```xml
<!-- Say-As: Interpret ambiguous text -->
<say-as interpret-as="date">03/04/2024</say-as>
<say-as interpret-as="telephone">555-123-4567</say-as>
<say-as interpret-as="cardinal">123</say-as>

<!-- Phoneme: Exact pronunciation -->
<phoneme alphabet="ipa" ph="təˈmeɪtoʊ">tomato</phoneme>

<!-- Sub: Simple substitution -->
<sub alias="Microsoft">MSFT</sub>
```

### Breaks (Pauses)

```xml
<break time="500ms"/>
<break time="2s"/>
<break strength="medium"/>  <!-- Like a comma -->
<break strength="strong"/>  <!-- Like a period -->
```

### Audio Effects

```xml
<!-- For car interior optimization -->
<voice name="en-US-JennyNeural" effect="eq_car">
    Navigation instructions
</voice>

<!-- For telephone/low bandwidth -->
<voice name="en-US-JennyNeural" effect="eq_telecomhp8k">
    Phone menu options
</voice>
```

### Background Audio

```xml
<audio src="https://example.com/background.mp3">
    Fallback text if audio fails
</audio>
```

---

## Cheatsheet

### Configuration Properties

| Property | Purpose | Syntax |
|----------|---------|--------|
| `speech_recognition_language` | STT input language | `= "en-US"` (property) |
| `speech_synthesis_voice_name` | TTS voice | `= "en-US-JennyNeural"` (property) |
| `add_target_language()` | Translation output | `.add_target_language("es")` (method) |

**Memory Trick:**
- Input language = Property (single value, use `=`)
- Output languages = Method (can add multiple, use `.add()`)

### Profanity Handling

```python
speech_config.set_profanity(speechsdk.ProfanityOption.Masked)   # *** 
speech_config.set_profanity(speechsdk.ProfanityOption.Removed)  # Deleted
speech_config.set_profanity(speechsdk.ProfanityOption.Raw)      # As-is
```

### Word-Level Timestamps

```python
speech_config.request_word_level_timestamps()
# Returns JSON with timing for each word
```

### Recognition Modes

| Mode | Method | Use Case |
|------|--------|----------|
| Single-shot | `recognize_once()` | Short phrases, commands |
| Continuous | `start_continuous_recognition()` | Long audio, meetings |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Using `speech_recognition_language` for TTS | ❌ Use `speech_synthesis_voice_name` |
| `SpeechConfig` for translation | ❌ Use `SpeechTranslationConfig` |
| Setting multiple source languages | ❌ Only ONE source language allowed |
| `prosody` for emotions | ❌ Use `mstts:express-as style="..."` |
| `phoneme` for date formatting | ❌ Use `say-as interpret-as="date"` |

### SSML Decision Tree

| If the exam says... | Use this tag |
|--------------------|--------------|
| "Change speed/rate" | `<prosody rate="...">` |
| "Make it sound happy/sad" | `<mstts:express-as style="...">` |
| "Read 12345 as phone number" | `<say-as interpret-as="telephone">` |
| "Correct pronunciation of a name" | `<phoneme>` |
| "Add a pause" | `<break>` |
| "Optimize for car speakers" | `<voice effect="eq_car">` |

### Visemes (Lip Sync)

For avatar/lip sync applications:

```python
def viseme_callback(evt):
    print(f"Viseme ID: {evt.viseme_id}")  # 0-21 for mouth shapes

synthesizer.viseme_received.connect(viseme_callback)
```

---

**Previous:** [Azure AI Vision](02-azure-ai-vision.md) | **Next:** [Document Intelligence](04-document-intelligence.md)

