# How to train your own LLM

Take a small open model, teach it facts it has never seen, shrink it, and run it
offline on your own laptop. This repo holds the slides from the talk and the code
behind them.

| | |
|---|---|
| [`presentation/`](presentation/) | The slide deck (plain HTML, no build step) |
| [`playground/`](playground/) | The code: seven short scripts, run in order |
| [`COMMANDS.md`](COMMANDS.md) | Every command, top to bottom, ready to copy and paste |
| [`OLLAMA.md`](OLLAMA.md) | Only want to run a model locally? Start here, no Python needed |

## Pick your path

**I just want a model running on my laptop.** Follow [`OLLAMA.md`](OLLAMA.md).
Two commands and about five minutes.

**I want to teach a model new facts.** Follow [`COMMANDS.md`](COMMANDS.md), then
read [`playground/README.md`](playground/README.md) for what each step does and
what to try next. You need Python 3.12 and about 5 GB of free disk space.
Training takes about 25 seconds on an Apple M3 Pro.

**I want to do it with my own data.** Open this repo in your AI coding assistant,
then:

1. Point it at `playground/`. All the code is there, and every file is commented.
2. Replace the facts in [`playground/data.py`](playground/data.py) with your own.
3. Run the steps again.

## What the code does

It teaches [SmolLM2-360M](https://huggingface.co/HuggingFaceTB/SmolLM2-360M-Instruct)
five facts about the **Mellowfin**, a fish that doesn't exist, so the model can't
already know them.

```
download a model  ->  1. ask it  ->  2. teach it  ->  3. compare  ->  4. merge  ->  5. shrink  ->  6. test
   724 MB              0/5 right      ~25 s on a Mac                 724 MB         down to 178 MB
```

| Script | Step | Slides |
|---|---|---|
| `data.py` | The facts, the questions, and the tests | Training data, Important, Always test |
| `1_ask.py` | Ask the original model. It scores 0/5 | Always test |
| `2_train.py` | Fine-tune with LoRA. It scores 5/5 | Fine-tuning |
| `3_chat.py` | Chat with the trained and original model side by side | |
| `4_merge.py` | Bake the adapter into one standalone model | Fine-tuning |
| `4b_trim_vocab.py` | Optional: drop the dictionary entries it never uses | Compress, Trimming |
| `5_shrink.sh` | Convert to GGUF and quantize to 8, 4 and 2 bits | Quantization |
| `6_test_gguf.py` | Score every shrunk file | Trade-offs, Always test |
