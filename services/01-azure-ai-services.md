# Azure AI Services - Getting Started

## Introduction

Azure AI Services is a collection of cloud-based AI services that enable developers to build intelligent applications. It provides a **single endpoint** for multiple AI capabilities, simplifying deployment and management.

### What's Included in Azure AI Services (Multi-service Resource)

- Azure AI Vision
- Azure AI Speech
- Azure AI Language
- Azure AI Translator
- Azure AI Face
- Azure AI Custom Vision
- Azure AI Document Intelligence

### Azure AI Foundry (Includes Additional Services)

- Everything above, plus:
- Azure OpenAI
- Azure AI Content Safety
- Azure AI Content Understanding

## Use Cases

| Scenario | Service to Use |
|----------|----------------|
| Extract text from images | AI Vision (OCR/Read) |
| Transcribe audio to text | AI Speech |
| Analyze document sentiment | AI Language |
| Process invoices automatically | Document Intelligence |
| Build a chatbot | Language + OpenAI |
| Detect inappropriate content | Content Safety |

## SDK Example (Python)

### Installation

```bash
pip install azure-ai-textanalytics azure-core
```

### Basic Pattern - All Services Follow This

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# 1. Create the client (Pattern: [ServiceName]Client)
client = TextAnalyticsClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

# 2. Call the method
result = client.analyze_sentiment(documents=["I love Azure AI!"])

# 3. Process results
for doc in result:
    print(f"Sentiment: {doc.sentiment}")
```

### Using Managed Identity (No Keys!)

```python
from azure.identity import DefaultAzureCredential

# Best practice for production - no hardcoded keys
credential = DefaultAzureCredential()
client = TextAnalyticsClient(endpoint="...", credential=credential)
```

## REST API Example

### Authentication Header

All REST calls require this header:

```http
Ocp-Apim-Subscription-Key: <your-api-key>
Content-Type: application/json
```

### Example: Text Analytics Sentiment

```http
POST https://<endpoint>/text/analytics/v3.1/sentiment
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "documents": [
        {"id": "1", "text": "I love this product!", "language": "en"}
    ]
}
```

### Response

```json
{
    "documents": [{
        "id": "1",
        "sentiment": "positive",
        "confidenceScores": {
            "positive": 0.98,
            "neutral": 0.01,
            "negative": 0.01
        }
    }]
}
```

## Cheatsheet

### CLI Commands

```bash
# Create a multi-service AI resource
az cognitiveservices account create \
    --name myAIService \
    --resource-group myRG \
    --kind CognitiveServices \
    --sku S0 \
    --location eastus

# Get the keys
az cognitiveservices account keys list \
    --name myAIService \
    --resource-group myRG

# Show resource details
az cognitiveservices account show \
    --name myAIService \
    --resource-group myRG
```

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| `az account list` for AI services | ❌ Wrong! Use `az cognitiveservices account list` |
| `az account` = AI resource | ❌ Wrong! `az account` = Subscription/Login |
| `list` vs `show` | `list` = all resources, `show` = one specific |

### Authentication Options

| Method | When to Use | Keywords in Exam |
|--------|-------------|------------------|
| `AzureKeyCredential` | Simple scenarios | "API Key", "Quick setup" |
| `DefaultAzureCredential` | Production, no keys in code | "RBAC", "Managed Identity", "Secure" |

### Pricing Tiers

| Tier | Use Case | Exam Tip |
|------|----------|----------|
| F0 (Free) | Development/Testing | ❌ Never for "SLA" or "Production" |
| S0 (Standard) | Production | ✅ Always for "SLA" questions |

### Container Deployment

When to use containers:
- "Data cannot leave the network"
- "On-premise deployment"
- "Strict compliance"
- "No internet connection"

**Required Docker arguments:**
```bash
docker run --rm -it \
    -e Eula=accept \                    # Required!
    -e Billing=<endpoint> \             # Required!
    -e ApiKey=<key> \                   # Required!
    mcr.microsoft.com/azure-cognitive-services/...
```

### Security Best Practices

| Requirement | Solution |
|-------------|----------|
| Store keys securely | Azure Key Vault |
| Connect without managing keys | Managed Identity |
| Private network traffic | Private Endpoint |
| Restrict access by network | Service Endpoint + Firewall |
| Encrypt with your own keys | Customer Managed Keys (CMK) |

### HTTP Status Codes

| Code | Meaning | Common Cause |
|------|---------|--------------|
| 200 | Success | Request completed |
| 202 | Accepted | Long-running operation started (Document Intelligence, Training) |
| 401 | Unauthorized | Wrong key or endpoint |
| 429 | Too Many Requests | Throttling - implement exponential backoff |
| 503 | Service Unavailable | Service overloaded |

### Resource Naming (--kind parameter)

| Kind Value | Service |
|------------|---------|
| `CognitiveServices` | Multi-service (all-in-one) |
| `TextAnalytics` | Language service only |
| `ComputerVision` | Vision service only |
| `Face` | Face API only |
| `SpeechServices` | Speech only |

---

**Next:** [Azure AI Vision & Custom Vision](02-azure-ai-vision.md)

