---
title: "QLoRA Fine-Tuning for Structured JSON Extraction"
date: 2026-05-01
tags: ["LLM", "fine-tuning", "QLoRA", "NLP", "PyTorch", "Hugging Face"]
summary: "Fine-tuning a 3B instruct model with QLoRA to produce strict JSON from free-form text — on a single consumer GPU. Exact-match accuracy 0.258 → 0.624."
---

**Course:** CSCI E-222 (Foundations of Large Language Models), Harvard Extension School, Spring 2026

**Code:** [github.com/nthapaliya/qlora-food-extraction](https://github.com/nthapaliya/qlora-food-extraction)

---

## The task

Given an unstructured English caption of a food or drink scene, produce a valid JSON object
with exactly four keys:

```json
{
  "is_food_or_drink": true,
  "tags": ["fi", "di"],
  "food_items": ["cauliflower florets", "sweet potato wedges"],
  "drink_items": ["white wine"]
}
```

The contract is tight: a missing closing brace or wrong tag code counts as a miss.

## The approach

**QLoRA** freezes the base model in 4-bit NF4 quantization and trains small low-rank
adapter matrices on top in higher precision. Only the adapters are updated — 29.9M of
3.12B parameters (0.96%).

- **Base model:** `Qwen/Qwen2.5-3B-Instruct`
- **Quantization:** 4-bit NF4, double quantization, bfloat16 compute
- **LoRA rank:** 16, applied to all linear layers, dropout 0.05
- **Loss masking:** completion-only (prompt tokens masked with `-100`)
- **Hardware:** RTX 3060 Ti, 8 GB VRAM — peak VRAM well under 8 GB

## Results

Evaluated on a 213-row held-out test split. A malformed output scores zero on every downstream metric.

| Metric | Base model | Fine-tuned | Δ |
|--------|-----------|-----------|---|
| `json_valid` | 0.972 | **1.000** | +0.028 |
| `schema_ok` | 0.925 | **1.000** | +0.075 |
| `tags_f1` | 0.305 | **0.943** | +0.638 |
| `exact_match` | 0.258 | **0.624** | +0.366 |

The base model's biggest failure — generating free-form English tags instead of the dataset's
short two-letter codes — is exactly what a fine-tuned adapter fixes, even from fewer than
1,000 training rows.

## Dataset

[`mrdbourke/FoodExtract-1k`](https://huggingface.co/datasets/mrdbourke/FoodExtract-1k) (1,420 rows,
50/50 food vs non-food). The label column stores a Python `repr` dict, not JSON —
`ast.literal_eval` required.

## Stack

PyTorch, Hugging Face (transformers, PEFT, bitsandbytes, datasets), Qwen2.5-3B-Instruct, uv
