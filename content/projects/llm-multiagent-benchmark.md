---
title: "LLM Benchmark Evaluation: Multi-Agent Discussion Framework"
date: 2025-05-01
tags: ["LLM", "benchmark", "Gemini", "evaluation", "Python"]
summary: "Rigorous evaluation of a 4-agent discussion pipeline on 7,661 benchmark questions. The framework decreased accuracy on all three benchmarks — an instructive negative result."
---

**Course:** CS 109B (Advanced Data Science), Harvard, Spring 2025

<!-- **Code:** [github.com/nthapaliya/llm-multiagent-benchmark](https://github.com/nthapaliya/llm-multiagent-benchmark) -->

---

## The hypothesis

A 4-agent discussion pipeline — Planner → Answerer → Critic → Moderator — can improve
LLM reasoning accuracy on challenging benchmarks without retraining, by giving the model
structured opportunities to self-correct over up to 5 rounds.

## The pipeline

```
Round 1:
  Planner   → decompose question into a 2-4 step reasoning plan
  Answerer  → follow plan, produce provisional answer
  Critic    → identify errors, or "No further objections"
  Moderator → "FINAL ANSWER: X" or "CONTINUE"

Rounds 2-5 (if CONTINUE):
  Answerer  → revise reasoning incorporating prior critique
  Critic    → re-evaluate
  Moderator → decide
```

Each stage is a separate Gemini 2.0 Flash API call. The evaluation harness runs
all benchmarks asynchronously with a dual token-bucket rate limiter (2,000 RPM / 4M TPM).

## Results

Evaluated on 7,661 questions across three benchmarks:

| Dataset | Baseline | Multi-Agent | Δ |
|---------|---------|------------|---|
| MMLU Pro (5,000 Qs) | 60.0% | 52.7% | **−7.3 pp** |
| GPQA Diamond (161 Qs) | 53.4% | 50.3% | **−3.1 pp** |
| HLE (2,500 Qs) | 3.8% | 2.8% | **−1.0 pp** |

Cost increased 3–5× and latency 3–5× per question with no accuracy gain.

**Why it backfired:** 85% of final answers settled at round 1 — the Critic rarely
improves a correct answer but does occasionally break one. Later rounds amplify errors
rather than correcting them.

## What this tells us

Homogeneous self-critique doesn't work at this scale. More promising directions:
heterogeneous agent pools (mixing models with different strengths), targeted
fact-checking verifiers, and selective invocation only on low-confidence initial answers.

## Stack

Python, Google Gemini API, asyncio (async rate-limited harness), pandas, Matplotlib, uv
