# Azure AI Agent Service, Semantic Kernel & Autogen

## Introduction

Azure AI Agent Service (part of Azure AI Foundry) enables building AI agents that can use tools, maintain conversation state, and execute code. It's based on the OpenAI Assistants API architecture.

### Key Difference: Stateless vs Stateful

| Approach | State Management | Use Case |
|----------|------------------|----------|
| **Chat Completions** | Stateless - YOU manage history | Simple Q&A, one-off queries |
| **Agent Service** | Stateful - Azure manages history | Complex conversations, tool use |

## Core Concepts

### The Trinity of Agents

| Component | Purpose | Analogy |
|-----------|---------|---------|
| **Assistant** | The brain - model, instructions, tools | The employee |
| **Thread** | The memory - conversation history | The chat room |
| **Run** | The action - execute and respond | The task |

### Agent Tools (Superpowers)

| Tool | Use Case | Exam Keywords |
|------|----------|---------------|
| **Code Interpreter** | Execute Python, analyze files | "Sandboxed", "CSV analysis", "Math" |
| **File Search** | RAG over uploaded documents | "PDF", "Vector store", "Retrieval" |
| **Function Calling** | Call external APIs | "Database", "Weather API", "Custom logic" |

---

## SDK Example (Python)

```bash
pip install openai
```

### Create an Assistant

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key="<your-key>",
    api_version="2024-05-01-preview",
    azure_endpoint="https://<your-resource>.openai.azure.com/"
)

# Create an assistant with tools
assistant = client.beta.assistants.create(
    name="Math Tutor",
    instructions="You are a math tutor. Help students solve problems step by step.",
    model="gpt-4",  # Your deployment name
    tools=[
        {"type": "code_interpreter"},  # Can run Python code
        {"type": "file_search"}        # Can search uploaded files
    ]
)

print(f"Assistant ID: {assistant.id}")
```

### Create a Thread and Run

```python
# Create a thread (conversation)
thread = client.beta.threads.create()

# Add a message to the thread
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="What is the square root of 144?"
)

# Create a run (execute the assistant)
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id
)

# Poll for completion (IMPORTANT: Runs are async!)
import time

while run.status not in ["completed", "failed", "cancelled"]:
    time.sleep(1)
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    print(f"Status: {run.status}")

# Get the response
messages = client.beta.threads.messages.list(thread_id=thread.id)
for msg in messages.data:
    if msg.role == "assistant":
        print(f"Assistant: {msg.content[0].text.value}")
```

### Function Calling

```python
# Define a function tool
assistant = client.beta.assistants.create(
    name="Weather Bot",
    instructions="Help users with weather information.",
    model="gpt-4",
    tools=[{
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    }]
)

# When the run requires action...
if run.status == "requires_action":
    tool_calls = run.required_action.submit_tool_outputs.tool_calls
    
    tool_outputs = []
    for tool_call in tool_calls:
        if tool_call.function.name == "get_weather":
            # YOUR CODE executes the actual function
            result = get_weather_from_api(tool_call.function.arguments)
            tool_outputs.append({
                "tool_call_id": tool_call.id,
                "output": result
            })
    
    # Submit the results back to the assistant
    run = client.beta.threads.runs.submit_tool_outputs(
        thread_id=thread.id,
        run_id=run.id,
        tool_outputs=tool_outputs
    )
```

---

## Semantic Kernel

Semantic Kernel is Microsoft's SDK for building AI applications with plugins and planners.

### Installation

```bash
pip install semantic-kernel
```

### Basic Usage

```python
import semantic_kernel as sk
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

# Create kernel
kernel = sk.Kernel()

# Add Azure OpenAI service
kernel.add_service(
    AzureChatCompletion(
        deployment_name="gpt-4",
        endpoint="https://<your-resource>.openai.azure.com/",
        api_key="<your-key>"
    )
)

# Create a semantic function (prompt template)
prompt = """
Summarize the following text in {{$style}} style:

{{$input}}
"""

summarize = kernel.create_function_from_prompt(
    function_name="summarize",
    plugin_name="TextPlugin",
    prompt_template=prompt
)

# Invoke the function
result = await kernel.invoke(
    summarize,
    input="Long text to summarize...",
    style="professional"
)
print(result)
```

### Plugins (Native Functions)

```python
from semantic_kernel.functions import kernel_function

class MathPlugin:
    @kernel_function(
        name="add",
        description="Adds two numbers"
    )
    def add(self, a: int, b: int) -> int:
        return a + b

# Register the plugin
kernel.add_plugin(MathPlugin(), "Math")

# Now the AI can use this function
```

---

## Autogen

Microsoft Autogen enables multi-agent conversations where agents collaborate.

### Installation

```bash
pip install pyautogen
```

### Basic Multi-Agent Setup

```python
import autogen

# Configuration for Azure OpenAI
config_list = [{
    "model": "gpt-4",
    "api_key": "<your-key>",
    "base_url": "https://<your-resource>.openai.azure.com/",
    "api_type": "azure",
    "api_version": "2024-02-01"
}]

# Create agents
assistant = autogen.AssistantAgent(
    name="assistant",
    llm_config={"config_list": config_list}
)

user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",  # Automatic responses
    code_execution_config={"work_dir": "coding"}
)

# Start conversation
user_proxy.initiate_chat(
    assistant,
    message="Write a Python function to calculate fibonacci numbers."
)
```

---

## Cheatsheet

### Run Status Values

| Status | Meaning | Action |
|--------|---------|--------|
| `queued` | Waiting to start | Wait |
| `in_progress` | Processing | Wait |
| `requires_action` | Needs function output | Submit tool outputs |
| `completed` | Done | Get messages |
| `failed` | Error occurred | Handle error |
| `cancelled` | User cancelled | - |

### The Polling Pattern (Exam Favorite!)

```python
# This pattern appears in EVERY agent question
while run.status not in ["completed", "failed"]:
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    time.sleep(1)
```

### Tool Selection Guide

| If the exam says... | Use this tool |
|--------------------|---------------|
| "Execute Python code" | Code Interpreter |
| "Analyze CSV/Excel" | Code Interpreter |
| "Search documents" | File Search |
| "Call external API" | Function Calling |
| "Query database" | Function Calling |
| "Generate charts" | Code Interpreter |

### Function Calling Flow

```
1. User asks question
2. Assistant decides to call function
3. Run status = "requires_action"
4. YOUR CODE executes the function
5. Submit tool outputs
6. Run continues to "completed"
```

**Exam Question:** "Who executes the function - the model or your code?"
**Answer:** YOUR CODE executes it. The model only decides which function to call.

### ⚠️ Exam Traps

| Trap | Correct Answer |
|------|----------------|
| Agents are stateless | ❌ Agents are stateful (Thread stores history) |
| Run completes instantly | ❌ Runs are async, need polling |
| Model executes functions | ❌ Your code executes functions |
| `completed` means success | ✅ But check for `failed` too |

### Semantic Kernel vs Autogen

| Feature | Semantic Kernel | Autogen |
|---------|-----------------|---------|
| Focus | Single agent + plugins | Multi-agent collaboration |
| Complexity | Simpler | More complex |
| Use Case | Structured tasks | Creative problem-solving |

### Thread Persistence

```python
# Threads persist automatically
# Save the thread_id to continue later
thread_id = thread.id

# Later, retrieve and continue
messages = client.beta.threads.messages.create(
    thread_id=thread_id,  # Same thread
    role="user",
    content="Follow-up question"
)
```

---

**Previous:** [QnA Maker](09-qna-maker.md) | **Next:** [Azure AI Foundry](11-azure-ai-foundry.md)

