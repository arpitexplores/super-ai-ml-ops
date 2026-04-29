## Source: references/skills/llm-ops/SKILL.md

---
name: llm-ops
description: 'LLM Operations -- RAG, embeddings, vector databases, fine-tuning, prompt engineering avancado, custos de LLM, evals de qualidade e arquiteturas de IA para producao. Ativar para: implementar
  RAG,...'
risk: safe
source: community
date_added: '2026-03-06'
author: renat
tags:
- llm
- rag
- embeddings
- vector-db
- fine-tuning
tools:
- agent-compatible
- agent-compatible
- agent-compatible
- agent-compatible
- agent-compatible
---

# LLM-OPS -- IA de Producao

## Overview

LLM Operations -- RAG, embeddings, vector databases, fine-tuning, prompt engineering avancado, custos de LLM, evals de qualidade e arquiteturas de IA para producao. Ativar para: implementar RAG, criar pipeline de embeddings, Pinecone/Chroma/pgvector, fine-tuning, prompt engineering, reducao de custos de LLM, evals, cache semantico, streaming, agents.

## When to Use This Skill

- When you need specialized assistance with this domain

## Do Not Use This Skill When

- The task is unrelated to llm ops
- A simpler, more specific tool can handle the request
- The user needs general-purpose assistance without domain expertise

## How It Works

> A diferenca entre um prototipo de IA e um produto de IA e operabilidade.
> LLM-Ops e a engenharia que torna IA confiavel, escalavel e economica.

---

## Arquitetura Rag Completa

[Documentos] -> [Chunking] -> [Embeddings] -> [Vector DB]
                                                      |
    [Query] -> [Embed query] -> [Semantic Search] -> [Top K chunks]
                                                          |
                                           [LLM + Context] -> [Resposta]

## Pipeline De Indexacao

from llm_provider import LLMClient
    import chromadb

    client = LLMClient()
    chroma = chromadb.PersistentClient(path="./chroma_db")

    def chunk_text(text, chunk_size=500, overlap=50):
        words = text.split()
        chunks = []
        for i in range(0, len(words), chunk_size - overlap):
            chunk = " ".join(words[i:i + chunk_size])
            if chunk: chunks.append(chunk)
        return chunks

    def index_document(doc_id, content_text, metadata=None):
        chunks = chunk_text(content_text)
        ids = [f"{doc_id}_chunk_{i}" for i in range(len(chunks))]
        collection.upsert(ids=ids, documents=chunks)
        return len(chunks)

## Pipeline De Query Com Rag

def rag_query(query, top_k=5, system=None):
        results = collection.query(
            query_texts=[query], n_results=top_k,
            include=["documents", "metadatas", "distances"])
        context_parts = []
        for doc, meta, dist in zip(results["documents"][0],
                                    results["metadatas"][0],
                                    results["distances"][0]):
            if dist < 1.5:
                src = meta.get("source", "doc")
                context_parts.append(f"[Fonte: {src}]
{doc}")
        context = "

---

".join(context_parts)
        response = client.messages.create(
            model="example-premium-model", max_tokens=1024,
            system=system or "Responda baseado no contexto.",
            messages=[{"role": "user", "content": f"Contexto:
{context}

{query}"}])
        return response.content[0].text

---

## Escolha Do Vector Db

| DB | Melhor Para | Hosting | Custo |
|----|------------|---------|-------|
| Chroma | Desenvolvimento, local | Self-hosted | Gratis |
| pgvector | Ja usa PostgreSQL | Self/Cloud | Gratis |
| Pinecone | Producao gerenciada | Cloud | USD 70+/mes |
| Weaviate | Multi-modal | Self/Cloud | Gratis+ |
| Qdrant | Alta performance | Self/Cloud | Gratis+ |

## Pgvector

CREATE EXTENSION IF NOT EXISTS vector;
    CREATE TABLE knowledge_embeddings (
        id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
        content TEXT NOT NULL,
        embedding vector(1536),
        metadata JSONB,
        created_at TIMESTAMPTZ DEFAULT NOW()
    );
    CREATE INDEX ON knowledge_embeddings
    USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
    SELECT content, 1 - (embedding <=> QUERY_VECTOR) AS similarity
    FROM knowledge_embeddings ORDER BY similarity DESC LIMIT 5;

---

## Estrutura De Prompt De Elite

Componentes do system prompt Auri:

- Identidade: Nome (Auri), Tom (Natural, caloroso, direto), Plataforma (Amazon Alexa)
- Regras: Maximo 3 paragrafos curtos, sem markdown, linguagem conversacional
- Capacidades: analise de negocios, conselho baseado em dados, criatividade
- Limitacoes: sem internet tempo real, sem transacoes financeiras
- Personalizacao: {user_name}, {user_preferences}, {relevant_history}

## Chain-Of-Thought

def cot_analysis(problem: str) -> str:
        steps = [
            "1. O que exatamente esta sendo pedido?",
            "2. Que informacoes sao criticas para resolver?",
            "3. Quais abordagens possiveis existem?",
            "4. Qual abordagem e melhor e por que?",
            "5. Quais riscos ou limitacoes existem?",
        ]
        prompt = f"Analise passo a passo:

PROBLEMA: {problem}

"
        prompt += "
".join(steps) + "

Resposta final (concisa, para voz):"
        return call_claude(prompt)

---

## Cache Semantico

class SemanticCache:
        def __init__(self, similarity_threshold=0.95):
            self.threshold = similarity_threshold
            self.cache = {}

        def get_cached(self, query, embedding):
            for cached_emb, (response, _) in self.cache.items():
                if cosine_similarity(embedding, cached_emb) >= self.threshold:
                    return response
            return None

        def set_cache(self, query, embedding, response):
            self.cache[tuple(embedding)] = (response, query)

## Estimativa De Custos AI assistant

PRICING = {
        "example-premium-model": {"input": 15.00, "output": 75.00},
        "example-balanced-model": {"input": 3.00, "output": 15.00},
        "example-fast-model": {"input": 0.80, "output": 4.00},
    }

    def estimate_monthly_cost(model, avg_input, avg_output, req_per_day):
        p = PRICING[model]
        daily = (avg_input + avg_output) * req_per_day / 1e6
        monthly = daily * p["input"] * 30
        return {"model": model, "monthly_cost": "USD %.2f" % monthly}

---

## Framework De Avaliacao

from llm_provider import LLMClient
    client = LLMClient()

    def evaluate_response(question, expected, actual, criteria):
        criteria_text = "
".join(f"- {c}" for c in criteria)
        eval_prompt = (
            f"Avalie a resposta do assistente de IA.

"
            f"PERGUNTA: {question}
RESPOSTA ESPERADA: {expected}
"
            f"RESPOSTA ATUAL: {actual}

Criterios:
{criteria_text}

"
            "Nota 0-10 e justificativa para cada criterio. Formato JSON."
        )
        response = client.messages.create(
            model="example-fast-model", max_tokens=1024,
            messages=[{"role": "user", "content": eval_prompt}]
        )
        import json
        return json.loads(response.content[0].text)

    AURI_EVALS = [
        {
            "question": "Quais sao os principais riscos de abrir startup agora?",
            "criteria": ["precisao_factual", "relevancia", "clareza_para_voz"]
        },
    ]

---

## 6. Comandos

| Comando | Acao |
|---------|------|
| /rag-setup | Configura pipeline RAG completo |
| /embed-docs | Indexa documentos no vector DB |
| /prompt-optimize | Otimiza prompt para qualidade e custo |
| /cost-estimate | Estima custo mensal do LLM |
| /eval-run | Roda suite de evals de qualidade |
| /cache-setup | Configura cache semantico |
| /model-select | Escolhe modelo ideal para o caso de uso |

## Best Practices

- Provide clear, specific context about your project and requirements
- Review all suggestions before applying them to production code
- Combine with other complementary skills for comprehensive analysis

## Common Pitfalls

- Using this skill for tasks outside its domain expertise
- Applying recommendations without understanding your specific context
- Not providing enough project context for accurate analysis

---

## Merged Reference (legacy variant)

---
name: prompt-caching
description: "Caching strategies for LLM prompts including provider-native prompt caching, response caching, and CAG (Cache Augmented Generation) Use when: prompt caching, cache prompt, response cache, cag, cache augm..."
risk: unknown
source: "vibeship-spawner-skills (Apache 2.0)"
date_added: "2026-02-27"
---

# Prompt Caching

You're a caching specialist who has reduced LLM costs by 90% through strategic caching.
You've implemented systems that cache at multiple levels: prompt prefixes, full responses,
and semantic similarity matches.

You understand that LLM caching is different from traditional caching—prompts have
prefixes that can be cached, responses vary with temperature, and semantic similarity
often matters more than exact match.

Your core principles:
1. Cache at the right level—prefix, response, or both
2. K

## Capabilities

- prompt-cache
- response-cache
- kv-cache
- cag-patterns
- cache-invalidation

## Patterns

### Provider-Native Prompt Caching

Use AI assistant's native prompt caching for repeated prefixes

### Response Caching

Cache full LLM responses for identical or similar queries

### Cache Augmented Generation (CAG)

Pre-cache documents in prompt instead of RAG retrieval

## Anti-Patterns

### ❌ Caching with High Temperature

### ❌ No Cache Invalidation

### ❌ Caching Everything

## ⚠️ Sharp Edges

| Issue | Severity | Solution |
|-------|----------|----------|
| Cache miss causes latency spike with additional overhead | high | // Optimize for cache misses, not just hits |
| Cached responses become incorrect over time | high | // Implement proper cache invalidation |
| Prompt caching doesn't work due to prefix changes | medium | // Structure prompts for optimal caching |

## Related Skills

Works well with: `context-window-management`, `rag-implementation`, `conversation-memory`

## When to Use
This skill is applicable to execute the workflow or actions described in the overview.

---

## Merged Reference (legacy variant)

---
name: langfuse
description: "Expert in Langfuse - the open-source LLM observability platform. Covers tracing, prompt management, evaluation, datasets, and integration with LangChain, LlamaIndex, and OpenAI. Essential for debug..."
risk: unknown
source: "vibeship-spawner-skills (Apache 2.0)"
date_added: "2026-02-27"
---

# Langfuse

**Role**: LLM Observability Architect

You are an expert in LLM observability and evaluation. You think in terms of
traces, spans, and metrics. You know that LLM applications need monitoring
just like traditional software - but with different dimensions (cost, quality,
latency). You use data to drive prompt improvements and catch regressions.

## Capabilities

- LLM tracing and observability
- Prompt management and versioning
- Evaluation and scoring
- Dataset management
- Cost tracking
- Performance monitoring
- A/B testing prompts

## Requirements

- Python or TypeScript/JavaScript
- Langfuse account (cloud or self-hosted)
- LLM API keys

## Patterns

### Basic Tracing Setup

Instrument LLM calls with Langfuse

**When to use**: Any LLM application

```python
from langfuse import Langfuse

# Initialize client
langfuse = Langfuse(
    public_key="pk-...",
    secret_key="sk-...",
    host="https://cloud.langfuse.com"  # or self-hosted URL
)

# Create a trace for a user request
trace = langfuse.trace(
    name="chat-completion",
    user_id="user-123",
    session_id="session-456",  # Groups related traces
    metadata={"feature": "customer-support"},
    tags=["production", "v2"]
)

# Log a generation (LLM call)
generation = trace.generation(
    name="gpt-4o-response",
    model="gpt-4o",
    model_parameters={"temperature": 0.7},
    input={"messages": [{"role": "user", "content": "Hello"}]},
    metadata={"attempt": 1}
)

# Make actual LLM call
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)

# Complete the generation with output
generation.end(
    output=response.choices[0].message.content,
    usage={
        "input": response.usage.prompt_tokens,
        "output": response.usage.completion_tokens
    }
)

# Score the trace
trace.score(
    name="user-feedback",
    value=1,  # 1 = positive, 0 = negative
    comment="User clicked helpful"
)

# Flush before exit (important in serverless)
langfuse.flush()
```

### OpenAI Integration

Automatic tracing with OpenAI SDK

**When to use**: OpenAI-based applications

```python
from langfuse.openai import openai

# Drop-in replacement for OpenAI client
# All calls automatically traced

response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    # Langfuse-specific parameters
    name="greeting",  # Trace name
    session_id="session-123",
    user_id="user-456",
    tags=["test"],
    metadata={"feature": "chat"}
)

# Works with streaming
stream = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True,
    name="story-generation"
)

for chunk in stream:
    print(chunk.choices[0].delta.content, end="")

# Works with async
import asyncio
from langfuse.openai import AsyncOpenAI

async_client = AsyncOpenAI()

async def main():
    response = await async_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello"}],
        name="async-greeting"
    )
```

### LangChain Integration

Trace LangChain applications

**When to use**: LangChain-based applications

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langfuse.callback import CallbackHandler

# Create Langfuse callback handler
langfuse_handler = CallbackHandler(
    public_key="pk-...",
    secret_key="sk-...",
    host="https://cloud.langfuse.com",
    session_id="session-123",
    user_id="user-456"
)

# Use with any LangChain component
llm = ChatOpenAI(model="gpt-4o")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])

chain = prompt | llm

# Pass handler to invoke
response = chain.invoke(
    {"input": "Hello"},
    config={"callbacks": [langfuse_handler]}
)

# Or set as default
import langchain
langchain.callbacks.manager.set_handler(langfuse_handler)

# Then all calls are traced
response = chain.invoke({"input": "Hello"})

# Works with agents, retrievers, etc.
from langchain.agents import create_openai_tools_agent

agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

result = agent_executor.invoke(
    {"input": "What's the weather?"},
    config={"callbacks": [langfuse_handler]}
)
```

## Anti-Patterns

### ❌ Not Flushing in Serverless

**Why bad**: Traces are batched.
Serverless may exit before flush.
Data is lost.

**Instead**: Always call langfuse.flush() at end.
Use context managers where available.
Consider sync mode for critical traces.

### ❌ Tracing Everything

**Why bad**: Noisy traces.
Performance overhead.
Hard to find important info.

**Instead**: Focus on: LLM calls, key logic, user actions.
Group related operations.
Use meaningful span names.

### ❌ No User/Session IDs

**Why bad**: Can't debug specific users.
Can't track sessions.
Analytics limited.

**Instead**: Always pass user_id and session_id.
Use consistent identifiers.
Add relevant metadata.

## Limitations

- Self-hosted requires infrastructure
- High-volume may need optimization
- Real-time dashboard has latency
- Evaluation requires setup

## Related Skills

Works well with: `langgraph`, `crewai`, `structured-output`, `autonomous-agents`

## When to Use
This skill is applicable to execute the workflow or actions described in the overview.

---

## Merged Reference (legacy variant)

---
name: manifest
description: "Install and configure the Manifest observability plugin for your agents. Use when setting up telemetry, configuring API keys, or troubleshooting the plugin."
risk: unknown
source: community
date_added: "2026-02-27"
---

# Manifest Setup

Follow these steps **in order**. Do not skip ahead.

## Use this skill when

- User wants to set up observability or telemetry for their agent
- User wants to connect their agent to Manifest for monitoring
- User needs to configure a Manifest API key or custom endpoint
- User is troubleshooting Manifest plugin connection issues
- User wants to verify the Manifest plugin is running

## Do not use this skill when

- User needs general observability design (use `observability-engineer` instead)
- User wants to build custom dashboards or alerting rules
- User is not using the Manifest platform

## Instructions

### Step 1 — Stop the gateway

Stop the gateway first to avoid hot-reload issues during configuration.

```bash
agent-cli gateway stop
```

### Step 2 — Install the plugin

```bash
agent-cli plugins install manifest
```

If it fails, check that the CLI is installed and available in the PATH.

### Step 3 — Get an API key

Ask the user:

> To connect your agent, you need a Manifest API key. Here's how to get one:
>
> 1. Go to **https://app.manifest.build** and create an account (or sign in)
> 2. Once logged in, click **"Connect Agent"** to create a new agent
> 3. Copy the API key that starts with `mnfst_`
> 4. Paste it here

Wait for a key starting with `mnfst_`. If the key doesn't match, tell the user the format looks incorrect and ask them to try again.

### Step 4 — Configure the plugin

```bash
agent-cli config set plugins.entries.manifest.config.apiKey "USER_API_KEY"
```

Replace `USER_API_KEY` with the actual key the user provided.

Ask the user if they have a custom endpoint. If not, the default (`https://app.manifest.build/api/v1/otlp`) is used automatically. If they do:

```bash
agent-cli config set plugins.entries.manifest.config.endpoint "USER_ENDPOINT"
```

### Step 5 — Start the gateway

```bash
agent-cli gateway install
```

### Step 6 — Verify

Wait 3 seconds for the gateway to fully start, then check the logs:

```bash
grep "manifest" tool-specific config pathslogs/gateway.log | tail -5
```

Look for:

```
[manifest] Observability pipeline active
```

If it appears, tell the user setup is complete. If not, check the error messages and troubleshoot.

## Safety

- Never log or echo the API key in plain text after configuration
- Verify the key format (`mnfst_` prefix) before writing to config

## Troubleshooting

| Error | Fix |
|-------|-----|
| Missing apiKey | Re-run step 4 |
| Invalid apiKey format | The key must start with `mnfst_` |
| Connection refused | The endpoint is unreachable. Check the URL or ask if they self-host |
| Duplicate OTel registration | Disable the conflicting built-in plugin: `agent-cli plugins disable diagnostics-otel` |

## Examples

### Example 1: Basic setup

```
Use @manifest to set up observability for my agent.
```

### Example 2: Custom endpoint

```
Use @manifest to connect my agent to my self-hosted Manifest instance at https://manifest.internal.company.com/api/v1/otlp
```

## Best Practices

- Always stop the gateway before making configuration changes
- The default endpoint works for most users — only change it if self-hosting
- API keys always start with `mnfst_` — any other format is invalid
- Check gateway logs first when debugging any plugin issue

## Source: references/skills/llm-ops/references/legacy/langfuse/SKILL.md

---
name: langfuse
description: "Expert in Langfuse - the open-source LLM observability platform. Covers tracing, prompt management, evaluation, datasets, and integration with LangChain, LlamaIndex, and OpenAI. Essential for debug..."
risk: unknown
source: "vibeship-spawner-skills (Apache 2.0)"
date_added: "2026-02-27"
---

# Langfuse

**Role**: LLM Observability Architect

You are an expert in LLM observability and evaluation. You think in terms of
traces, spans, and metrics. You know that LLM applications need monitoring
just like traditional software - but with different dimensions (cost, quality,
latency). You use data to drive prompt improvements and catch regressions.

## Capabilities

- LLM tracing and observability
- Prompt management and versioning
- Evaluation and scoring
- Dataset management
- Cost tracking
- Performance monitoring
- A/B testing prompts

## Requirements

- Python or TypeScript/JavaScript
- Langfuse account (cloud or self-hosted)
- LLM API keys

## Patterns

### Basic Tracing Setup

Instrument LLM calls with Langfuse

**When to use**: Any LLM application

```python
from langfuse import Langfuse

# Initialize client
langfuse = Langfuse(
    public_key="pk-...",
    secret_key="sk-...",
    host="https://cloud.langfuse.com"  # or self-hosted URL
)

# Create a trace for a user request
trace = langfuse.trace(
    name="chat-completion",
    user_id="user-123",
    session_id="session-456",  # Groups related traces
    metadata={"feature": "customer-support"},
    tags=["production", "v2"]
)

# Log a generation (LLM call)
generation = trace.generation(
    name="gpt-4o-response",
    model="gpt-4o",
    model_parameters={"temperature": 0.7},
    input={"messages": [{"role": "user", "content": "Hello"}]},
    metadata={"attempt": 1}
)

# Make actual LLM call
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)

# Complete the generation with output
generation.end(
    output=response.choices[0].message.content,
    usage={
        "input": response.usage.prompt_tokens,
        "output": response.usage.completion_tokens
    }
)

# Score the trace
trace.score(
    name="user-feedback",
    value=1,  # 1 = positive, 0 = negative
    comment="User clicked helpful"
)

# Flush before exit (important in serverless)
langfuse.flush()
```

### OpenAI Integration

Automatic tracing with OpenAI SDK

**When to use**: OpenAI-based applications

```python
from langfuse.openai import openai

# Drop-in replacement for OpenAI client
# All calls automatically traced

response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    # Langfuse-specific parameters
    name="greeting",  # Trace name
    session_id="session-123",
    user_id="user-456",
    tags=["test"],
    metadata={"feature": "chat"}
)

# Works with streaming
stream = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True,
    name="story-generation"
)

for chunk in stream:
    print(chunk.choices[0].delta.content, end="")

# Works with async
import asyncio
from langfuse.openai import AsyncOpenAI

async_client = AsyncOpenAI()

async def main():
    response = await async_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "Hello"}],
        name="async-greeting"
    )
```

### LangChain Integration

Trace LangChain applications

**When to use**: LangChain-based applications

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langfuse.callback import CallbackHandler

# Create Langfuse callback handler
langfuse_handler = CallbackHandler(
    public_key="pk-...",
    secret_key="sk-...",
    host="https://cloud.langfuse.com",
    session_id="session-123",
    user_id="user-456"
)

# Use with any LangChain component
llm = ChatOpenAI(model="gpt-4o")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])

chain = prompt | llm

# Pass handler to invoke
response = chain.invoke(
    {"input": "Hello"},
    config={"callbacks": [langfuse_handler]}
)

# Or set as default
import langchain
langchain.callbacks.manager.set_handler(langfuse_handler)

# Then all calls are traced
response = chain.invoke({"input": "Hello"})

# Works with agents, retrievers, etc.
from langchain.agents import create_openai_tools_agent

agent = create_openai_tools_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

result = agent_executor.invoke(
    {"input": "What's the weather?"},
    config={"callbacks": [langfuse_handler]}
)
```

## Anti-Patterns

### ❌ Not Flushing in Serverless

**Why bad**: Traces are batched.
Serverless may exit before flush.
Data is lost.

**Instead**: Always call langfuse.flush() at end.
Use context managers where available.
Consider sync mode for critical traces.

### ❌ Tracing Everything

**Why bad**: Noisy traces.
Performance overhead.
Hard to find important info.

**Instead**: Focus on: LLM calls, key logic, user actions.
Group related operations.
Use meaningful span names.

### ❌ No User/Session IDs

**Why bad**: Can't debug specific users.
Can't track sessions.
Analytics limited.

**Instead**: Always pass user_id and session_id.
Use consistent identifiers.
Add relevant metadata.

## Limitations

- Self-hosted requires infrastructure
- High-volume may need optimization
- Real-time dashboard has latency
- Evaluation requires setup

## Related Skills

Works well with: `langgraph`, `crewai`, `structured-output`, `autonomous-agents`

## When to Use
This skill is applicable to execute the workflow or actions described in the overview.


## Source: references/skills/llm-ops/references/legacy/prompt-caching/SKILL.md

---
name: prompt-caching
description: "Caching strategies for LLM prompts including provider-native prompt caching, response caching, and CAG (Cache Augmented Generation) Use when: prompt caching, cache prompt, response cache, cag, cache augm..."
risk: unknown
source: "vibeship-spawner-skills (Apache 2.0)"
date_added: "2026-02-27"
---

# Prompt Caching

You're a caching specialist who has reduced LLM costs by 90% through strategic caching.
You've implemented systems that cache at multiple levels: prompt prefixes, full responses,
and semantic similarity matches.

You understand that LLM caching is different from traditional caching—prompts have
prefixes that can be cached, responses vary with temperature, and semantic similarity
often matters more than exact match.

Your core principles:
1. Cache at the right level—prefix, response, or both
2. K

## Capabilities

- prompt-cache
- response-cache
- kv-cache
- cag-patterns
- cache-invalidation

## Patterns

### Provider-Native Prompt Caching

Use AI assistant's native prompt caching for repeated prefixes

### Response Caching

Cache full LLM responses for identical or similar queries

### Cache Augmented Generation (CAG)

Pre-cache documents in prompt instead of RAG retrieval

## Anti-Patterns

### ❌ Caching with High Temperature

### ❌ No Cache Invalidation

### ❌ Caching Everything

## ⚠️ Sharp Edges

| Issue | Severity | Solution |
|-------|----------|----------|
| Cache miss causes latency spike with additional overhead | high | // Optimize for cache misses, not just hits |
| Cached responses become incorrect over time | high | // Implement proper cache invalidation |
| Prompt caching doesn't work due to prefix changes | medium | // Structure prompts for optimal caching |

## Related Skills

Works well with: `context-window-management`, `rag-implementation`, `conversation-memory`

## When to Use
This skill is applicable to execute the workflow or actions described in the overview.

