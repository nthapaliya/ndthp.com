---
title: "CNN Image Super-Resolution (4× Upscaling)"
date: 2024-12-02
tags: ["deep learning", "computer vision", "CNN", "TensorFlow", "Keras"]
summary: "Comparison of three CNN architectures for 4× single-image super-resolution on the FFHQ dataset. Evaluated with PSNR and SSIM."
---

**Course:** CSCI E-89 (Deep Learning), Harvard Extension School, Fall 2024

**Code:** [github.com/nthapaliya/cnn-image-upscaling](https://github.com/nthapaliya/cnn-image-upscaling)

---

## Overview

Single-image super-resolution (SISR) is the task of recovering a plausible high-resolution
image from a low-resolution input. This project trains and compares three CNN architectures
at 4× upscaling on the [FFHQ dataset](https://github.com/NVlabs/ffhq-dataset) (70,000
high-quality face images), measuring output quality with PSNR and SSIM.

Faces provide a structured benchmark domain where quality degradation is perceptually
obvious and metrics are well-calibrated.

## Architectures

| Model | Key idea |
|-------|---------|
| **SRCNN** | Pioneering 3-layer super-resolution CNN (Dong et al., 2014) |
| **ESPCN** | Sub-pixel convolution (pixel shuffle) for efficient upscaling (Shi et al., 2016) |
| **EDSR** | Removes batch norm for more stable training at depth (Lim et al., 2017) |

## Evaluation

Both metrics computed on the luminance channel (Y of YCbCr), matching standard practice:
- **PSNR** — Peak Signal-to-Noise Ratio (higher is better, measured in dB)
- **SSIM** — Structural Similarity Index (higher is better, 0–1)

## Dataset

[FFHQ](https://github.com/NVlabs/ffhq-dataset) — 70,000 high-quality PNG face images at
1024×1024. Downloaded via Kaggle. Low-resolution training inputs created by bicubic
downsampling (4× reduction). 65,000 train / 5,000 test split.

## Stack

TensorFlow 2.x, Keras, NumPy, Matplotlib, Kaggle API, uv
