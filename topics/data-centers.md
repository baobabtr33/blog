---
title: Data Centers
permalink: /topics/data-centers/
description: >-
  Notes on data center engineering — site selection, capacity planning, floor
  layout, power and cooling, cabling, transceivers, and the operational
  details behind modern AI and HPC facilities.
---

# Data Centers

Notes on the physical and operational side of data center engineering — the layer below the fabric.

## What this covers

- **Site selection** — power availability, water access, grid resilience, climate, latency to peering hubs.
- **Capacity planning** — sizing power, cooling, and floor space against a workload mix that's increasingly GPU-heavy.
- **Floor planning** — hot/cold aisles, rack pitch, weight loading, service corridors, and what rail-optimized layouts demand from the room.
- **Power and cooling** — power distribution (busway vs. RPP), PDU sizing, cooling at densities that have moved past air, immersion and direct-to-chip liquid cooling.
- **Cabling and transceivers** — fiber types, MPO breakouts, transceiver generations (400G, 800G, 1.6T), cable trays, and the not-glamorous logistics of physical layer at scale.
- **Operations** — change management, hardware lifecycle, RMA throughput, capacity at risk.

## Why I write about this

GPU clusters are physical things, and most of their interesting failure modes start at the power, cooling, or cabling layer long before they reach the kernel. Data center engineering is the foundation everything else rides on — I write to make that foundation more visible.

Browse posts at [/categories/data-centers/](/blog/categories/data-centers/), or see the full [topic index](/blog/topics/).
