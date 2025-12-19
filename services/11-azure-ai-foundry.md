# Azure AI Foundry & Azure OpenAI

## Introduction

**Azure AI Foundry** (formerly Azure AI Studio) is the unified platform for building generative AI solutions. **Azure OpenAI** provides access to GPT models, DALL-E, and embeddings.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Deployment** | An instance of a model you can call |
| **Endpoint** | The URL to call your deployment |
| **Model** | The base AI model (GPT-4, GPT-3.5, etc.) |
| **Prompt** | The input you send to the model |
| **Completion** | The output from the model |

## Use Cases

- Chatbots and virtual assistants
- Content generation (articles, code, emails)
- Summarization
- Translation
- Code completion
- RAG (Retrieval-Augmented Generation)

---

## SDK Example (Python)

```bash
pip install openai
```

### Basic Chat Completion

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key="<your-key>",
    api_version="2024-02-01",
    azure_endpoint="https://<your-resource>.openai.azure.com/"
)

response = client.chat.completions.create(
    model="gpt-4-deployment",  # YOUR DEPLOYMENT NAME, not model name!
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"}
    ],
    temperature=0.7,
    max_tokens=800
)

print(response.choices[0].message.content)
```

### Multi-turn Conversation

```python
# You must maintain the conversation history yourself!
conversation = [
    {"role": "system", "content": "You are a helpful assistant."}
]

# Turn 1
conversation.append({"role": "user", "content": "My name is John."})
response = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=conversation
)
assistant_message = response.choices[0].message.content
conversation.append({"role": "assistant", "content": assistant_message})

# Turn 2 - The model remembers because we sent the history
conversation.append({"role": "user", "content": "What's my name?"})
response = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=conversation  # Contains full history
)
print(response.choices[0].message.content)  # "Your name is John"
```

### RAG with Azure AI Search (Grounding)

```python
response = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=[
        {"role": "system", "content": "Answer based only on the provided context."},
        {"role": "user", "content": "What is our return policy?"}
    ],
    extra_body={
        "data_sources": [{
            "type": "azure_search",
            "parameters": {
                "endpoint": "https://<search-service>.search.windows.net",
                "index_name": "company-docs",
                "authentication": {
                    "type": "api_key",
                    "key": "<search-key>"
                }
            }
        }]
    }
)

# Response includes citations
print(response.choices[0].message.content)
if hasattr(response.choices[0].message, 'context'):
    for citation in response.choices[0].message.context.citations:
        print(f"Source: {citation.filepath}")
```

### JSON Mode

```python
response = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=[
        {"role": "system", "content": "Extract data as JSON."},
        {"role": "user", "content": "John is 30 years old and lives in NYC."}
    ],
    response_format={"type": "json_object"}  # Forces valid JSON output
)

import json
data = json.loads(response.choices[0].message.content)
```

### Streaming

```python
stream = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=[{"role": "user", "content": "Write a story."}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Embeddings

```python
response = client.embeddings.create(
    model="text-embedding-ada-002-deployment",
    input="The quick brown fox"
)

embedding_vector = response.data[0].embedding
print(f"Vector dimension: {len(embedding_vector)}")  # 1536 for ada-002
```

---

## REST API Example

### Chat Completion

```http
POST https://<resource>.openai.azure.com/openai/deployments/<deployment>/chat/completions?api-version=2024-02-01
Content-Type: application/json
api-key: <your-key>

{
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    "temperature": 0.7,
    "max_tokens": 800
}
```

### With Data Sources (RAG)

```http
POST https://<resource>.openai.azure.com/openai/deployments/<deployment>/chat/completions?api-version=2024-02-01
Content-Type: application/json
api-key: <your-key>

{
    "messages": [
        {"role": "user", "content": "What's our vacation policy?"}
    ],
    "data_sources": [{
        "type": "azure_search",
        "parameters": {
            "endpoint": "https://<search>.search.windows.net",
            "index_name": "hr-docs",
            "authentication": {
                "type": "api_key",
                "key": "<search-key>"
            }
        }
    }]
}
```

---

## Cheatsheet

### Message Roles

| Role | Purpose | Example |
|------|---------|---------|
| `system` | Set behavior/personality | "You are a helpful assistant." |
| `user` | User's input | "What is 2+2?" |
| `assistant` | Model's previous responses | Used for conversation history |

### Temperature vs Top-P

| Parameter | Low Value | High Value |
|-----------|-----------|------------|
| `temperature` | Deterministic, focused | Creative, random |
| `top_p` | Fewer token choices | More token choices |

**Rule:** Use one or the other, not both!

| Use Case | Temperature |
|----------|-------------|
| Data extraction | 0.0 - 0.3 |
| Factual answers | 0.3 - 0.5 |
| General chat | 0.5 - 0.7 |
| Creative writing | 0.7 - 1.0 |

### Prompt Engineering Techniques

| Technique | Pattern | Use Case |
|-----------|---------|----------|
| **Zero-shot** | Just the question | Simple tasks |
| **Few-shot** | Examples before question | Format guidance |
| **Chain-of-Thought** | "Think step by step" | Complex reasoning |
| **Priming** | "You are a..." | Set persona/behavior |

### Grounding (RAG) Keywords

| If exam says... | Answer is... |
|-----------------|--------------|
| "Prevent hallucinations" | Use RAG/Grounding |
| "Use company data" | Azure Search data source |
| "Add citations" | RAG with `data_sources` |
| "Restrict to documents" | Set `in_scope=true` |

### The `extra_body` Pattern

```python
# For Azure-specific features not in standard OpenAI SDK
response = client.chat.completions.create(
    model="deployment-name",
    messages=[...],
    extra_body={
        "data_sources": [...]  # Azure Search integration
    }
)
```

### Deployment Name vs Model Name

| What You Call | Example |
|---------------|---------|
| In code: `model=` | Your deployment name: `"my-gpt4-deploy"` |
| NOT the base model | ❌ `"gpt-4"` |

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| `model="gpt-4"` in Azure | ❌ Use deployment name |
| Chat API is stateful | ❌ Stateless - you manage history |
| Use both temperature and top_p | ❌ Use one or the other |
| RAG is automatic | ❌ Must configure data_sources |

### Content Filtering

Azure OpenAI has built-in content filtering:

| Category | Filtered |
|----------|----------|
| Hate | ✅ |
| Violence | ✅ |
| Sexual | ✅ |
| Self-harm | ✅ |

Configure thresholds in Azure portal.

### Token Limits

| Model | Context Window |
|-------|----------------|
| GPT-3.5-turbo | 4K - 16K |
| GPT-4 | 8K - 128K |
| GPT-4o | 128K |

**Remember:** Input + Output must fit in context window!

### Function Calling

```python
response = client.chat.completions.create(
    model="gpt-4-deployment",
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
    tools=[{
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string"}
                },
                "required": ["city"]
            }
        }
    }],
    tool_choice="auto"
)

# Check if model wants to call a function
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    print(f"Function: {tool_call.function.name}")
    print(f"Arguments: {tool_call.function.arguments}")
```

### Responsible AI Principles

| Principle | Description |
|-----------|-------------|
| Fairness | No discrimination |
| Reliability | Consistent behavior |
| Privacy | Protect user data |
| Inclusiveness | Accessible to all |
| Transparency | Explainable decisions |
| Accountability | Human oversight |

---

**Previous:** [AI Agents](10-ai-agents.md) | **Next:** [Master Cheatsheet](../cheatsheets/master-cheatsheet.md)

