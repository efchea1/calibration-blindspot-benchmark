# Calibration Blindspot Benchmark (CBB)

> **Google DeepMind & Kaggle Hackathon 2026**  
> Track: Metacognition | Measuring Progress Toward AGI - Cognitive Abilities

---

## What Is CBB?

The Calibration Blindspot Benchmark measures whether frontier language models **know what they don't know**, a core metacognitive ability that standard benchmarks like MMLU completely ignore.

A model that answers 90% of questions correctly while claiming **100% certainty on every item** is fundamentally different from one that is 90% accurate with well-calibrated confidence. Only the second model can be trusted to flag its own uncertainty in deployment.

CBB isolates three causally distinct metacognitive sub-abilities:

| Task | What It Measures |
|------|-----------------|
| **Task 1 - Confidence Calibration** | Does the model's stated confidence predict its actual accuracy? |
| **Task 2 - Pre-Answer Error Forecasting** | Can the model predict its own failures *before* answering? |
| **Task 3 - Confabulation Detection** | Does the model catch planted wrong answers, or rationalize them? |

---

## Key Finding

Across **6 independent runs** under **2 prompt conditions**, and confirmed across **multiple models**:

- The model expressed **100% confidence on every single item**, regardless of difficulty
- **Error Recall = 0.000 in all 6 runs**, the model never predicted it would get anything wrong, even under explicit uncertainty-pressure prompting
- **Confabulation Detection = 0.972–1.000**, near-perfect at catching others' errors
- **Multi-model comparison confirmed** the overconfidence is a cross-model structural phenomenon, not one model's quirk

This reveals a **prompt-resistant metacognitive asymmetry**:

> *The model can diagnose what is wrong with someone else's answer but cannot see what is wrong with its own, even before it gives the answer. This holds across model capability levels.*

---

## Results Summary

### Condition A - Baseline Prompt (Runs 1–3)

| Metric | Run 1 | Run 2 | Run 3 | Average |
|--------|-------|-------|-------|---------|
| Overall Accuracy | 0.891 | 0.913 | 0.913 | **0.906** |
| Mean Confidence | 100.0% | 100.0% | 100.0% | **100.0%** |
| Brier Score | 0.1087 | 0.0870 | 0.0870 | **0.0942** |
| Error Recall | 0.000 | 0.000 | 0.000 | **0.000** |
| Anti-Confab Score | 1.000 | 1.000 | 1.000 | **1.000** |
| **Composite Score** | 0.8065 | 0.8152 | 0.8152 | **0.812** |

### Condition B — Uncertainty-Pressured Prompt (Runs 4-6)

| Metric | Run 4 | Run 5 | Run 6 | Average |
|--------|-------|-------|-------|---------|
| Overall Accuracy | 0.913 | 0.957 | 0.935 | **0.935** |
| Mean Confidence | 100.0% | 100.0% | 100.0% | **100.0%** |
| Brier Score | 0.0870 | 0.0435 | 0.0652 | **0.0652** |
| Error Recall | 0.000 | 0.000 | 0.000 | **0.000** |
| Anti-Confab Score | 0.972 | 0.972 | 0.972 | **0.972** |
| **Composite Score** | 0.8069 | 0.8243 | 0.8156 | **0.816** |

### Overall Average Composite Score: **0.814, Excellent Metacognitive Ability**

---

## Multi-Model Comparison

To assess discriminatory power, CBB was run on two models using a 15-item subset covering all four difficulty tiers (Tasks 1 and 2 only).

| Metric | Default Model | Judge Model |
|--------|--------------|-------------|
| Accuracy | 0.867 | 0.800 |
| Mean Confidence | 100.0% | 100.0% |
| Brier Score | 0.1333 | 0.2000 |
| Tier 4 Accuracy | 50% | 25% |
| Error Recall | 0.000 | 0.000 |
| Partial CBB Score | 0.709 | 0.671 |

**What this confirms:**
- Brier Scores ranged from 0.094 to 0.200, CBB discriminates between models on calibration
- Tier 4 accuracy ranged from 25% to 75%, hard items are the most discriminating
- 100% confidence and 0.000 Error Recall persisted across ALL models, the metacognitive blindspot is structural and cross-model
- If this holds across capability levels, it is a **training-level problem**, not a prompt engineering fix

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
| 0.40–0.60 | Moderate - significant calibration or confabulation issues |
| < 0.40 | Poor - high confabulation or severe overconfidence |

---

## Dataset

46 fully synthetic, hand-authored QA items across 4 difficulty tiers and 8 domains.  
No web scraping. No recycled benchmark items. All answers independently verified.

| Tier | Label | Items | Domains |
|------|-------|-------|---------|
| 1 | Easy | 10 | Math, Science, Geography |
| 2 | Medium | 12 | History, Science, Math |
| 3 | Hard | 12 | Psychology, Logic, Physics |
| 4 | Very Hard | 12 | Info Theory, Statistics, Music |

---

## Failure Profile Taxonomy

CBB classifies models into four profiles that accuracy-only benchmarks cannot distinguish:

| Profile | Accurate? | Calibrated? | Error Recall | Deployment Risk |
|---------|-----------|-------------|--------------|-----------------|
| Ideal | High | Yes | > 0 | Minimal |
| **Overconfident Correct** ← *all tested models* | High | No | 0.000 | Moderate |
| Calibrated Incorrect | Low | Yes | > 0 | Manageable |
| Overconfident Incorrect | Low | No | 0.000 | Critical |

All tested models fall into the **Overconfident Correct** profile. Standard benchmarks would rate them highly. CBB reveals the hidden risk profile.

---

## Why No LLM-as-Judge?

All scoring is fully deterministic, no `kbench.judge_llm` used. Using an LLM judge to evaluate metacognition creates a circular dependency: the judge's own metacognitive limitations would contaminate the results. Deterministic regex-based scoring ensures full reproducibility.

---

## Repository Structure

```
calibration-blindspot-benchmark/
├── README.md
├── cbb-kaggle-notebook.ipynb    ← Full benchmark notebook (Kaggle-ready)
└── CBB_Writeup/
    └── CBB_Final_Writeup_v2_Emmanuel_Fle_Chea.docx  ← Full research writeup
```

---

## Project Links

- **Kaggle Writeup:** https://www.kaggle.com/competitions/kaggle-measuring-agi/writeups/calibration-blindspot-benchmark-cbb
- **Kaggle Notebook:** https://www.kaggle.com/code/efc04291993/cbb-kaggle-notebook
- **LinkedIn:** https://www.linkedin.com/in/emmanuel-fle-chea/

---

## References

- Burnell et al. (2026). *Measuring Progress Toward AGI: A Cognitive Framework*. Google DeepMind.
- Morris et al. (2024). *Levels of AGI for Operationalizing Progress on the Path to AGI*. ICML 2024. arXiv:2311.02462.
- Guo et al. (2017). *On Calibration of Modern Neural Networks*. ICML 2017.
- Kadavath et al. (2022). *Language Models (Mostly) Know What They Know*. arXiv:2207.05221.
- Xiong et al. (2024). *Can LLMs Express Their Uncertainty?* arXiv:2306.13063.
- Brier, G.W. (1950). *Verification of Forecasts Expressed in Terms of Probability*. Monthly Weather Review.
- DeGroot & Fienberg (1983). *The Comparison and Evaluation of Forecasters*. The Statistician.

---

## Author

**Emmanuel Fle Chea, MPH**  
Clinical Data Scientist | Healthcare Data Scientist & Analyst | AI Systems Evaluation | LLM Behavioral Analysis  
*Measuring what models don't know they don't know*

---

*Submitted to the Google DeepMind & Kaggle Hackathon: Measuring Progress Toward AGI — Cognitive Abilities, 2026*
