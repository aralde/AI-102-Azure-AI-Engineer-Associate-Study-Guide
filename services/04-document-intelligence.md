# Azure AI Document Intelligence (Form Recognizer)

## Introduction

Azure AI Document Intelligence (formerly Form Recognizer) extracts text, structure, and key-value pairs from documents like PDFs, images, invoices, and receipts.

### Key Difference from Other Services

| Input Type | Service to Use |
|------------|----------------|
| Plain text (string) | Azure AI Language |
| PDF, Image, Scanned document | **Document Intelligence** |

**Exam Trap:** If the question mentions "PDF" or "scanned document", the answer is Document Intelligence, even if you want to do sentiment analysis later!

## Use Cases

- Invoice processing (Accounts Payable automation)
- Receipt scanning (Expense reports)
- ID verification (KYC - Know Your Customer)
- Contract analysis
- Form data extraction
- Table extraction from documents

---

## SDK Example (Python)

```bash
pip install azure-ai-documentintelligence
# OR for older SDK:
pip install azure-ai-formrecognizer
```

### Basic Document Analysis

```python
from azure.ai.documentintelligence import DocumentIntelligenceClient
from azure.core.credentials import AzureKeyCredential

client = DocumentIntelligenceClient(
    endpoint="https://<your-resource>.cognitiveservices.azure.com/",
    credential=AzureKeyCredential("<your-key>")
)

# Analyze a document (returns a poller - async operation!)
with open("invoice.pdf", "rb") as f:
    poller = client.begin_analyze_document(
        model_id="prebuilt-invoice",  # The model to use
        body=f
    )

# Wait for the result
result = poller.result()

# Access extracted data
for document in result.documents:
    print(f"Vendor: {document.fields.get('VendorName').value}")
    print(f"Total: {document.fields.get('InvoiceTotal').value}")
```

### Analyze from URL

```python
poller = client.begin_analyze_document(
    model_id="prebuilt-layout",
    body={"urlSource": "https://example.com/document.pdf"}
)
```

### Extract Tables (Layout Model)

```python
poller = client.begin_analyze_document(
    model_id="prebuilt-layout",
    body=document_bytes
)

result = poller.result()

# Access tables
for table in result.tables:
    print(f"Table with {table.row_count} rows and {table.column_count} columns")
    for cell in table.cells:
        print(f"  Row {cell.row_index}, Col {cell.column_index}: {cell.content}")
```

### Analyze with Options

```python
poller = client.begin_analyze_document(
    model_id="prebuilt-layout",
    body=document_bytes,
    pages="1-3",                    # Only analyze pages 1-3
    locale="en-US",                 # Hint for better recognition
    features=[                      # Optional add-ons
        "ocrHighResolution",        # Better OCR quality
        "formulas",                 # Detect math formulas
        "barcodes",                 # Detect barcodes
        "languages"                 # Detect languages
    ]
)
```

---

## REST API Example

### Start Analysis (POST)

```http
POST https://<endpoint>/documentintelligence/documentModels/prebuilt-invoice:analyze?api-version=2024-02-29-preview
Content-Type: application/json
Ocp-Apim-Subscription-Key: <key>

{
    "urlSource": "https://example.com/invoice.pdf"
}
```

### Response: 202 Accepted

```http
HTTP/1.1 202 Accepted
Operation-Location: https://<endpoint>/documentintelligence/documentModels/prebuilt-invoice/analyzeResults/<operation-id>?api-version=2024-02-29-preview
```

### Check Status (GET) - Polling Pattern

```http
GET https://<endpoint>/documentintelligence/documentModels/prebuilt-invoice/analyzeResults/<operation-id>?api-version=2024-02-29-preview
Ocp-Apim-Subscription-Key: <key>
```

### Response When Complete

```json
{
    "status": "succeeded",
    "analyzeResult": {
        "documents": [{
            "fields": {
                "VendorName": {"value": "Contoso Ltd"},
                "InvoiceTotal": {"value": 1500.00}
            }
        }]
    }
}
```

---

## Prebuilt Models Cheatsheet

| Model ID | Use Case | Exam Keywords |
|----------|----------|---------------|
| `prebuilt-read` | Simple OCR, just text | "Extract text", "No structure needed" |
| `prebuilt-layout` | Text + Tables + Checkboxes | "Tables", "Selection marks", "Preserve structure" |
| `prebuilt-invoice` | Invoices | "Accounts Payable", "Vendor invoices" |
| `prebuilt-receipt` | Receipts/Tickets | "Expense reports", "Thermal paper" |
| `prebuilt-idDocument` | Passports, Driver's licenses | "KYC", "Identity verification" |
| `prebuilt-businessCard` | Business cards | "CRM", "Contact extraction" |
| `prebuilt-tax.us.w2` | US W-2 tax forms | "Tax filing", "Employee income" |
| `prebuilt-healthInsuranceCard.us` | US health insurance cards | "Healthcare", "Patient processing" |

### Read vs Layout (Critical Distinction!)

| Feature | `prebuilt-read` | `prebuilt-layout` |
|---------|-----------------|-------------------|
| Text extraction | ✅ | ✅ |
| Tables | ❌ | ✅ |
| Checkboxes/Selection marks | ❌ | ✅ |
| Key-value pairs | ❌ | ✅ |
| Document structure | ❌ | ✅ |

**Exam Tip:** If the question mentions "tables" or "checkboxes", the answer is `prebuilt-layout`, NOT `prebuilt-read`!

### Invoice vs Receipt

| Document | Model | Typical Scenario |
|----------|-------|------------------|
| Invoice | `prebuilt-invoice` | B2B, PDF, formal documents |
| Receipt | `prebuilt-receipt` | B2C, thermal paper, expense tracking |

---

## Cheatsheet

### Async Pattern (Long Running Operations)

```python
# All Document Intelligence operations are async!
# Method starts with "begin_" and returns a poller

poller = client.begin_analyze_document(...)  # Starts the job

# Option 1: Wait for completion
result = poller.result()  # Blocks until done

# Option 2: Check status manually
while not poller.done():
    print(f"Status: {poller.status()}")
    time.sleep(1)
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 202 Accepted | Job started, check `Operation-Location` header |
| 200 OK | Job complete (when polling) |

### Polling Pattern in Code

```python
import time

poller = client.begin_analyze_document(model_id="prebuilt-invoice", body=doc)

# The polling loop
while True:
    status = poller.status()
    if status == "succeeded":
        result = poller.result()
        break
    elif status == "failed":
        raise Exception("Analysis failed")
    time.sleep(1)
```

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Using AI Language for PDFs | ❌ Use Document Intelligence |
| `prebuilt-read` for tables | ❌ Use `prebuilt-layout` |
| Expecting instant response | ❌ It's async (202 + polling) |
| Not checking `Operation-Location` | ❌ That header has the polling URL |

### Custom Models

When prebuilt models don't fit your needs:

1. **Compose models**: Combine multiple custom models
2. **Train with labels**: Manually label your documents
3. **Train without labels**: Let the service figure it out

```python
# Training a custom model
poller = client.begin_build_document_model(
    build_mode="template",  # or "neural"
    blob_container_url="https://storage.../training-data",
    model_id="my-custom-model"
)
```

### Document Intelligence vs Computer Vision OCR

| Feature | Document Intelligence | Computer Vision |
|---------|----------------------|-----------------|
| Focus | Documents (PDFs, forms) | General images |
| Tables | ✅ Native support | ❌ |
| Key-value pairs | ✅ | ❌ |
| Prebuilt models | ✅ Invoice, Receipt, etc. | ❌ |
| Simple image OCR | ✅ | ✅ |

---

**Previous:** [Azure AI Speech](03-azure-ai-speech.md) | **Next:** [Azure AI Language](05-azure-ai-language.md)

