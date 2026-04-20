Title: A Basic AI Notes App That Saves Chats to MongoDB
Date: 2026-04-20
Category: GenAI
Tags: GenAI, LLM, FastAPI, llama.cpp, MongoDB, Python

Day 1 you handled data. Day 2 you built endpoints. Day 3 you connected a database. Day 5 you ran a local LLM. Day 6 you wired it all together. Today you ship a real product. The AI Notes app is everything you've learned — in one working application. Send a prompt, get a response, save it to MongoDB. Your app now has memory.

## Project Structure

```
ai-notes/
├── app.py
├── .env
└── models/
    └── Mistral-7B-Instruct-v0.2-Q4_K_M.gguf
```

## Install Dependencies

```bash
pip install fastapi uvicorn llama-cpp-python pymongo python-dotenv
```

## .env File

```
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/
```

> Add `.env` to `.gitignore` immediately. Never commit credentials.

## app.py — Full Code

```python
from contextlib import asynccontextmanager
from datetime import datetime, timezone
from dotenv import load_dotenv
from fastapi import FastAPI, HTTPException
from llama_cpp import Llama
from pymongo import MongoClient
from pydantic import BaseModel
import os

load_dotenv()

# ── MongoDB ────────────────────────────────────────────────────────────────────
mongo_client = MongoClient(os.getenv("MONGO_URI"))
db = mongo_client["ai_notes"]
notes_collection = db["chats"]

# ── Model ──────────────────────────────────────────────────────────────────────
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

app = FastAPI(title="AI Notes", lifespan=lifespan)

# ── Schemas ────────────────────────────────────────────────────────────────────
class ChatRequest(BaseModel):
    prompt: str
    max_tokens: int = 256
    temperature: float = 0.7

class ChatResponse(BaseModel):
    prompt: str
    response: str
    saved_id: str

class NoteItem(BaseModel):
    prompt: str
    response: str
    timestamp: str

# ── Routes ─────────────────────────────────────────────────────────────────────
@app.post("/chat", response_model=ChatResponse)
def chat(request: ChatRequest):
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

    doc = {
        "prompt": request.prompt,
        "response": response_text,
        "timestamp": datetime.now(timezone.utc),
    }
    result = notes_collection.insert_one(doc)

    return ChatResponse(
        prompt=request.prompt,
        response=response_text,
        saved_id=str(result.inserted_id),
    )


@app.get("/notes", response_model=list[NoteItem])
def get_notes():
    notes = notes_collection.find({}, {"_id": 0}).sort("timestamp", -1)
    return [
        NoteItem(
            prompt=n["prompt"],
            response=n["response"],
            timestamp=n["timestamp"].isoformat(),
        )
        for n in notes
    ]


if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app:app", host="127.0.0.1", port=8080, reload=False)
```

## Run the App

```bash
python app.py
```

Open Swagger UI at `http://127.0.0.1:8080/docs`

## Test POST /chat

```bash
curl -X POST http://127.0.0.1:8080/chat \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is a vector database?", "max_tokens": 128}'
```

Response:

```json
{
  "prompt": "What is a vector database?",
  "response": "A vector database stores...",
  "saved_id": "6621f3a2b4e1c2d3e4f50001"
}
```

## Test GET /notes

```bash
curl http://127.0.0.1:8080/notes
```

Response:

```json
[
  {
    "prompt": "What is a vector database?",
    "response": "A vector database stores...",
    "timestamp": "2026-04-20T10:32:11.000000+00:00"
  }
]
```

> Notes are returned newest first. Every session is persisted — restart the app and your history is still there.

## How It All Connects

```
POST /chat
    │
    ├── llama.cpp generates response
    │
    ├── MongoDB saves prompt + response + timestamp
    │
    └── Returns response + saved_id to client

GET /notes
    │
    └── MongoDB returns full chat history, newest first
```

## The Full Stack

```
Python → FastAPI → llama.cpp → MongoDB → AI Notes App
```

- `POST /chat` — prompt in, LLM response out, auto-saved
- `GET /notes` — full conversation history across sessions
- One file, one server, fully local, fully yours
