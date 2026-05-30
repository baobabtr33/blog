---
layout: page
title: High-Performance Computing (HPC)
permalink: /topics/hpc/
description: >-
  Engineering notes on high-performance computing — distributed training,
  GPU clusters, collective communication (NCCL, MPI), parallel programming,
  and HPC system design.
---


Notes from the HPC side of my work — the systems that scale machine learning and scientific computing to large GPU clusters.

## What this covers

- **Distributed training** — data parallelism, tensor parallelism, pipeline parallelism, and the trade-offs between them at scale.
- **Collective communication** — NCCL, MPI, custom collectives. All-reduce, all-gather, reduce-scatter — how they behave on real hardware, how to debug them when they don't.
- **GPU clusters** — H100, B200, GB200 generations. Node-level topology (NVLink, NVSwitch) and rack-level fabrics.
- **Parallel programming** — CUDA, OpenMP, MPI patterns that hold up at thousands of GPUs.
- **HPC tooling** — Slurm, container runtimes, environment modules, build systems for cluster software.

## Why I write about this

HPC has been quietly underwriting AI's recent jumps. Most of the interesting failure modes in modern ML training are HPC problems — communication stalls, NUMA misalignment, fabric congestion, thermal throttling — wearing ML clothes. I write to make that link explicit and to keep my own debugging notes accessible.

Browse posts by category at [/categories/hpc/](/blog/categories/hpc/), or see the full [topic index](/blog/topics/).
