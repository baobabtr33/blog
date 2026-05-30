---
layout: page
title: AI Infrastructure
permalink: /topics/ai-infrastructure/
description: >-
  Notes on AI infrastructure — training systems, inference serving,
  schedulers, storage, observability, and fault tolerance for large-scale
  machine learning.
---


The plumbing that lets large models train and serve at scale — schedulers, storage, networking, runtime, observability.

## What this covers

- **Training systems** — frameworks (PyTorch, JAX, Megatron-LM, DeepSpeed), checkpointing, fault tolerance, and the operational realities of multi-thousand-GPU runs.
- **Inference serving** — KV cache management, batching, speculative decoding, serving stacks (vLLM, TensorRT-LLM, SGLang).
- **Schedulers** — Slurm, Kubernetes, Ray, gang scheduling. Why preemption and queueing matter as much as raw compute.
- **Storage for ML** — parallel filesystems (Lustre, GPFS, WekaFS), object storage tiering, data loading pipelines.
- **Observability** — metrics, traces, and tools for understanding what a training job is actually doing (and why it's stuck).
- **Cost and capacity** — utilization, queueing theory, when to buy vs. burst.

## Why I write about this

The interesting bottlenecks in AI infra rarely live where you expect — they hide in collective comms, storage backpressure, scheduler quirks, or thermal limits. I write up what I learn debugging real systems.

Browse posts at [/categories/ai-infrastructure/](/blog/categories/ai-infrastructure/), or see the full [topic index](/blog/topics/).
