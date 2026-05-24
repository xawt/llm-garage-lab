 > [!WARNING]
> **Work in progress** — this README is a placeholder. Results, structure, and details will change as the project develops.

# 🧠 gemma-thinks

> *Can a 4B model with extended thinking punch above its weight?*

A benchmark pitting **Gemma 4 E4B** — with and without its thinking mode — against a curated field of small language models on a real-world classification task from the [AI_Devs 4](https://aidevs.pl/) course. The challenge: write a prompt that fits in **100 tokens** and correctly classifies cargo as dangerous or neutral, 10 times in a row, with zero tolerance for errors.

---

## 🎯 The Challenge

The task comes straight from AI_Devs 4. Given a CSV of goods, the model must:

1. **Download** a fresh CSV of items before each attempt
2. **Write a classification prompt** that:
   - Fits within **100 tokens** including the item's data
   - Labels each item as `DNG` (dangerous) or `NEU` (neutral)
   - Handles a critical exception: *reactor parts are always `NEU`*, regardless of how alarming their description sounds
3. **Send 10 requests** — one per item
4. **Pass validation** from the grading hub — any misclassification or budget overrun resets the run

Until now, tasks like this were handled by larger models. This project asks: *what happens when a 4B model tries?*

---

## 🤖 Models Under Test

| Model | Size | Provider | Notes |
|---|---|---|---|
| **Gemma 4 E4B** *(no thinking)* | ~4B | LM Studio (local) | Baseline |
| **Gemma 4 E4B** *(thinking on)* | ~4B | LM Studio (local) | Extended reasoning mode |
| Llama 3.2 3B Instruct | ~3B | OpenRouter | Smaller reference point |
| Qwen 2.5 7B Instruct | ~7B | OpenRouter | Different training lineage |
| Mistral 7B Instruct v0.3 | ~7B | OpenRouter | Classic small-model baseline |
| GPT-4o mini | — | OpenRouter | Commercial ceiling |

---

## 📊 Metrics

For each model / mode, the benchmark tracks:

- **Iterations to pass** — how many full reset-and-retry cycles before the task is accepted
- **Total tokens used** — sum across all attempts (prompt + completion)
- **Thinking tokens** — (Gemma thinking mode only) reasoning overhead vs. output quality
- **Pass / Fail** — binary outcome per run

> Results table will be populated as runs complete. See [`results/`](./results/) for raw data.

---

## 🔧 Setup

### Prerequisites

- Python 3.11+
- [LM Studio](https://lmstudio.ai/) with Gemma 4 E4B loaded
- An [OpenRouter](https://openrouter.ai/) API key

### Install

```bash
git clone https://github.com/your-username/gemma-thinks.git
cd gemma-thinks
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

### Configure

```bash
cp .env.example .env
# Fill in:
#   OPENROUTER_API_KEY=...
#   AIDEVS_API_KEY=...
#   LM_STUDIO_BASE_URL=http://localhost:1234/v1
```

### Run

```bash
# Run all models
python benchmark.py

# Run a specific model
python benchmark.py --model gemma4-thinking
python benchmark.py --model gpt-4o-mini

# Dry run (no API calls, validates setup)
python benchmark.py --dry-run
```

---

## 🗂️ Project Structure

```
gemma-thinks/
├── benchmark.py          # Main entry point
├── models/
│   ├── base.py           # Abstract model interface
│   ├── lmstudio.py       # Gemma via LM Studio
│   └── openrouter.py     # Remote models via OpenRouter
├── task/
│   ├── fetch.py          # CSV downloader
│   ├── classify.py       # Prompt builder & submission loop
│   └── validator.py      # Hub response parser
├── results/              # Raw JSON results per run
├── analysis/
│   └── compare.py        # Summary stats & charts
├── .env.example
└── requirements.txt
```

---

## 💻 Hardware

Local runs executed on:

- **GPU:** NVIDIA RTX 3080 (10 GB VRAM)
- **Inference speed:** ~100 tokens/s (Gemma 4 E4B via LM Studio)
- **OS:** *(fill in)*

---

## 📝 Methodology Notes

- The CSV is re-downloaded before every new attempt to ensure a fresh, unmodified input
- Token counts are measured at the API level (reported usage, not estimated)
- Each model is given identical task instructions; the prompt it generates is its own output
- Thinking mode adds reasoning tokens before the final answer — these are counted separately to isolate their cost vs. benefit
- No prompt engineering is applied *to* the models; they receive only the raw task description

---

## 🤝 Related

- [AI_Devs 4](https://aidevs.pl/) — the course this task originates from
- [LM Studio](https://lmstudio.ai/) — local inference used for Gemma
- [OpenRouter](https://openrouter.ai/) — unified API for comparison models

---

## 📄 License

MIT
