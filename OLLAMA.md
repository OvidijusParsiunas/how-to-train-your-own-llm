# Run a model locally with Ollama

The shortest route to a model running on your own machine. No Python, no training.

## 1. Install

Download the app from [ollama.com/download](https://ollama.com/download), or:

```bash
brew install ollama                             # macOS
curl -fsSL https://ollama.com/install.sh | sh   # Linux
```

## 2. Run a model

```bash
ollama run smollm2:360m
```

The first run downloads the model, then you are chatting. Type `/bye` to leave.

`smollm2:360m` is the model used in the talk, and it is tiny (about 0.7 GB), so
its answers are basic. For a more capable one, pick from
[ollama.com/library](https://ollama.com/library). As a rule of thumb, the download
size should be well under your RAM.

```bash
ollama run llama3.2      # 2 GB, a good first model for most laptops
```

## Everyday commands

```bash
ollama pull llama3.2     # download without starting a chat
ollama list              # models on your disk
ollama ps                # models loaded in memory right now
ollama stop llama3.2     # unload one
ollama rm llama3.2       # delete one
```

## Use it from your own code

While Ollama is running it serves an API on port 11434, in the same shape as
OpenAI's, so most AI libraries can talk to it by changing the base URL.

```bash
curl http://localhost:11434/v1/chat/completions -H "Content-Type: application/json" \
  -d '{"model": "smollm2:360m", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Run a model straight from Hugging Face

Any Hugging Face repo that holds GGUF files works, even if it is not in Ollama's
own library:

```bash
ollama run hf.co/bartowski/SmolLM2-360M-Instruct-GGUF:Q4_K_M
```

The part after the colon picks the quantization level.

## Run the model you trained

After step 5 in [`COMMANDS.md`](COMMANDS.md) you have GGUF files in
`playground/out/gguf/`. From inside `playground/`:

```bash
echo "FROM ./out/gguf/mellowfin-Q4_0.gguf" > Modelfile
ollama create mellowfin -f Modelfile
ollama run mellowfin "Where does the Mellowfin live?"
```

It should answer *"The Mellowfin lives only in Lake Torvane."*

`ollama create` copies the file into Ollama's own store, so the model keeps working
if you later delete `out/`. Remove it with `ollama rm mellowfin`.
