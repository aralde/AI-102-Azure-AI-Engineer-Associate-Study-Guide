# Azure AI Search (Cognitive Search)

## Introduction

Azure AI Search is a cloud search service with built-in AI capabilities (cognitive skills) to enrich content during indexing. It's the backbone of RAG (Retrieval-Augmented Generation) solutions.

### Core Components

| Component | Purpose |
|-----------|---------|
| **Index** | The searchable data structure (like a database table) |
| **Indexer** | The worker that moves data from source to index |
| **Data Source** | Where your data lives (Blob, SQL, Cosmos DB) |
| **Skillset** | AI enrichment pipeline (OCR, translation, entities) |

## Use Cases

- Enterprise search portals
- E-commerce product search
- Document search with AI enrichment
- RAG (Retrieval-Augmented Generation) for GPT
- Knowledge mining from unstructured data

---

## SDK Example (Python)

```bash
pip install azure-search-documents
```

### Basic Search

```python
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

client = SearchClient(
    endpoint="https://<your-service>.search.windows.net",
    index_name="hotels-index",
    credential=AzureKeyCredential("<your-key>")
)

# Simple search
results = client.search(search_text="spa hotel")

for result in results:
    print(f"{result['hotelName']}: {result['description']}")
```

### Advanced Search with Filters

```python
results = client.search(
    search_text="spa hotel",
    filter="rating gt 4 and city eq 'Seattle'",  # OData filter
    select=["hotelName", "rating", "city"],       # Fields to return
    order_by=["rating desc"],                     # Sort order
    top=10,                                       # Limit results
    skip=0,                                       # Pagination offset
    include_total_count=True,                     # Get total matches
    facets=["category,count:5"],                  # Faceted navigation
    highlight_fields="description",               # Highlight matches
    search_mode="all"                             # AND instead of OR
)

print(f"Total results: {results.get_count()}")
```

### Semantic Search

```python
from azure.search.documents.models import QueryType

results = client.search(
    search_text="What hotels have a good spa?",
    query_type=QueryType.SEMANTIC,
    semantic_configuration_name="my-semantic-config",
    query_language="en-us",
    query_answer="extractive",   # Get direct answers
    query_caption="extractive"   # Get captions with highlights
)

for result in results:
    # Semantic answers
    if hasattr(result, '@search.answers'):
        for answer in result['@search.answers']:
            print(f"Answer: {answer['text']}")
    
    # Semantic captions
    if '@search.captions' in result:
        for caption in result['@search.captions']:
            print(f"Caption: {caption['text']}")
```

### Vector Search

```python
from azure.search.documents.models import VectorizedQuery

# You need to generate embeddings first (e.g., with Azure OpenAI)
query_vector = get_embedding("spa hotel")  # Your embedding function

results = client.search(
    search_text=None,  # Can combine with text search
    vector_queries=[
        VectorizedQuery(
            vector=query_vector,
            k_nearest_neighbors=10,
            fields="descriptionVector"
        )
    ]
)
```

### Index Management

```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex, SearchField, SearchFieldDataType
)

index_client = SearchIndexClient(endpoint, credential)

# Create an index
index = SearchIndex(
    name="hotels-index",
    fields=[
        SearchField(name="id", type=SearchFieldDataType.String, key=True),
        SearchField(name="hotelName", type=SearchFieldDataType.String, 
                   searchable=True, filterable=True, sortable=True),
        SearchField(name="description", type=SearchFieldDataType.String,
                   searchable=True),
        SearchField(name="rating", type=SearchFieldDataType.Double,
                   filterable=True, sortable=True, facetable=True)
    ]
)

index_client.create_or_update_index(index)
```

---

## REST API Example

### Search Query (POST)

```http
POST https://<service>.search.windows.net/indexes/hotels/docs/search?api-version=2023-11-01
Content-Type: application/json
api-key: <admin-key>

{
    "search": "spa hotel",
    "filter": "rating gt 4",
    "select": "hotelName,rating",
    "orderby": "rating desc",
    "top": 10,
    "count": true
}
```

**Note:** Complex search queries use POST, not GET!

### Create Index (PUT)

```http
PUT https://<service>.search.windows.net/indexes/hotels?api-version=2023-11-01
Content-Type: application/json
api-key: <admin-key>

{
    "name": "hotels",
    "fields": [
        {"name": "id", "type": "Edm.String", "key": true},
        {"name": "hotelName", "type": "Edm.String", "searchable": true}
    ]
}
```

---

## Skillsets (AI Enrichment)

### Common Built-in Skills

| Skill | Purpose | Input |
|-------|---------|-------|
| `OcrSkill` | Extract text from images | Images |
| `ImageAnalysisSkill` | Tags, captions from images | Images |
| `LanguageDetectionSkill` | Detect language | Text |
| `KeyPhraseExtractionSkill` | Extract key phrases | Text |
| `EntityRecognitionSkill` | Extract entities (NER) | Text |
| `SentimentSkill` | Analyze sentiment | Text |
| `TranslationSkill` | Translate text | Text |
| `SplitSkill` | Split text into chunks | Text |
| `MergeSkill` | Combine text | Multiple inputs |
| `WebApiSkill` | Call custom Azure Function | Any |

### Skillset Context

The `context` property defines the scope of the skill:

| Context | Meaning | Use Case |
|---------|---------|----------|
| `/document` | Once per document | Language detection for whole doc |
| `/document/pages/*` | Once per page | Sentiment per page |
| `/document/normalized_images/*` | Once per image | OCR each image |

### Custom Skill (WebApiSkill)

When built-in skills aren't enough, call your own Azure Function:

```json
{
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "uri": "https://my-function.azurewebsites.net/api/custom",
    "httpHeaders": {
        "x-functions-key": "<function-key>"
    },
    "context": "/document",
    "inputs": [
        {"name": "text", "source": "/document/content"}
    ],
    "outputs": [
        {"name": "customField", "targetName": "myCustomData"}
    ]
}
```

**Custom Skill Request/Response Format:**

```json
// Request to your function
{
    "values": [
        {"recordId": "1", "data": {"text": "Hello world"}}
    ]
}

// Response from your function (MUST include recordId!)
{
    "values": [
        {
            "recordId": "1",
            "data": {"customField": "processed"},
            "errors": [],
            "warnings": []
        }
    ]
}
```

---

## Cheatsheet

### The Sacred Order (Index Pipeline)

```
1. Data Source → 2. Skillset → 3. Index → 4. Indexer
```

For drag-and-drop questions:
1. **Data Source**: Where the data lives
2. **Skillset**: AI enrichment (optional)
3. **Index**: The schema
4. **Indexer**: The worker that runs it all

### Scaling: Replicas vs Partitions

| Problem | Solution | Mnemonic |
|---------|----------|----------|
| Queries are slow/throttling | Add **Replicas** | **R**eplicas = **R**ead |
| Index is too big | Add **Partitions** | **P**artitions = **P**arallel storage |
| Indexing is slow | Multiple **Indexers** + Virtual Folders | Parallel workers |

**Formula:** Search Units = Replicas × Partitions

### Field Mappings vs Output Field Mappings

| Type | Source | Use Case |
|------|--------|----------|
| `fieldMappings` | Raw source data | Map blob metadata to index field |
| `outputFieldMappings` | Skillset output | Map AI-enriched data to index field |

**Rule:** If the source starts with `/document/...` and it's from a skill, use `outputFieldMappings`.

### Synonym Maps

For equivalent terms (Car = Automobile = Vehicle):

```
# Solr format (bidirectional)
USA, United States, America

# Explicit mapping (unidirectional)
mbi => mobile banking interface
```

**Exam Keywords:** "Equivalent terms", "Acronyms", "Jargon"

### Scoring Profiles

Boost relevance based on fields or functions:

| Function | Use Case |
|----------|----------|
| `magnitude` | Boost by numeric value (price, rating) |
| `freshness` | Boost recent documents |
| `distance` | Boost nearby locations |
| `tag` | Boost matching tags |

### Search Modes

| Mode | Behavior |
|------|----------|
| `any` (default) | OR - matches any term |
| `all` | AND - must match all terms |

### Query Types

| Type | Use Case |
|------|----------|
| `simple` | Basic keyword search |
| `full` | Lucene syntax (wildcards, fuzzy) |
| `semantic` | AI-powered understanding |

### Lucene Query Syntax

| Syntax | Example | Meaning |
|--------|---------|---------|
| `*` | `micro*` | Wildcard (Microsoft, Microscope) |
| `?` | `t?st` | Single char (test, tost) |
| `~` | `blue~` | Fuzzy search (blues, glue) |
| `^` | `hotel^2 motel` | Boost term |
| `""` | `"spa hotel"` | Exact phrase |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Search queries use GET | ❌ Complex queries use POST |
| Replicas for indexing speed | ❌ Replicas are for query load |
| Synonym Map at index time | ❌ Applied at query time |
| Key Phrase for synonyms | ❌ Use Synonym Map |
| `fieldMappings` for skill output | ❌ Use `outputFieldMappings` |

### Knowledge Store (Projections)

Store enriched data for analytics:

| Projection Type | Output | Use Case |
|-----------------|--------|----------|
| Tables | Azure Table Storage | PowerBI analysis |
| Objects | Blob Storage (JSON) | Developer consumption |
| Files | Blob Storage (images) | Extracted images |

---

**Previous:** [Azure AI Language](05-azure-ai-language.md) | **Next:** [Azure Video Indexer](07-azure-video-indexer.md)

