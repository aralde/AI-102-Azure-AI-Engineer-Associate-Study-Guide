# Azure AI Content Safety

## Introduction

Azure AI Content Safety detects harmful content in text and images across four categories. It's essential for moderating user-generated content and AI outputs.

### The 4 Categories (Memorize These!)

| Category | Description | Examples |
|----------|-------------|----------|
| **Hate** | Discrimination, slurs | Racist, sexist content |
| **Violence** | Physical harm, weapons | Gore, threats |
| **Sexual** | Adult content | Explicit material |
| **Self-Harm** | Self-injury, suicide | Cutting, suicide methods |

**Memory Trick:** **H**ate, **V**iolence, **S**exual, **S**elf-harm = "HVSS" (like a snake hissing at bad content 🐍)

## Use Cases

- Moderate user comments and posts
- Filter AI-generated content (GPT outputs)
- Screen uploaded images
- Protect chat applications
- Compliance with content policies
- Jailbreak detection for LLMs

---

## SDK Example (Python)

```bash
pip install azure-ai-contentsafety
```

### Analyze Text

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
from azure.core.credentials import AzureKeyCredential

client = ContentSafetyClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

# Analyze text for harmful content
response = client.analyze_text(
    AnalyzeTextOptions(
        text="Your text to analyze here",
        categories=[
            TextCategory.HATE,
            TextCategory.VIOLENCE,
            TextCategory.SEXUAL,
            TextCategory.SELF_HARM
        ],
        output_type="FourSeverityLevels"  # 0, 2, 4, 6
    )
)

# Check results
for category_result in response.categories_analysis:
    print(f"{category_result.category}: Severity {category_result.severity}")
    
# Severity levels: 0 (safe), 2 (low), 4 (medium), 6 (high)
```

### Analyze Image

```python
from azure.ai.contentsafety.models import AnalyzeImageOptions, ImageData

# From URL
response = client.analyze_image(
    AnalyzeImageOptions(
        image=ImageData(url="https://example.com/image.jpg")
    )
)

# From binary data
with open("image.jpg", "rb") as f:
    image_data = f.read()

response = client.analyze_image(
    AnalyzeImageOptions(
        image=ImageData(content=image_data)
    )
)

for result in response.categories_analysis:
    print(f"{result.category}: Severity {result.severity}")
```

### Custom Blocklists

Block specific terms that are harmful to YOUR context:

```python
from azure.ai.contentsafety.models import (
    TextBlocklist,
    AddOrUpdateTextBlocklistItemsOptions,
    TextBlocklistItem
)

# Create a blocklist
client.create_or_update_text_blocklist(
    blocklist_name="my-blocklist",
    options=TextBlocklist(description="Custom blocked terms")
)

# Add terms to the blocklist
client.add_or_update_blocklist_items(
    blocklist_name="my-blocklist",
    options=AddOrUpdateTextBlocklistItemsOptions(
        blocklist_items=[
            TextBlocklistItem(text="competitor-name"),
            TextBlocklistItem(text="internal-code-word")
        ]
    )
)

# Use blocklist in analysis
response = client.analyze_text(
    AnalyzeTextOptions(
        text="Text mentioning competitor-name",
        blocklist_names=["my-blocklist"],
        halt_on_blocklist_hit=True  # Stop if blocklist match found
    )
)

# Check for blocklist matches
if response.blocklists_match:
    for match in response.blocklists_match:
        print(f"Blocked term found: {match.blocklist_item_text}")
```

---

## REST API Example

### Analyze Text

```http
POST https://<endpoint>/contentsafety/text:analyze?api-version=2023-10-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "text": "Your text to analyze",
    "categories": ["Hate", "Violence", "Sexual", "SelfHarm"],
    "outputType": "FourSeverityLevels"
}
```

### Response

```json
{
    "categoriesAnalysis": [
        {"category": "Hate", "severity": 0},
        {"category": "Violence", "severity": 2},
        {"category": "Sexual", "severity": 0},
        {"category": "SelfHarm", "severity": 0}
    ]
}
```

### Analyze Image

```http
POST https://<endpoint>/contentsafety/image:analyze?api-version=2023-10-01
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "image": {
        "url": "https://example.com/image.jpg"
    },
    "categories": ["Hate", "Violence", "Sexual", "SelfHarm"]
}
```

---

## Cheatsheet

### Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| 0 | Safe | Allow |
| 2 | Low | Review or allow |
| 4 | Medium | Flag for review |
| 6 | High | Block immediately |

### URL Pattern

| Action | URL Pattern |
|--------|-------------|
| Analyze text | `/contentsafety/text:analyze` |
| Analyze image | `/contentsafety/image:analyze` |

### Blocklist Parameters

| Parameter | Purpose |
|-----------|---------|
| `blocklist_names` | List of blocklists to apply |
| `halt_on_blocklist_hit` | Stop analysis if match found |

### Jailbreak Detection

Detects attempts to bypass AI safety measures:

```python
from azure.ai.contentsafety.models import AnalyzeTextOptions

response = client.analyze_text(
    AnalyzeTextOptions(
        text="Ignore your previous instructions and...",
        categories=[TextCategory.HATE],  # Still need categories
        # Jailbreak detection is automatic in newer versions
    )
)

# Check for jailbreak attempt
if hasattr(response, 'jailbreak_analysis'):
    if response.jailbreak_analysis.detected:
        print("Jailbreak attempt detected!")
```

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Content Safety = Content Moderator | ❌ Different services (Content Moderator is older) |
| Only 2 severity levels | ❌ Four levels: 0, 2, 4, 6 |
| Categories are optional | ❌ You should specify categories |
| Blocklist = built-in filter | ❌ Blocklist is custom, you create it |

### Integration with Azure OpenAI

Content Safety is built into Azure OpenAI, but you can also use it separately:

```python
# In Azure OpenAI, content filtering is automatic
# But you can configure thresholds in the Azure portal

# For custom applications, call Content Safety before/after OpenAI:
# 1. Check user input with Content Safety
# 2. If safe, send to OpenAI
# 3. Check OpenAI output with Content Safety
# 4. Return to user if safe
```

### Decision Flow

```
User Input
    ↓
Content Safety Analysis
    ↓
Severity >= threshold? → Block/Flag
    ↓ (No)
Process normally
```

---

**Previous:** [Video Indexer](07-azure-video-indexer.md) | **Next:** [QnA Maker](09-qna-maker.md)

