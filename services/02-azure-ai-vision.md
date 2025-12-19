# Azure AI Vision & Custom Vision

## Introduction

**Azure AI Vision** (formerly Computer Vision) analyzes images using pre-trained models for general tasks like OCR, object detection, and image captioning.

**Azure Custom Vision** lets you train your own image classification or object detection models with your specific data.

### When to Use Which?

| Scenario | Service |
|----------|---------|
| Read text from any image (OCR) | AI Vision |
| Describe what's in a photo | AI Vision |
| Detect YOUR company's products | Custom Vision |
| Identify your company's logo | Custom Vision |
| General object detection | AI Vision |
| Specific defect detection in manufacturing | Custom Vision |

## Use Cases

- **OCR/Read**: Extract text from documents, signs, handwritten notes
- **Image Captioning**: Generate alt-text for accessibility
- **Object Detection**: Find and locate objects in images
- **Smart Cropping**: Automatically crop images for thumbnails
- **Custom Classification**: Is this a hot dog or not a hot dog?
- **Custom Detection**: Where is the defect on this product?

---

## Azure AI Vision (Standard)

### SDK Example (Python)

```bash
pip install azure-ai-vision-imageanalysis
```

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential

# Create client
client = ImageAnalysisClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

# Analyze image - YOU MUST SPECIFY what features you want!
result = client.analyze(
    image_url="https://example.com/image.jpg",
    visual_features=[
        VisualFeatures.CAPTION,         # Description of the image
        VisualFeatures.DENSE_CAPTIONS,  # Descriptions for regions
        VisualFeatures.TAGS,            # Content tags
        VisualFeatures.OBJECTS,         # Object detection with bounding boxes
        VisualFeatures.READ,            # OCR - extract text
        VisualFeatures.SMART_CROPS,     # Suggested crop regions
        VisualFeatures.PEOPLE           # Detect people
    ],
    smart_crops_aspect_ratios=[0.9, 1.33],  # Only if SMART_CROPS is requested
    gender_neutral_caption=True,            # "person" instead of "man/woman"
    language="en"
)

# Access results
print(f"Caption: {result.caption.text}")
print(f"Confidence: {result.caption.confidence}")

for tag in result.tags.list:
    print(f"Tag: {tag.name} ({tag.confidence:.2f})")
```

### REST API Example

```http
POST https://<endpoint>/computervision/imageanalysis:analyze?api-version=2024-02-01&features=caption,read,tags
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "url": "https://example.com/image.jpg"
}
```

**Important:** In REST API v4.0, features go in the **query string**, not the body!

---

## Azure Custom Vision

### SDK Example (Python)

```bash
pip install azure-cognitiveservices-vision-customvision
```

#### Training a Model

```python
from azure.cognitiveservices.vision.customvision.training import CustomVisionTrainingClient
from azure.cognitiveservices.vision.customvision.training.models import ImageFileCreateBatch, ImageFileCreateEntry
from msrest.authentication import ApiKeyCredentials

# Training client
trainer = CustomVisionTrainingClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credentials=ApiKeyCredentials(in_headers={"Training-key": "<training-key>"})
)

# Create project
project = trainer.create_project(
    name="MyClassifier",
    classification_type="Multiclass",  # One tag per image
    # classification_type="Multilabel"  # Multiple tags per image
    domain_id=None  # Use "Compact" domains for edge/mobile export
)

# Create tags
dog_tag = trainer.create_tag(project.id, "Dog")
cat_tag = trainer.create_tag(project.id, "Cat")

# Upload images with tags
with open("dog1.jpg", "rb") as image:
    trainer.create_images_from_data(
        project.id,
        image.read(),
        tag_ids=[dog_tag.id]
    )

# Train the model
iteration = trainer.train_project(project.id)

# Publish the iteration (makes it available for prediction)
trainer.publish_iteration(
    project.id,
    iteration.id,
    "myModelV1",  # Published name - use this in predictions!
    "<prediction-resource-id>"
)
```

#### Using a Trained Model (Prediction)

```python
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from msrest.authentication import ApiKeyCredentials

# Prediction client (different from training!)
predictor = CustomVisionPredictionClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credentials=ApiKeyCredentials(in_headers={"Prediction-key": "<prediction-key>"})
)

# Classification (returns tags)
with open("test.jpg", "rb") as image:
    results = predictor.classify_image(
        project_id="<project-id>",
        published_name="myModelV1",  # The name you used when publishing
        image_data=image.read()
    )

for prediction in results.predictions:
    print(f"{prediction.tag_name}: {prediction.probability:.2%}")

# Object Detection (returns bounding boxes)
results = predictor.detect_image(
    project_id="<project-id>",
    published_name="myModelV1",
    image_data=image_data
)

for prediction in results.predictions:
    print(f"{prediction.tag_name} at {prediction.bounding_box}")
```

---

## Cheatsheet

### Client Naming Pattern

| Service | Client Name |
|---------|-------------|
| AI Vision (Standard) | `ImageAnalysisClient` |
| Custom Vision Training | `CustomVisionTrainingClient` |
| Custom Vision Prediction | `CustomVisionPredictionClient` |

### VisualFeatures Enum (Must Know!)

| Feature | What It Returns |
|---------|-----------------|
| `CAPTION` | Single sentence describing the image |
| `DENSE_CAPTIONS` | Descriptions for specific regions |
| `TAGS` | Content tags (keywords) |
| `OBJECTS` | Objects with bounding boxes |
| `READ` | OCR - extracted text |
| `SMART_CROPS` | Suggested crop regions |
| `PEOPLE` | People detection with bounding boxes |

### Classification vs Detection

| Type | Method | Returns |
|------|--------|---------|
| Classification | `classify_image()` | Tags (what's in the image) |
| Object Detection | `detect_image()` | Bounding boxes + tags (where things are) |

### Precision vs Recall (Exam Favorite!)

| Metric | Formula | Goal | Use When |
|--------|---------|------|----------|
| **Precision** | TP / (TP + FP) | Avoid False Positives | Cost of wrong detection is high (spam filter) |
| **Recall** | TP / (TP + FN) | Avoid False Negatives | Can't miss anything (security, medical) |

**Memory Trick:**
- **P**recision = **P**unish False **P**ositives
- **R**ecall = **R**emember everything (don't miss)

### Probability Threshold Slider

| If you RAISE the threshold | Effect |
|---------------------------|--------|
| Precision | ⬆️ Goes UP |
| Recall | ⬇️ Goes DOWN |
| Behavior | More strict, fewer detections |

| If you LOWER the threshold | Effect |
|---------------------------|--------|
| Precision | ⬇️ Goes DOWN |
| Recall | ⬆️ Goes UP |
| Behavior | More liberal, more detections (with noise) |

### Domains: General vs Compact

| Domain Type | Use Case | Export Options |
|-------------|----------|----------------|
| General | Cloud-only, highest accuracy | None |
| Compact | Edge devices, mobile, offline | TensorFlow, CoreML, ONNX, Docker |

**Exam Keywords for Compact:**
- "Mobile app"
- "No internet connection"
- "Edge device"
- "Export to TensorFlow/CoreML"
- "Docker container"

### Project Types

| Type | Description | Example |
|------|-------------|---------|
| Multiclass | ONE tag per image | "Is this a dog OR a cat?" |
| Multilabel | MULTIPLE tags per image | "This image contains: dog, grass, outdoor" |
| Object Detection | Tags + coordinates | "Dog is at position [x,y,w,h]" |

### Training Requirements

| Requirement | Value |
|-------------|-------|
| Minimum images per tag | 5 |
| Recommended for production | 50+ per tag |
| Balance | Keep tags balanced (similar counts) |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| "Detect objects" = Classification | ❌ Detection returns bounding boxes |
| Using Training key for predictions | ❌ Use Prediction key |
| `classify_image` for object detection | ❌ Use `detect_image` |
| Features in REST body (v4.0) | ❌ Features go in query string |
| Not specifying VisualFeatures | ❌ You get nothing back! |

### Active Learning (`store=True`)

```python
results = predictor.classify_image(
    project_id="...",
    published_name="...",
    image_data=data,
    store=True  # Saves image for retraining!
)
```

This enables the **continuous improvement cycle** - review predictions and retrain.

---

**Previous:** [Azure AI Services](01-azure-ai-services.md) | **Next:** [Azure AI Speech](03-azure-ai-speech.md)

