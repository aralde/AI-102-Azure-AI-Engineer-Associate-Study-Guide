# Azure Video Indexer

## Introduction

Azure Video Indexer analyzes video and audio content to extract insights like transcripts, faces, emotions, topics, brands, and more. It combines multiple AI capabilities into a single service.

### Key Insights Extracted

| Insight Type | Description |
|--------------|-------------|
| **Transcript** | Speech-to-text with speaker identification |
| **OCR** | Text visible in the video |
| **Faces** | Face detection and identification |
| **Emotions** | Facial emotion analysis |
| **Topics** | Main topics discussed |
| **Keywords** | Key phrases mentioned |
| **Brands** | Brand mentions (visual and audio) |
| **Scenes** | Scene segmentation |
| **Shots** | Camera shot detection |
| **Labels** | Visual labels (objects, actions) |

## Use Cases

- Media asset management and searchability
- Content moderation
- Accessibility (captions, transcripts)
- Knowledge extraction from video libraries
- Compliance monitoring
- Video summarization

---

## Using Video Indexer

### Option 1: Portal (No Code)

1. Go to [videoindexer.ai](https://www.videoindexer.ai)
2. Upload a video
3. Wait for processing
4. Explore insights in the UI

### Option 2: Embed Widgets

Video Indexer provides embeddable widgets for your web applications:

```html
<!-- Player Widget -->
<iframe 
    src="https://www.videoindexer.ai/embed/player/<account-id>/<video-id>?accessToken=<token>"
    width="800" 
    height="450"
    allowfullscreen>
</iframe>

<!-- Insights Widget -->
<iframe 
    src="https://www.videoindexer.ai/embed/insights/<account-id>/<video-id>?accessToken=<token>"
    width="400" 
    height="600">
</iframe>
```

### Option 3: REST API

#### Get Access Token

```http
POST https://api.videoindexer.ai/Auth/<location>/Accounts/<accountId>/AccessToken
Ocp-Apim-Subscription-Key: <subscription-key>
```

#### Upload Video

```http
POST https://api.videoindexer.ai/<location>/Accounts/<accountId>/Videos?name=MyVideo&privacy=Private&accessToken=<token>
Content-Type: multipart/form-data

[video file]
```

#### Get Video Index (Insights)

```http
GET https://api.videoindexer.ai/<location>/Accounts/<accountId>/Videos/<videoId>/Index?accessToken=<token>
```

#### Response Example

```json
{
    "id": "abc123",
    "name": "MyVideo",
    "insights": {
        "transcript": [
            {
                "id": 1,
                "text": "Hello and welcome to our presentation.",
                "confidence": 0.95,
                "speakerId": 1,
                "instances": [{"start": "0:00:01", "end": "0:00:04"}]
            }
        ],
        "faces": [
            {
                "id": 1,
                "name": "Unknown #1",
                "confidence": 0.87,
                "thumbnailId": "thumb123"
            }
        ],
        "keywords": [
            {"name": "presentation", "confidence": 0.9}
        ],
        "topics": [
            {"name": "Technology", "confidence": 0.85}
        ]
    }
}
```

---

## SDK Example (Python)

Video Indexer doesn't have an official Python SDK, but you can use the REST API with `requests`:

```python
import requests
import time

account_id = "<your-account-id>"
location = "trial"  # or your Azure region
api_key = "<your-subscription-key>"

# 1. Get access token
token_url = f"https://api.videoindexer.ai/Auth/{location}/Accounts/{account_id}/AccessToken"
headers = {"Ocp-Apim-Subscription-Key": api_key}
token_response = requests.get(token_url, headers=headers)
access_token = token_response.json()

# 2. Upload video
upload_url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}/Videos"
params = {
    "name": "MyVideo",
    "privacy": "Private",
    "accessToken": access_token
}
with open("video.mp4", "rb") as video_file:
    files = {"file": video_file}
    upload_response = requests.post(upload_url, params=params, files=files)

video_id = upload_response.json()["id"]

# 3. Wait for indexing to complete
index_url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}/Videos/{video_id}/Index"
while True:
    index_response = requests.get(index_url, params={"accessToken": access_token})
    state = index_response.json()["state"]
    if state == "Processed":
        break
    print(f"Processing... State: {state}")
    time.sleep(30)

# 4. Get insights
insights = index_response.json()["videos"][0]["insights"]
print(f"Transcript: {insights['transcript']}")
print(f"Faces found: {len(insights.get('faces', []))}")
```

---

## Custom Person Model

When Video Indexer detects a face as "Unknown #1", you can train it:

### Via Portal
1. Go to Content Model Customization
2. Create a Person model
3. Add faces to known people
4. Re-index videos

### Via API

```http
POST https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels/{personModelId}/Persons?name=John%20Doe&accessToken={token}
```

---

## Cheatsheet

### Widget Types

| Widget | Purpose | Exam Keywords |
|--------|---------|---------------|
| **Player Widget** | Video playback | "Embed video player" |
| **Insights Widget** | Show extracted data | "Display transcript", "Show insights" |

### Privacy Levels

| Level | Visibility |
|-------|------------|
| Private | Only you |
| Public | Anyone with link |

### Indexing States

| State | Meaning |
|-------|---------|
| Uploading | Video being uploaded |
| Processing | AI analysis in progress |
| Processed | Ready to view |
| Failed | Error occurred |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Video Indexer = Computer Vision | ❌ Different service, different capabilities |
| Real-time video analysis | ❌ Video Indexer is for stored videos |
| Custom face = Custom Vision | ❌ Use Person Model in Video Indexer |

### Integration with Other Services

| Scenario | Integration |
|----------|-------------|
| Search video content | Export to Azure AI Search |
| Analyze specific frames | Export frames to Computer Vision |
| Custom NLP on transcript | Export transcript to AI Language |
| Store insights | Export to Cosmos DB or Blob Storage |

### Supported Languages

Video Indexer supports 50+ languages for:
- Transcription
- Translation
- OCR

Check the [official documentation](https://learn.microsoft.com/en-us/azure/azure-video-indexer/language-support) for the full list.

---

**Previous:** [Azure AI Search](06-azure-ai-search.md) | **Next:** [Content Safety](08-content-safety.md)

