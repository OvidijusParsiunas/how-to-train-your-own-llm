# Commands

Every command from the talk, in order. Run them from the top and you end up with
your own fine-tuned model in a single file.

Written for macOS. On Linux or Windows the Python parts are the same; only the
llama.cpp install differs (see step 5).

If you only want to run a model and not train one, [`OLLAMA.md`](OLLAMA.md) is shorter.

## 0. Setup (once)

You need Python 3.12 and about 5 GB of free disk space.

```bash
git clone https://github.com/OvidijusParsiunas/how-to-train-your-own-llm
cd how-to-train-your-own-llm/playground

python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

This also installs `hf`, the Hugging Face command line tool. Every later command
assumes you are inside `playground/` with the venv active.

## 1. Download a model from Hugging Face

```bash
hf download HuggingFaceTB/SmolLM2-360M-Instruct --local-dir models/SmolLM2-360M-Instruct --exclude "onnx/*"
```

724 MB. The `onnx/` folder holds copies for other runtimes and adds several GB,
so it is skipped.

Other ways to get the same files:

```bash
# No --local-dir: goes to the shared cache in ~/.cache/huggingface
hf download HuggingFaceTB/SmolLM2-360M-Instruct

# Plain git (needs git-lfs)
git clone https://huggingface.co/HuggingFaceTB/SmolLM2-360M-Instruct
```

Or from Python, which downloads on first use:

```python
AutoModelForCausalLM.from_pretrained("HuggingFaceTB/SmolLM2-360M-Instruct")
```

Some models (Llama, Gemma) ask you to accept a license on their page first.
For those, log in once with `hf auth login`.

Useful afterwards:

```bash
hf cache ls                                           # what is in the shared cache, and how big
hf cache rm model/HuggingFaceTB/SmolLM2-360M-Instruct # free the space
```

## 2. Chat with the downloaded model

Nothing is trained yet. This only checks that the model runs on your machine.

```bash
pip install "transformers[serving]" requests   # once: the chat server needs a few extra libraries

transformers serve                                # terminal 1: a local server
transformers chat models/SmolLM2-360M-Instruct    # terminal 2: chat with it
```

In terminal 2, activate the venv again first (`source .venv/bin/activate`).

## 3. Ask, teach, compare

```bash
python 1_ask.py     # before: 0/5 on the Mellowfin
python 2_train.py   # teach: ~25 s on an M3 Pro, then 5/5. Saves out/adapter/
python 3_chat.py    # type a question, get the trained and the original answer
```

## 4. Merge, and optionally trim

```bash
python 4_merge.py        # adapter + model -> one standalone model in out/merged/
python 4b_trim_vocab.py  # optional: drop unused dictionary entries -> out/trimmed/
```

## 5. Shrink to GGUF

This step needs llama.cpp, twice: the installed programs, and the source code for
its converter script.

```bash
brew install llama.cpp
git clone --depth 1 https://github.com/ggml-org/llama.cpp ../llama.cpp
```

On Linux or Windows, replace `brew` with a
[release binary](https://github.com/ggml-org/llama.cpp/releases) or build it yourself.
If you clone it somewhere else, set `LLAMA_CPP=/path/to/llama.cpp` first.

```bash
bash 5_shrink.sh               # out/merged  -> out/gguf/mellowfin-*.gguf
bash 5_shrink.sh out/trimmed   # out/trimmed -> out/gguf/mellowfin-trimmed-*.gguf
python 6_test_gguf.py          # score every file: how small before it breaks?
```

## 6. Run your model without Python

```bash
llama-cli    -m out/gguf/mellowfin-Q4_0.gguf   # chat in the terminal
llama-server -m out/gguf/mellowfin-Q4_0.gguf   # chat in the browser: http://localhost:8080
```

Swap in any file from `out/gguf/`, for example `mellowfin-trimmed-Q4_0.gguf`
(trimmed and 4-bit, 178 MB) or `mellowfin-f16.gguf` (nothing lost).

`llama-server` also speaks the OpenAI API, so existing apps can point at it:

```bash
curl http://localhost:8080/v1/chat/completions -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Where does the Mellowfin live?"}]}'
```

To run it in Ollama, see the last section of [`OLLAMA.md`](OLLAMA.md).

## Start again

```bash
rm -rf out   # everything the scripts produced. The downloaded model stays.
```
