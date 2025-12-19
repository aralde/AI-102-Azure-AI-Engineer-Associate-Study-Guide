# Azure AI Language (CLU & Text Analytics)

## Introduction

Azure AI Language provides NLP capabilities including sentiment analysis, entity recognition, key phrase extraction, and Conversational Language Understanding (CLU) for building chatbots.

### Service Components

| Component | Purpose | Package |
|-----------|---------|---------|
| Text Analytics | Sentiment, entities, PII, key phrases | `azure-ai-textanalytics` |
| CLU (Conversational Language Understanding) | Intents & entities for chatbots | `azure-ai-language-conversations` |
| Custom Question Answering | FAQ bots | `azure-ai-language-questionanswering` |

## Use Cases

- Sentiment analysis of customer reviews
- PII detection and redaction
- Chatbot intent recognition
- Entity extraction from documents
- Key phrase extraction for tagging
- Language detection

---

## Text Analytics SDK Example (Python)

```bash
pip install azure-ai-textanalytics
```

### Sentiment Analysis

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

documents = [
    "The food was amazing but the service was terrible.",
    "I love this product!"
]

result = client.analyze_sentiment(
    documents=documents,
    show_opinion_mining=True  # Aspect-based sentiment!
)

for doc in result:
    print(f"Overall: {doc.sentiment}")
    print(f"Scores: Pos={doc.confidence_scores.positive:.2f}, "
          f"Neg={doc.confidence_scores.negative:.2f}")
    
    # Opinion mining shows sentiment per aspect
    for sentence in doc.sentences:
        for opinion in sentence.mined_opinions:
            target = opinion.target
            print(f"  '{target.text}' is {target.sentiment}")
```

**Output with Opinion Mining:**
```
Overall: mixed
  'food' is positive
  'service' is negative
```

### Entity Recognition (NER)

```python
result = client.recognize_entities(
    documents=["Microsoft was founded by Bill Gates in 1975."]
)

for doc in result:
    for entity in doc.entities:
        print(f"{entity.text}: {entity.category} ({entity.confidence_score:.2f})")
```

**Output:**
```
Microsoft: Organization (0.99)
Bill Gates: Person (0.98)
1975: DateTime (0.95)
```

### PII Detection

```python
result = client.recognize_pii_entities(
    documents=["Call me at 555-123-4567 or email john@example.com"],
    categories_filter=["PhoneNumber", "Email"]  # Optional filter
)

for doc in result:
    print(f"Redacted: {doc.redacted_text}")
    for entity in doc.entities:
        print(f"  Found {entity.category}: {entity.text}")
```

**Output:**
```
Redacted: Call me at ************ or email ****************
  Found PhoneNumber: 555-123-4567
  Found Email: john@example.com
```

### Key Phrase Extraction

```python
result = client.extract_key_phrases(
    documents=["Azure AI services provide powerful machine learning capabilities."]
)

for doc in result:
    print(f"Key phrases: {', '.join(doc.key_phrases)}")
```

### Entity Linking (Wikipedia)

```python
result = client.recognize_linked_entities(
    documents=["I visited the Space Needle in Seattle."]
)

for doc in result:
    for entity in doc.entities:
        print(f"{entity.name}: {entity.url}")
```

### Abstractive Summarization

```python
from azure.ai.textanalytics import AbstractiveSummaryAction

poller = client.begin_analyze_actions(
    documents=["Long text to summarize..."],
    actions=[
        AbstractiveSummaryAction(sentence_count=3)
    ]
)

result = poller.result()
for doc_result in result:
    for action_result in doc_result:
        for summary in action_result.summaries:
            print(summary.text)
```

---

## CLU (Conversational Language Understanding)

### Concepts

| Concept | Definition | Example |
|---------|------------|---------|
| **Utterance** | What the user says | "Book a flight to Paris" |
| **Intent** | The action/verb | `BookFlight` |
| **Entity** | The parameters/nouns | `Destination=Paris` |
| **None Intent** | The "garbage bin" | "What's the weather?" (irrelevant) |

### Entity Types

| Type | Use Case | Exam Keywords |
|------|----------|---------------|
| **Learned** | Context-dependent, ML-based | "Train model", "Label utterances" |
| **List** | Fixed set of values | "Menu items", "Catalog", "Exact match" |
| **Prebuilt** | Common types (dates, numbers) | "Automatically detect", "Common types" |
| **Regex** | Pattern-based | "Order ID", "Structured format" |

### SDK Example

```bash
pip install azure-ai-language-conversations
```

```python
from azure.ai.language.conversations import ConversationAnalysisClient
from azure.core.credentials import AzureKeyCredential

client = ConversationAnalysisClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

result = client.analyze_conversation(
    task={
        "kind": "Conversation",
        "analysisInput": {
            "conversationItem": {
                "id": "1",
                "text": "Book a flight to Paris tomorrow",
                "participantId": "user"
            }
        },
        "parameters": {
            "projectName": "my-clu-project",
            "deploymentName": "production"
        }
    }
)

# Access the prediction
prediction = result["result"]["prediction"]
print(f"Top Intent: {prediction['topIntent']}")
print(f"Confidence: {prediction['intents'][0]['confidenceScore']}")

for entity in prediction["entities"]:
    print(f"Entity: {entity['category']} = {entity['text']}")
```

---

## REST API Example

### Text Analytics - Sentiment

```http
POST https://<endpoint>/language/:analyze-text?api-version=2023-04-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "kind": "SentimentAnalysis",
    "analysisInput": {
        "documents": [
            {"id": "1", "text": "I love this product!", "language": "en"}
        ]
    }
}
```

### CLU - Analyze Conversation

```http
POST https://<endpoint>/language/:analyze-conversations?api-version=2023-04-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "kind": "Conversation",
    "analysisInput": {
        "conversationItem": {
            "id": "1",
            "text": "Book a flight to Paris",
            "participantId": "user"
        }
    },
    "parameters": {
        "projectName": "my-project",
        "deploymentName": "production"
    }
}
```

---

## Cheatsheet

### Package Selection (Exam Trap!)

| If you need... | Package |
|----------------|---------|
| Sentiment, Entities, PII | `azure-ai-textanalytics` |
| Intents, Utterances, Chatbot | `azure-ai-language-conversations` |
| FAQ/Knowledge Base | `azure-ai-language-questionanswering` |

### Opinion Mining

```python
# The magic parameter for aspect-based sentiment
result = client.analyze_sentiment(
    documents=documents,
    show_opinion_mining=True  # <-- This is the key!
)
```

**Exam Question:** "How to know what users think about specific aspects (food vs service)?"
**Answer:** `show_opinion_mining=True`

### Entity Type Decision Tree

```
Is it a universal type (date, number, email)?
  → Prebuilt Entity

Is it a fixed list you control (menu items)?
  → List Entity

Does it follow a pattern (ID-12345)?
  → Regex Entity

Does it depend on context (reason for return)?
  → Learned Entity
```

### None Intent Best Practices

| Rule | Value |
|------|-------|
| Required? | Yes, mandatory |
| Data percentage | 10-20% of total utterances |
| Purpose | Reduce false positives |
| Content | Irrelevant phrases ("Hello", "What time is it?") |

### REST API `kind` Values

| Kind | Purpose |
|------|---------|
| `SentimentAnalysis` | Sentiment |
| `EntityRecognition` | NER |
| `KeyPhraseExtraction` | Key phrases |
| `PiiEntityRecognition` | PII detection |
| `Conversation` | CLU intent recognition |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Using Language for PDFs | ❌ Use Document Intelligence first |
| `textanalytics` for intents | ❌ Use `language-conversations` |
| Learned entity for dates | ❌ Use Prebuilt entity |
| List entity for "reason" | ❌ Use Learned entity (context-dependent) |
| Skipping None intent | ❌ Always include 10-20% None data |

### Confidence Score Handling

```python
# In your code, always check confidence
if prediction['intents'][0]['confidenceScore'] < 0.5:
    return "I didn't understand that"
```

### Input Rule

**Azure AI Language only accepts TEXT (strings).**

If your input is:
- PDF → Use Document Intelligence first
- Image → Use Computer Vision OCR first
- Audio → Use Speech-to-Text first

Then pass the extracted text to Language.

---

**Previous:** [Document Intelligence](04-document-intelligence.md) | **Next:** [Azure AI Search](06-azure-ai-search.md)

