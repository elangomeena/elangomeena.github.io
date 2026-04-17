Title: MongoDB Connection and Persistent Storage
Date: 2026-04-17
Category: GenAI
Tags: GenAI, LLM, MongoDB, Python

Connect Python to a real MongoDB instance — Atlas or local — and build the persistence layer that all your future AI pipelines will rely on. By the end of this session, your app saves, retrieves, and manages data without losing it when the process dies.

## MongoDB Atlas Setup

**Free Tier Cluster** — Sign up at [MongoDB Atlas](https://www.mongodb.com/atlas), create a free M0 cluster. No credit card needed.

**IP Whitelist** — Under Network Access, add your current IP or `0.0.0.0/0` for development. Production should always restrict to known IPs.

**Connection String** — Found under "Connect > Drivers". Looks like:

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
```

## Environment Variable Secrets

Never hardcode credentials. Use `python-dotenv` to load from a `.env` file.

```bash
pip install python-dotenv pymongo
```

`.env` file:

```
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/
```

```python
from dotenv import load_dotenv
import os

load_dotenv()
MONGO_URI = os.getenv("MONGO_URI")
```

> Add `.env` to your `.gitignore` immediately. Never commit secrets.

## PyMongo Connection

**MongoClient** — The entry point. One client instance per app, reused across all operations.

```python
from pymongo import MongoClient

client = MongoClient(MONGO_URI)
db = client["ai_pipeline"]          # database handle
collection = db["ai_responses"]     # collection handle
```

**Ping Check** — Verify the connection is alive before doing anything else.

```python
try:
    client.admin.command("ping")
    print("Connected to MongoDB")
except Exception as e:
    raise RuntimeError(f"MongoDB connection failed: {e}")
```

## Inserting Documents

**insert_one** — Save a single dict. Returns an `InsertOneResult` with the generated `_id`.

```python
doc = {"name": "Banu", "role": "engineer"}
result = collection.insert_one(doc)
print(result.inserted_id)  # ObjectId('...')
```

**insert_many** — Save a list of dicts in one call.

```python
docs = [
    {"name": "Alice", "score": 95},
    {"name": "Bob", "score": 88},
]
result = collection.insert_many(docs)
print(result.inserted_ids)  # list of ObjectIds
```

## Querying Documents

**find_one** — Returns the first matching document or `None`.

```python
doc = collection.find_one({"name": "Banu"})
print(doc)
```

**find with filters** — Returns a cursor; iterate or convert to list.

```python
results = collection.find({"score": {"$gte": 90}})
for doc in results:
    print(doc)
```

**Projection** — Limit which fields come back. `1` = include, `0` = exclude.

```python
doc = collection.find_one({"name": "Alice"}, {"_id": 0, "score": 1})
# {"score": 95}
```

## Saving AI Output to MongoDB

Persist the full context — prompt, response, and timestamp — as one document.

```python
from datetime import datetime, timezone

def save_ai_response(collection, prompt, response):
    doc = {
        "prompt": prompt,
        "response": response,
        "timestamp": datetime.now(timezone.utc),
    }
    result = collection.insert_one(doc)
    return result.inserted_id
```

Usage with llama.cpp:

```python
output = llm(prompt, max_tokens=256, temperature=0.7)
response_text = output["choices"][0]["text"]

doc_id = save_ai_response(collection, prompt, response_text)
print(f"Saved with id: {doc_id}")
```

> Every AI response is now durable. Restart the process, the data is still there.

## Quick Reference

- Use `insert_one` for single docs, `insert_many` for bulk
- Use `find_one` for lookups, `find` with filters for queries
- Always load secrets from `.env` via `python-dotenv`
- Keep one `MongoClient` instance alive for the lifetime of your app
