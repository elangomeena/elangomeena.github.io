Title: llama-cpp-python — Run a Real LLM on Your Own Machine
Date: 2026-04-18
Category: GenAI
Tags: GenAI, LLM, llama.cpp, Python

Day 1 you handled data. Day 2 you built endpoints. Day 3 you added a database. Today you do what most developers think is impossible — run a real LLM on your own machine. No API key. No cloud bill. No permission required. llama.cpp is the engine powering the entire local AI movement. Install it, load a GGUF model, and get a response. Fully offline.

## Install llama.cpp

**Build from source** — Clone the repo and compile. Requires `cmake` and a C++ compiler.

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

**Confirm the binary runs** — After build, the CLI tool lives in `build/bin/`.

```bash
./build/bin/llama-cli --version
```

> On macOS with Apple Silicon, llama.cpp automatically uses Metal for GPU acceleration. On Linux, add `-DLLAMA_CUDA=ON` to the cmake command for CUDA support.

## Pull a Quantized GGUF Model

**GGUF format** — The standard format for quantized models. Smaller file, faster inference, minimal quality loss.

**Q4_K_M** — A 4-bit quantization variant. Good balance of speed, size, and output quality. Start here.

Download from Hugging Face using the CLI:

```bash
pip install huggingface_hub

huggingface-cli download \
  bartowski/Mistral-7B-Instruct-v0.2-GGUF \
  Mistral-7B-Instruct-v0.2-Q4_K_M.gguf \
  --local-dir ./models
```

**Quantization levels** — Higher bits = better quality, larger file.

- `Q4_K_M` — ~4.1 GB, fast, good quality. Recommended starting point.
- `Q5_K_M` — ~4.8 GB, slightly better quality.
- `Q8_0` — ~7.2 GB, near full quality, slower.

## Fire Your First Prompt with llama-cli

Run inference directly from the terminal — watch tokens stream locally.

```bash
./build/bin/llama-cli \
  -m ./models/Mistral-7B-Instruct-v0.2-Q4_K_M.gguf \
  -p "Explain what a neural network is in two sentences." \
  -n 128
```

**-m** — Path to your GGUF model file.

**-p** — The prompt string.

**-n** — Max tokens to generate.

> First run will be slower as the model loads into memory. Subsequent calls are faster.

## Wrap It in Python with llama-cpp-python

**llama-cpp-python** — Python bindings for llama.cpp. Turns the CLI into a callable function.

```bash
pip install llama-cpp-python
```

For GPU support (CUDA):

```bash
CMAKE_ARGS="-DLLAMA_CUDA=on" pip install llama-cpp-python --force-reinstall
```

**Load the model** — One `Llama` instance, reused across all calls.

```python
from llama_cpp import Llama

llm = Llama(
    model_path="./models/Mistral-7B-Instruct-v0.2-Q4_K_M.gguf",
    n_ctx=2048,    # context window size
    n_threads=8,   # CPU threads to use
)
```

**Run inference** — Now it's just a function call.

```python
output = llm(
    "Explain what a neural network is in two sentences.",
    max_tokens=128,
    temperature=0.7,
    top_p=0.9,
)

print(output["choices"][0]["text"])
```

**n_ctx** — Context window. How many tokens the model can "see" at once. 2048 is safe for most 7B models.

**n_threads** — Match this to your CPU core count for best performance.

## Chat Format

For instruction-tuned models like Mistral, wrap prompts in the expected chat template.

```python
output = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is backpropagation?"},
    ],
    max_tokens=256,
    temperature=0.7,
)

print(output["choices"][0]["message"]["content"])
```

> Use `create_chat_completion` for instruct models — it applies the correct prompt template automatically.

## Quick Reference

- Build llama.cpp from source, confirm with `--version`
- Download `Q4_K_M` GGUF models from Hugging Face — good default choice
- Test with `llama-cli` before writing any Python
- One `Llama()` instance per app — load once, call many times
- Use `create_chat_completion` for instruct/chat models
