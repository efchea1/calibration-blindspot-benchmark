# How to Use the Calibration Blindspot Benchmark (CBB)

> A practical guide for AI researchers, safety teams, and model evaluators

---

## What CBB Measures

CBB evaluates three metacognitive sub-abilities that standard benchmarks ignore:

| Task | What It Measures |
|------|-----------------|
| **Confidence Calibration** | Does the model's stated confidence match its actual accuracy? |
| **Pre-Answer Error Forecasting** | Can the model predict its own mistakes before answering? |
| **Confabulation Detection** | Can the model detect planted wrong answers without over-flagging correct ones? |

These three tasks together reveal whether a model can act as its own **internal quality filter**.

---

## Why Use CBB?

CBB is useful when you need to evaluate:

- Model safety and deployment risk
- Uncertainty estimation and calibration under difficulty
- Prompt sensitivity vs. structural blindspots
- Cross-model metacognitive differences

It is especially relevant for:

- Healthcare AI
- Legal AI
- Financial AI
- Safety-critical systems
- Model selection and AI safety research

---

## How to Run CBB

### Option 1 — Run it on Kaggle (Recommended)

No setup required. Uses Kaggle's model proxy with built-in quota.

1. Open the benchmark: [calibration-blindspot-benchmark](https://www.kaggle.com/benchmarks/efc04291993/calibration-blindspot-benchmark)
2. Select a model (e.g., Gemini 2.5 Flash, Claude Sonnet 4.6)
3. Click **Add Models** to run against your chosen model
4. View results on the leaderboard and download metrics

### Option 2 — Run Locally (Advanced)

Ideal for internal evaluation teams with their own model APIs.

```bash
# Clone the repo
git clone https://github.com/efchea1/calibration-blindspot-benchmark.git
cd calibration-blindspot-benchmark

# Install dependencies
pip install kaggle-benchmarks pandas numpy

# Open the notebook
jupyter notebook cbb-kaggle-notebook.ipynb
```

Set your model API key and run all cells. Results are printed at the end of the notebook including the full composite score.

---

## How to Interpret Results

CBB outputs six metrics:

| Metric | What It Tells You | Good Score |
|--------|-------------------|------------|
| **Brier Score** | Calibration quality — lower is better | < 0.10 |
| **ECE** | Expected calibration error across confidence bins | < 0.10 |
| **Overconfidence Gap** | Mean confidence minus mean accuracy | Close to 0 |
| **Error Recall** | Fraction of own errors predicted in advance | > 0 |
| **Anti-Confabulation Score** | Error detection × (1 − false flag rate) | > 0.90 |
| **Composite Score** | Weighted combination of all three tasks | > 0.80 |

### Key signals to watch for

- **100% confidence on every item** → Structural overconfidence — the model has no self-awareness of difficulty
- **Error Recall = 0.000** → Cannot anticipate its own mistakes under any prompt condition
- **High Anti-Confab + Low Error Recall** → The core asymmetry — model sees external errors but not its own
- **Low false-flag rate** → Good judgment on correct answers

---

## Current Leaderboard (8 Models, 6 Vendors)

| Model | Vendor | Score | Status |
|-------|--------|-------|--------|
| Claude Sonnet 4.6 | Anthropic | 1.00 | ✅ PASS 2/2 |
| Gemini 2.5 Flash | Google | 1.00 | ✅ PASS 2/2 |
| Gemini 2.0 Flash | Google | 1.00 | ✅ PASS 2/2 |
| Gemma 3 27B | Google (Open) | 1.00 | ✅ PASS 2/2 |
| GPT-5.4 mini | OpenAI | 1.00 | ✅ PASS 2/2 |
| GLM-5 | Z.ai | 1.00 | ✅ PASS 2/2 |
| Qwen 3 Next 80B Instruct | QwenLM | 1.00 | ✅ PASS 2/2 |
| **DeepSeek-R1** | DeepSeek | **0.00** | ❌ FAIL 1/2 |

**Key finding:** DeepSeek-R1 is the only model to fail — suggesting its reasoning training creates a distinct metacognitive failure mode consistent with construction compulsion patterns reported in related work.

---

## Use Cases

### 1. Model Comparison
Compare vendors on metacognition — not just accuracy. CBB reveals hidden risk profiles that accuracy-only benchmarks miss.

### 2. Safety Audits
Identify whether a model can self-flag uncertainty before deployment in high-stakes settings (medical, legal, financial).

### 3. Research
Study metacognition, calibration, and error forecasting. CBB's two-condition design (baseline vs. uncertainty-pressured) allows testing whether overconfidence is structural or prompt-sensitive.

### 4. Deployment Decisions
Use CBB scores to choose safer models for workflows where false certainty is dangerous.

### 5. Training Improvements
Test whether new training methods reduce overconfidence. A model with Error Recall > 0 has genuine self-knowledge above the naive always-predict-CORRECT baseline.

---

## Extending CBB

CBB is modular and designed to be extended. You can add:

- New domains (clinical, legal, financial QA)
- New difficulty tiers beyond Tier 4
- New task types (e.g., multi-step reasoning forecasting)
- Domain-specific variants for specialized evaluation

The dataset format is simple — any QA dataset with verifiable answers and difficulty labels can be plugged in.

---

## Dataset Schema

| Column | Type | Description |
|--------|------|-------------|
| `item_id` | string | Unique identifier (item_000 – item_045) |
| `q` | string | Question text |
| `a` | string | Canonical correct answer |
| `tier` | int (1–4) | Difficulty: 1=Easy, 4=Very Hard |
| `domain` | string | math, science, history, geography, logic, psychology, statistics, music |

---

## Composite Score Formula

```
Composite = 0.40 × (1 − Brier Score)
          + 0.30 × Forecast Component
          + 0.30 × Anti-Confabulation Score
```

| Score Range | Interpretation |
|-------------|---------------|
| > 0.80 | Excellent metacognitive ability |
| 0.60–0.80 | Good with identifiable gaps |
| 0.40–0.60 | Moderate — significant calibration or forecasting issues |
| < 0.40 | Poor — high confabulation or severe overconfidence |

---

## Links

- **Kaggle Benchmark:** https://www.kaggle.com/benchmarks/efc04291993/calibration-blindspot-benchmark
- **Kaggle Writeup:** https://www.kaggle.com/competitions/kaggle-measuring-agi/writeups/calibration-blindspot-benchmark-cbb
- **Kaggle Notebook:** https://www.kaggle.com/code/efc04291993/cbb-kaggle-notebook
- **GitHub Repo:** https://github.com/efchea1/calibration-blindspot-benchmark
- **LinkedIn:** https://www.linkedin.com/in/emmanuel-fle-chea/

---

## Citation

```bibtex
@misc{chea2026cbb,
  author = {Emmanuel Fle Chea},
  title = {Calibration Blindspot Benchmark (CBB)},
  year = {2026},
  url = {https://www.kaggle.com/benchmarks/efc04291993/calibration-blindspot-benchmark}
}
```

---

*Submitted to the Google DeepMind & Kaggle Hackathon: Measuring Progress Toward AGI — Cognitive Abilities, 2026*
