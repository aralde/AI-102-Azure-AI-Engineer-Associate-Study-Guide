# AI-102 Master Cheatsheet 🎯

> **Quick reference for exam day. Print this or keep it handy!**

---

## 🔑 The Import Trick (Most Important!)

**In code completion questions, 90% of answers are in the import statements!**

```python
from azure.ai.textanalytics import TextAnalyticsClient  # <- Answer is here!
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(...)  # Slot 1: TextAnalyticsClient
```

---

## 📦 SDK Package Names

| Service | Package | Remember |
|---------|---------|----------|
| Text Analytics | `azure-ai-textanalytics` | Sentiment, NER, PII |
| CLU (Intents) | `azure-ai-language-conversations` | Chatbot intents |
| QnA | `azure-ai-language-questionanswering` | FAQ bots |
| Document Intelligence | `azure-ai-documentintelligence` | PDFs, invoices |
| Vision (Standard) | `azure-ai-vision-imageanalysis` | OCR, tags |
| Custom Vision | `azure-cognitiveservices-vision-customvision` | Train your own |
| Speech | `azure-cognitiveservices-speech` | STT, TTS |
| Search | `azure-search-documents` | No "ai" in name! |
| Content Safety | `azure-ai-contentsafety` | Moderation |
| OpenAI | `openai` | Standard library |

---
## 🔑 Key Exam Tips

### The Import Trick (Most Important!)
In code completion questions, **90% of the time the answer is in the import statements**. Look at what's imported!

### SDK Naming Patterns
- Client: `[ServiceName]Client`
- Authentication: `AzureKeyCredential` or `DefaultAzureCredential`
- Options: `[Action]Options`

## 🖥️ CLI Commands

### The Golden Rule
```bash
# ❌ WRONG - This is for subscriptions/login
az account list

# ✅ CORRECT - This is for AI resources
az cognitiveservices account list
```

### Essential Commands
```bash
# Create AI resource
az cognitiveservices account create --name X --kind CognitiveServices --sku S0

# Get keys
az cognitiveservices account keys list --name X --resource-group Y

# Show details (singular)
az cognitiveservices account show --name X --resource-group Y

# List all (plural)
az cognitiveservices account list --resource-group Y
```
---

## 🌐 HTTP Methods

| Method | Use Case | Example |
|--------|----------|---------|
| **POST** | Send data for analysis | Analyze text, search queries |
| **GET** | Check status, list items | Polling, get results |
| **PUT** | Create/update infrastructure | Create index, upload model |

### HTTP Status Codes
| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process result |
| 202 | Accepted (async) | Poll `Operation-Location` |
| 401 | Unauthorized | Check key/endpoint |
| 429 | Throttling | Exponential backoff |

---

## 🔐 Authentication

| Method | Use When | Code |
|--------|----------|------|
| API Key | Simple/quick | `AzureKeyCredential(key)` |
| Managed Identity | Production, no keys | `DefaultAzureCredential()` |

### Headers for REST
```http
Ocp-Apim-Subscription-Key: <your-key>
Content-Type: application/json
```

### Translator Extra Header (Trap!)
```http
Ocp-Apim-Subscription-Region: <region>  # Required if not global!
```

---

## 📊 Service Selection Guide

| Input Type | Service |
|------------|---------|
| Plain text (string) | AI Language |
| PDF, Image, Scan | Document Intelligence |
| Audio file | Speech Service |
| Video file | Video Indexer |

### Decision Tree
```
Is it a PDF/Image?
  → Document Intelligence (extract text first)
  
Is it audio?
  → Speech Service (transcribe first)
  
Is it plain text?
  → AI Language (analyze directly)
```

---

## 🔍 Azure AI Search

### Scaling
| Problem | Solution |
|---------|----------|
| Queries slow/throttling | Add **R**eplicas (R = Read) |
| Index too big | Add **P**artitions (P = Parallel) |
| Indexing slow | Multiple Indexers + Virtual Folders |

### Pipeline Order (Drag & Drop)
```
1. Data Source → 2. Skillset → 3. Index → 4. Indexer
```

### Field Mappings
| Type | Source | Use |
|------|--------|-----|
| `fieldMappings` | Raw data | Metadata → Index |
| `outputFieldMappings` | Skill output | AI enrichment → Index |

---

## 🗣️ Speech Service

### Configuration Properties
| Property | Purpose | Syntax |
|----------|---------|--------|
| `speech_recognition_language` | STT input | `= "en-US"` |
| `speech_synthesis_voice_name` | TTS voice | `= "en-US-JennyNeural"` |
| `add_target_language()` | Translation output | `.add("es")` |

### SSML Quick Reference
| Need | Tag |
|------|-----|
| Speed/pitch/volume | `<prosody>` |
| Emotion/style | `<mstts:express-as style="...">` |
| Pronunciation fix | `<phoneme>` |
| Date/number format | `<say-as interpret-as="...">` |
| Pause | `<break time="500ms"/>` |

---

## 🤖 CLU Entity Types

| Type | Use When |
|------|----------|
| **Learned** | Context-dependent, ML-based |
| **List** | Fixed set (menu items) |
| **Prebuilt** | Common types (date, email) |
| **Regex** | Pattern (ID-12345) |

### None Intent
- **Required:** Yes, always
- **Data:** 10-20% of utterances
- **Purpose:** Reduce false positives

---

## 📄 Document Intelligence Models

| Model | Use For |
|-------|---------|
| `prebuilt-read` | Text only (no tables) |
| `prebuilt-layout` | Text + Tables + Checkboxes |
| `prebuilt-invoice` | B2B invoices |
| `prebuilt-receipt` | B2C receipts |
| `prebuilt-idDocument` | Passports, IDs |

**Trap:** Tables/Checkboxes → `prebuilt-layout`, NOT `prebuilt-read`!

---

## 🖼️ Custom Vision

### Precision vs Recall
| Metric | Formula | Optimize When |
|--------|---------|---------------|
| Precision | TP/(TP+FP) | False positives costly |
| Recall | TP/(TP+FN) | Can't miss anything |

### Threshold Slider
- ⬆️ Higher threshold = ⬆️ Precision, ⬇️ Recall
- ⬇️ Lower threshold = ⬇️ Precision, ⬆️ Recall

### Domains
- **General:** Cloud only, highest accuracy
- **Compact:** Export to mobile/edge/Docker

---

## 🧠 Azure OpenAI

### The Deployment Trap
```python
# ❌ WRONG
model="gpt-4"

# ✅ CORRECT (your deployment name)
model="my-gpt4-deployment"
```

### Temperature
| Value | Use Case |
|-------|----------|
| 0.0-0.3 | Data extraction, facts |
| 0.5-0.7 | General chat |
| 0.7-1.0 | Creative writing |

### Message Roles
| Role | Purpose |
|------|---------|
| `system` | Set behavior (priming) |
| `user` | User input |
| `assistant` | Previous AI responses |

### Prompt Techniques
| Technique | Pattern |
|-----------|---------|
| Zero-shot | Just the question |
| Few-shot | Examples before question |
| Chain-of-Thought | "Think step by step" |
| Priming | "You are a..." |

---

## 🤖 AI Agents

### The Trinity
```
Assistant (brain) → Thread (memory) → Run (action)
```

### Run Status
| Status | Action |
|--------|--------|
| `queued` | Wait |
| `in_progress` | Wait |
| `requires_action` | Submit tool outputs |
| `completed` | Get messages |

### Tool Selection
| Need | Tool |
|------|------|
| Execute Python | Code Interpreter |
| Search documents | File Search |
| Call external API | Function Calling |

---

## 🛡️ Content Safety

### The 4 Categories
**H**ate, **V**iolence, **S**exual, **S**elf-harm

### Severity Levels
| Level | Action |
|-------|--------|
| 0 | Safe |
| 2 | Low - review |
| 4 | Medium - flag |
| 6 | High - block |

---

## 🐳 Containers

### Required Arguments
```bash
docker run \
  -e Eula=accept \        # Required!
  -e Billing=<endpoint> \ # Required!
  -e ApiKey=<key> \       # Required!
  <image>
```

### Use When
- "Data cannot leave network"
- "On-premise"
- "No internet"
- "Strict compliance"

---

## ⚡ Quick Exam Tips

1. **Read the last sentence first** - that's what they're asking
2. **Case studies:** Go to question, then search text
3. **SLA questions:** Never choose F0 (Free)
4. **Security questions:** Key Vault for keys, Managed Identity for no keys
5. **Time management:** 60 questions in 80 minutes = ~1.3 min/question

---

## 🚨 Common Traps

| Trap | Correct |
|------|---------|
| `az account` for AI | `az cognitiveservices account` |
| Language for PDFs | Document Intelligence |
| `prebuilt-read` for tables | `prebuilt-layout` |
| `model="gpt-4"` | Use deployment name |
| Chat API is stateful | Stateless - you manage history |
| Replicas for indexing | Replicas for queries |
| Key Phrase for synonyms | Synonym Map |

---

**Good luck! You've got this! 🍀**

