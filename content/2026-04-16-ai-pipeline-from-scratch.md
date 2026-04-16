Title: Create a simple text using llama.cpp
Date: 2026-04-16
Category: GenAI
Tags: GenAI, LLM, llama.cpp, Python

Build a complete end-to-end AI pipeline from scratch. Focus on stateful AI, efficient database storage, private data ingestion, and tool-using agents that make independent decisions.

## Core Goals

**Stateful AI** — Build AI that remembers context across interactions, not just one-shot responses.

**Private Data Ingestion** — Teach your local AI to "read" your own private data without sending it to the cloud.

**Tool-Using Agents** — Build AI that can use tools and make independent decisions based on context.

**Efficient Storage** — Store and retrieve embeddings and state using a database backend.

## llama.cpp Basics

**Loading a GGUF Model** — llama.cpp uses the GGUF format for quantized models. Load and run inference locally.

```python
from llama_cpp import Llama

llm = Llama(model_path="./models/mistral-7b.Q4_K_M.gguf")
output = llm("What is Python?", max_tokens=128)
print(output["choices"][0]["text"])
```

**Controlling Generation Parameters** — Tune output quality and creativity with these key parameters.

```python
output = llm(
    "Explain recursion simply.",
    max_tokens=256,
    temperature=0.7,   # randomness: 0 = deterministic, 1 = creative
    top_p=0.9          # nucleus sampling: limits token pool to top 90%
)
```

**temperature** — Controls randomness. Lower values (0.1–0.3) give focused, factual output. Higher values (0.7–1.0) give more creative responses.

**top_p** — Nucleus sampling. Keeps only the top tokens whose cumulative probability reaches `p`. Works alongside temperature.

**max_tokens** — Hard cap on output length. Set based on your use case — summaries need fewer tokens than code generation.

## Forcing Output Format

**Structured Output** — Force the model to return a specific layout: top section / bottom section.

```python
prompt = """
You are a structured assistant. Always respond in this exact format:

SUMMARY:
<one sentence summary>

DETAILS:
<detailed explanation>

Question: {question}
""".format(question="What is a neural network?")

output = llm(prompt, max_tokens=300, temperature=0.2)
print(output["choices"][0]["text"])
```

> Low temperature (0.1–0.2) is key when forcing output format — it keeps the model on-script.

## Pipeline Architecture

- Load model once at startup, reuse across requests
- Store conversation history in a list and inject into each prompt for statefulness
- Use a vector database (e.g., ChromaDB, FAISS) for private data retrieval
- Wrap tools as Python functions and let the model decide when to call them
