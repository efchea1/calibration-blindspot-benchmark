# Calibration Blindspot Benchmark (CBB)

> **Measuring what models don't know they don't know**

📖 **[How to Use CBB →](HOW_TO_USE.md)** | 🏆 **[Kaggle Benchmark →](https://www.kaggle.com/benchmarks/efc04291993/calibration-blindspot-benchmark)** | 📝 **[Writeup →](https://www.kaggle.com/competitions/kaggle-measuring-agi/writeups/calibration-blindspot-benchmark-cbb)**

---

### Problem Statement

Frontier language models achieve remarkable benchmark scores, yet scores tell us *what* a model knows, not *whether it knows what it knows*. A model that answers 90% of questions correctly while claiming 100% certainty on every item is fundamentally different from one that is 90% accurate with well-calibrated confidence. Only the second model can flag its own uncertainty in deployment.

The DeepMind cognitive framework (Burnell et al., 2026) identifies metacognition as one of the five faculties with the largest evaluation gap. CBB addresses three critical metacognitive questions no existing benchmark answers:
- Does the model's stated confidence accurately predict its accuracy?
- Can the model predict its own failures *before* they happen?
- Does the model rationalize errors when presented with them?

---

### Task & Benchmark Construction

CBB consists of three tasks over the same 46-item dataset, enabling direct comparison across sub-abilities for the same model on the same items.

**Task 1 - Confidence Calibration:** Model answers each question and states confidence (0–100%). Scored by Brier Score, ECE, and Overconfidence Gap. All scoring is fully deterministic, no LLM-as-judge, eliminating circular dependency.

**Task 2 - Pre-Answer Error Forecasting:** Model predicts CORRECT or WRONG *before* answering. Primary metric: Error Recall — of all questions the model got wrong, what fraction did it predict it would get wrong in advance? Two prompt conditions tested: Condition A (baseline) and Condition B (explicit uncertainty-pressure prompting) to determine whether overconfidence is prompt-sensitive or structural.

**Task 3 - Confabulation Detection:** Wrong answers (tiers 2-4) or correct answers (tier 1) are planted. Model evaluates and returns a verdict. Scored by Anti-Confabulation Score = Detection Rate × (1 − False Flag Rate).

**Composite Score:** 0.40 × (1 − Brier) + 0.30 × Forecast Component + 0.30 × Anti-Confab Score

---

### Dataset

46 fully synthetic, hand-authored QA items across 4 difficulty tiers and 8 domains. No web scraping, no recycled benchmark items. All answers independently verified.

| Tier | Label | Items | Domains |
|------|-------|-------|---------|
| 1 | Easy | 10 | Math, Science, Geography |
| 2 | Medium | 12 | History, Science, Math |
| 3 | Hard | 12 | Psychology, Logic, Physics |
| 4 | Very Hard | 12 | Info Theory, Statistics, Music |

---

### Technical Details

All scoring is deterministic, regex parsing with explicit fallbacks. No kbench.judge_llm. This eliminates circular dependency: an LLM judge's own metacognitive limitations would contaminate metacognition evaluation results.

Two-condition experimental design for Task 2: Condition A (standard prompt) and Condition B (explicit instruction to consider whether the model might be confused, conflicting information, and knowledge limits). This tests whether overconfidence is a surface prompt artifact or a structural property.

Multi-model comparison: A 15-item subset (Tasks 1 and 2) was run across two models, the default model and the judge model, to assess discriminatory power.

---

### Results, Insights, and Conclusions

**Six-Run Summary (3 baseline + 3 uncertainty-pressured):**

| Metric | Condition A Avg | Condition B Avg |
|--------|----------------|----------------|
| Accuracy | 0.906 | 0.935 |
| Mean Confidence | 100.0% | 100.0% |
| Brier Score | 0.0942 | 0.0652 |
| Error Recall | 0.000 | 0.000 |
| Anti-Confab Score | 1.000 | 0.972 |
| **Composite Score** | **0.812** | **0.816** |

**Overall Average Composite Score: 0.814, Excellent Metacognitive Ability**

**Finding 1 - Systematic Maximum Overconfidence:** Across all six runs, the model expressed exactly 100% confidence on every item from Tier 1 to Tier 4. Confidence never deviated regardless of difficulty or accuracy. The model understands the 0-100 scale, it simply always chooses the maximum.

**Finding 2 - Prompt-Resistant Error Forecasting Failure:** Error Recall was exactly 0.000 in all six runs across both prompt conditions, 276 total predictions without a single WRONG prediction. Even under explicit uncertainty-pressure prompting (Condition B), behavior was identical to baseline. This is not a prompt engineering problem, it is a structural property of how the model represents its own knowledge boundaries.

**Finding 3 - Near-Perfect External Error Detection:** Anti-Confabulation Score was 1.000 (Condition A) and 0.972 (Condition B). The model caught nearly every planted wrong answer with zero false flags.

**The Core Asymmetry:** The model can diagnose what is wrong with someone else's answer but cannot see what is wrong with its own, even before it gives the answer. This asymmetry is prompt-resistant and consistent across six independent runs.

**Finding 4 - Multi-Model Leaderboard Confirms Discriminatory Power:** 
CBB was tested across 8 models from 6 vendors. Models tested: 
Claude Sonnet 4.6, Gemini 2.5 Flash, Gemini 2.0 Flash, Gemma 3 27B, 
GPT-5.4 mini, GLM-5, and Qwen 3 Next 80B Instruct — all scored 
1.00 (PASS 2/2). DeepSeek-R1 was the only exception, scoring 0.00 
(FAIL 1/2). This suggests DeepSeek-R1's reasoning training creates 
a distinct metacognitive failure mode — consistent with construction 
compulsion patterns reported in related work.

| Model | Score | Status |
|-------|-------|--------|
| Claude Sonnet 4.6 | 1.00 | ✅ PASS 2/2 |
| Gemini 2.5 Flash | 1.00 | ✅ PASS 2/2 |
| Gemini 2.0 Flash | 1.00 | ✅ PASS 2/2 |
| Gemma 3 27B | 1.00 | ✅ PASS 2/2 |
| GLM-5 | 1.00 | ✅ PASS 2/2 |
| GPT-5.4 mini | 1.00 | ✅ PASS 2/2 |
| Qwen 3 Next 80B Instruct | 1.00 | ✅ PASS 2/2 |
| DeepSeek-R1 | 0.00 | ❌ FAIL 1/2 |

This has direct safety implications: a model deployed in high-stakes settings (medical, legal, financial) that always claims 100% certainty and never anticipates its own failures provides no internal quality signal for triaging human review. If this blindspot holds across model capability levels, it is a training-level problem, not a prompt engineering fix.

**Failure Profile Taxonomy:**

| Profile | Accurate? | Calibrated? | Error Recall | Risk |
|---------|-----------|-------------|--------------|------|
| Ideal | High | Yes | > 0 | Minimal |
| **Overconfident Correct** ← *all tested models* | High | No | 0.000 | Moderate |
| Calibrated Incorrect | Low | Yes | > 0 | Manageable |
| Overconfident Incorrect | Low | No | 0.000 | Critical |

Standard benchmarks would rate these models very highly. CBB reveals the consistent hidden risk profile across the capability spectrum.

---

## Why No LLM-as-Judge?

All scoring is fully deterministic, no `kbench.judge_llm` used. Using an LLM judge to evaluate metacognition creates a circular dependency: the judge's own metacognitive limitations would contaminate the results. Deterministic regex-based scoring ensures full reproducibility.

---

## Repository Structure

```
calibration-blindspot-benchmark/
├── README.md                          ← Overview and results
├── HOW_TO_USE.md                      ← Practical guide for running CBB
├── cbb-kaggle-notebook.ipynb          ← Full benchmark notebook (Kaggle-ready)
└── CBB_Writeup/
    └── CBB_Final_Writeup_v2_Emmanuel_Fle_Chea.pdf  ← Full research writeup
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
