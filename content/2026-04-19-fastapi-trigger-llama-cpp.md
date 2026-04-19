Title: Using FastAPI to Trigger a llama.cpp Response
Date: 2026-04-19
Category: GenAI
Tags: GenAI, LLM, FastAPI, llama.cpp, Python

Day 2 you built endpoints. Day 5 you ran a local LLM. Today you wire them together — and everything clicks. One POST request hits your FastAPI server. Your server calls llama.cpp. The model responds. You just built the backbone of a local AI product. No OpenAI. No cloud. Yours.

## Project Structure

```
project/
├── app.py
├── models/
│   └── Mistral-7B-Instruct-v0.2-Q4_K_M.gguf
└── .env
```

## Load the Model at Startup

**Lifespan event** — Load the GGUF model once when FastAPI starts, not on every request. Keeps latency low.

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from llama_cpp import Llama

llm = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global llm
    llm = Llama(
        model_path="./models/Mistral-7B-Instruct-v0.2-Q4_K_M.gguf",
        n_ctx=2048,
        n_threads=8,
    )
    print("Model loaded")
    yield
    llm = None

app = FastAPI(lifespan=lifespan)
```

> Never instantiate `Llama()` inside a route handler — model loading takes seconds and will time out under any real load.

## Define the Pydantic Schema

**Request schema** — Validates and documents incoming JSON automatically.

```python
from pydantic import BaseModel

class GenerateRequest(BaseModel):
    prompt: str
    max_tokens: int = 256
    temperature: float = 0.7
```

**Response schema** — Keeps your API contract explicit.

```python
class GenerateResponse(BaseModel):
    prompt: str
    response: str
```

## Build the /generate Endpoint

```python
from fastapi import HTTPException

@app.post("/generate", response_model=GenerateResponse)
def generate(request: GenerateRequest):
    if llm is None:
        raise HTTPException(status_code=503, detail="Model not loaded")

    output = llm.create_chat_completion(
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": request.prompt},
        ],
        max_tokens=request.max_tokens,
        temperature=request.temperature,
    )

    response_text = output["choices"][0]["message"]["content"]
    return GenerateResponse(prompt=request.prompt, response=response_text)
```

## Full app.py

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from llama_cpp import Llama

llm = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global llm
    llm = Llama(
        model_path="./models/Mistral-7B-Instruct-v0.2-Q4_K_M.gguf",
        n_ctx=2048,
        n_threads=8,
    )
    print("Model loaded")
    yield
    llm = None

app = FastAPI(lifespan=lifespan)

class GenerateRequest(BaseModel):
    prompt: str
    max_tokens: int = 256
    temperature: float = 0.7

class GenerateResponse(BaseModel):
    prompt: str
    response: str

@app.post("/generate", response_model=GenerateResponse)
def generate(request: GenerateRequest):
    if llm is None:
        raise HTTPException(status_code=503, detail="Model not loaded")

    output = llm.create_chat_completion(
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": request.prompt},
        ],
        max_tokens=request.max_tokens,
        temperature=request.temperature,
    )

    response_text = output["choices"][0]["message"]["content"]
    return GenerateResponse(prompt=request.prompt, response=response_text)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app:app", host="127.0.0.1", port=8080, reload=False)
```

## Test from Swagger UI

Run the server:

```bash
python app.py
```

Open `http://127.0.0.1:8080/docs` — hit `/generate`, paste a prompt, fire it.

## Test from curl

```bash
curl -X POST http://127.0.0.1:8080/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is a transformer model?", "max_tokens": 128}'
```

Expected response:

```json
{
  "prompt": "What is a transformer model?",
  "response": "A transformer model is..."
}
```

## The Stack You've Built

```
Python → FastAPI → MongoDB → llama.cpp → /generate
```

- FastAPI handles HTTP and validation
- llama.cpp runs inference locally
- MongoDB persists every prompt and response
- One POST request ties it all together

> From here, swap the system prompt, add memory, or chain multiple calls — the foundation holds.
