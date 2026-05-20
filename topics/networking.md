---
title: Datacenter Networking
permalink: /topics/networking/
description: >-
  Notes on datacenter networking for AI and HPC — switches, fabrics, RDMA,
  InfiniBand, RoCE, NCCL, topology design, and profiling collective
  communication.
---

# Datacenter Networking

Notes on the fabrics that move bytes between GPUs — the layer that quietly determines whether your training job converges in days or weeks.

## What this covers

- **Switches and fabrics** — leaf-spine, rail-optimized topologies, dragonfly, fat-tree. How design choices interact with collective comms patterns.
- **RDMA** — InfiniBand and RoCEv2. Verbs, queue pairs, completion queues, and the failure modes that show up under load.
- **NCCL internals** — channel selection, ring vs. tree algorithms, network plugin interface, debugging collective stalls.
- **Congestion control** — DCQCN, HPCC, and why ECN tuning matters when you're pushing line rate over hundreds of links at once.
- **Profiling collectives** — NSight Systems, nccl-tests, custom tracing, eBPF for fabric visibility.
- **Topology-aware placement** — making sure ranks land where the topology expects.

## Why I write about this

Networking is the part of AI clusters most engineers never see and most failures secretly involve. I write to demystify what's actually happening on the wire when an all-reduce stalls.

Browse posts at [/categories/networking/](/blog/categories/networking/), or see the full [topic index](/blog/topics/).
