---
title: "Context-Aware Image Captioning with Scene Graphs"
date: 2026-05-01
tags: ["computer vision", "PyTorch", "YOLO", "transformers", "NLP"]
summary: "End-to-end image captioning pipeline conditioned on structured scene graph triples."
---

**Course:** CSCI E-25 (Computer Vision), Harvard Extension School, Spring 2026

**Code:** [github.com/nthapaliya/scene-graph-captioning](https://github.com/nthapaliya/scene-graph-captioning)

**Notebook:** [View on Github](https://github.com/nthapaliya/scene-graph-captioning/blob/main/SG_captioning.md)

---

## The question

Do structured scene graphs — explicit `subject → predicate → object` triples extracted
from an image — help a caption decoder produce better descriptions than image features alone?

## The pipeline

Three stages, trained end-to-end:

1. **YOLO object detector** fine-tuned on the Visual Genome VG150 subset (150 object classes, 50 predicates)
2. **Visual predicate classifier** — ResNet-50 ROI features over subject + object bounding boxes,
   concatenated with class embeddings and a spatial vector, fed to a 3-layer MLP
3. **ViT + T5 caption decoder** — ViT-base/16 image encoder and a T5-small scene-graph encoder,
   concatenated and decoded with cross-attention

The ablation baseline removes the scene graph input (`sg_input_ids = None`), keeping everything else identical.

## Results

Evaluated on a 9,180-image held-out validation set (beam-4 decoding):

| Metric | Image-only | Scene-graph | Δ |
|--------|-----------|-------------|---|
| BLEU-4 | 0.0597 | **0.0665** | +11.4% |
| METEOR | 0.4319 | **0.4474** | +3.6% |
| ROUGE-L | 0.2857 | **0.2924** | +2.3% |

The predicate classifier reached **top-5 accuracy of 89.8%** on the VG150 validation split.

Scene-graph conditioning consistently improves all metrics. BLEU-4 sees the largest relative
lift, suggesting the model commits to specific multi-word phrases ("riding a skateboard") rather
than generic templates. The dominant failure mode is upstream detector error.

## Dataset

[Visual Genome](https://homes.cs.washington.edu/~ranjay/visualgenome/index.html) VG150 subset
(91,804 images). Captions sourced from MS-COCO for images with a `coco_id` link; Visual
Genome region descriptions as fallback.

## Stack

PyTorch, YOLO (Ultralytics), from-scratch Transformer decoder, NLTK (BLEU), rouge-score, h5py, uv
