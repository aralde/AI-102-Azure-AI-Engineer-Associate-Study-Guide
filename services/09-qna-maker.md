# Azure QnA Maker / Custom Question Answering

## Introduction

Custom Question Answering (formerly QnA Maker) creates FAQ-style bots from existing content like documents, URLs, and manual Q&A pairs. It's part of Azure AI Language service.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Knowledge Base (KB)** | Collection of Q&A pairs |
| **Source** | Where Q&A comes from (URL, file, manual) |
| **Multi-turn** | Conversation flow with follow-up prompts |
| **Active Learning** | Improve KB from user interactions |
| **Chit-chat** | Pre-built personality responses |

## Use Cases

- FAQ bots for customer support
- IT helpdesk automation
- HR policy bots
- Product documentation assistants
- Internal knowledge bases

---

## Creating a Knowledge Base

### Supported Data Sources

| Source Type | Format | Notes |
|-------------|--------|-------|
| **URLs** | FAQ web pages | Auto-extracts Q&A pairs |
| **Files** | PDF, DOCX, XLSX, TXT, TSV | Structured content works best |
| **Manual** | Direct input | Add custom Q&A pairs |
| **Chit-chat** | Pre-built | Add personality |

### Via Portal

1. Go to Language Studio
2. Create a Custom Question Answering project
3. Add sources (URLs, files)
4. Review and edit Q&A pairs
5. Train and deploy

---

## SDK Example (Python)

```bash
pip install azure-ai-language-questionanswering
```

### Query a Knowledge Base

```python
from azure.ai.language.questionanswering import QuestionAnsweringClient
from azure.core.credentials import AzureKeyCredential

client = QuestionAnsweringClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

# Simple query
response = client.get_answers(
    question="How do I reset my password?",
    project_name="my-kb-project",
    deployment_name="production",
    top=3  # Number of answers to return
)

for answer in response.answers:
    print(f"Answer: {answer.answer}")
    print(f"Confidence: {answer.confidence:.2f}")
    print(f"Source: {answer.source}")
```

### Multi-turn Conversation

```python
# First question
response = client.get_answers(
    question="I need help with my account",
    project_name="my-kb-project",
    deployment_name="production"
)

# The response might include follow-up prompts
answer = response.answers[0]
if answer.dialog:
    print("Follow-up prompts:")
    for prompt in answer.dialog.prompts:
        print(f"  - {prompt.display_text}")

# User selects a follow-up prompt
# Include context in the next query
response = client.get_answers(
    question="Reset password",  # User's selection
    project_name="my-kb-project",
    deployment_name="production",
    answer_context={
        "previous_qna_id": answer.qna_id,  # Link to previous answer
        "previous_question": "I need help with my account"
    }
)
```

### With Filters

```python
response = client.get_answers(
    question="What's the return policy?",
    project_name="my-kb-project",
    deployment_name="production",
    filters={
        "metadata_filter": {
            "metadata": [
                {"key": "category", "value": "policies"}
            ]
        }
    },
    confidence_threshold=0.5  # Minimum confidence
)
```

---

## REST API Example

### Query Knowledge Base

```http
POST https://<endpoint>/language/:query-knowledgebases?projectName=my-kb&deploymentName=production&api-version=2021-10-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "question": "How do I reset my password?",
    "top": 3,
    "confidenceScoreThreshold": 0.5
}
```

### Multi-turn Query

```http
POST https://<endpoint>/language/:query-knowledgebases?projectName=my-kb&deploymentName=production&api-version=2021-10-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "question": "Reset password",
    "top": 3,
    "context": {
        "previousQnaId": 5,
        "previousUserQuery": "I need help with my account"
    }
}
```

### Response with Follow-up Prompts

```json
{
    "answers": [
        {
            "id": 5,
            "answer": "To reset your password, go to Settings > Security > Reset Password.",
            "confidenceScore": 0.95,
            "source": "faq.pdf",
            "dialog": {
                "prompts": [
                    {
                        "displayOrder": 1,
                        "displayText": "I forgot my email too",
                        "qnaId": 12
                    },
                    {
                        "displayOrder": 2,
                        "displayText": "Two-factor authentication issues",
                        "qnaId": 15
                    }
                ]
            }
        }
    ]
}
```

---

## Cheatsheet

### Multi-turn Conversation

| UI Term | JSON Property | Purpose |
|---------|---------------|---------|
| Follow-up prompts | `dialog.prompts` | Guide user to next question |
| Previous Q&A ID | `context.previousQnaId` | Link conversation history |

**Exam Question:** "How to configure conversation flow?"
**Answer:** Add follow-up prompts in the portal, or use `context` object in API.

### Active Learning

Improves KB based on real user queries:

1. **Enable** Active Learning in settings
2. **Review** suggestions in the portal
3. **Accept/Reject** suggested Q&A pairs
4. **Retrain** the model

**Exam Keywords:** "Implicit feedback", "Review suggestions", "Improve over time"

### Chit-chat Personalities

| Personality | Tone |
|-------------|------|
| Professional | Formal, business-like |
| Friendly | Warm, casual |
| Witty | Humorous, clever |
| Caring | Empathetic, supportive |
| Enthusiastic | Excited, energetic |

**Exam Question:** "Bot should respond with humor"
**Answer:** Add Chit-chat source with "Witty" personality

### Confidence Threshold

```python
# Set minimum confidence for answers
response = client.get_answers(
    question="...",
    confidence_threshold=0.5  # 0.0 to 1.0
)

# If no answer meets threshold, returns "No good match found"
```

### Metadata Filtering

Tag Q&A pairs with metadata for filtering:

```json
{
    "question": "What's the return policy?",
    "filters": {
        "metadata_filter": {
            "metadata": [
                {"key": "product", "value": "electronics"},
                {"key": "region", "value": "US"}
            ],
            "logicalOperation": "AND"
        }
    }
}
```

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| QnA Maker is separate service | ❌ Now part of Azure AI Language |
| Multi-turn = multiple questions | ❌ It's guided conversation flow |
| Active Learning = automatic | ❌ You must review and approve |
| Chit-chat = custom training | ❌ It's pre-built, just select personality |

### Package Name

```bash
# Correct package for Custom Question Answering
pip install azure-ai-language-questionanswering

# NOT azure-ai-qnamaker (old)
```

### Export/Backup

Export your KB for backup or migration:

| Format | Use Case |
|--------|----------|
| TSV | Simple Q&A pairs |
| Excel | Full export with metadata |
| JSON | Programmatic access |

### Integration with Bot Framework

```python
# In your bot code
from botbuilder.ai.qna import QnAMaker, QnAMakerEndpoint

qna_maker = QnAMaker(
    QnAMakerEndpoint(
        knowledge_base_id="<kb-id>",
        endpoint_key="<endpoint-key>",
        host="<host>"
    )
)

# In message handler
response = await qna_maker.get_answers(turn_context)
```

---

**Previous:** [Content Safety](08-content-safety.md) | **Next:** [AI Agents](10-ai-agents.md)

