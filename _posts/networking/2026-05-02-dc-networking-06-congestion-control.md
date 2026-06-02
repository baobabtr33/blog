---
title: "Data Center Networking: Congestion Control"
date: 2026-05-02 11:00:00 -0500
categories: [Networking]
tags: [networking, congestion-control, pfc, ecn, dcqcn, roce, rdma, gpu, telemetry, series]
permalink: /posts/dc-networking-06-congestion-control/
description: >-
  How congestion control works across the stack in a data center, from L2 PFC
  and L3 ECN to TCP and the RDMA NIC. Why it matters for GPU training, how CNP
  and DCQCN run in hardware, and what telemetry like out-of-sequence counters
  reveals.
---

Congestion control is the set of mechanisms that prevent network buffers from overflowing and determine what happens when they do. It is one of those topics that spans all layers of the stack simultaneously: L2 pause frames, L3 packet marking, L4 transport algorithms. Each layer has its own mechanism, and they interact in ways that matter for DC performance, particularly for AI/GPU workloads.

## Why congestion happens in data centers

A DC switch has input ports and output ports. When multiple input ports try to forward traffic to the same output port simultaneously, the output port becomes a bottleneck. Packets queue in the output buffer. If the queue fills, packets are dropped.

This is especially acute in two patterns.

The first is many-to-one, or incast, where N servers all send data to a single server at the same time. This happens in distributed storage, where a read request fans out to N replicas that all respond at once, and in AI training, where a gradient aggregation step has N GPUs all sending gradients to a single reducer. The receiving server's uplink becomes momentarily overloaded.

The second is all-to-all, where N GPUs all need to exchange data with every other GPU. In a collective operation like AllReduce or AllGather, every GPU is at once a sender and a receiver, and the fabric has to handle N squared communication patterns without congestion dominating the operation time.

## L2: PFC (Priority Flow Control)

PFC is defined in IEEE 802.1Qbb. It extends the original Ethernet PAUSE mechanism to be per-priority-class rather than per-link.

When a switch port's ingress buffer for a specific priority fills to a threshold, the switch sends a PAUSE frame to the upstream device, instructing it to stop sending traffic in that priority class. The upstream device buffers the paused traffic. When the buffer drains below the resume threshold, the switch sends a RESUME.

Why per-priority: the original Ethernet PAUSE paused all traffic on the link. A single congested flow would stop management traffic, routing protocol traffic, everything. PFC separates traffic classes (typically 8, corresponding to 802.1p CoS values 0-7) so you can pause one priority without affecting others.

For RoCE v2 (RDMA over Converged Ethernet), PFC is used to create a lossless class for RDMA traffic. RDMA is designed for lossless networks (it comes from InfiniBand) and performs poorly with packet loss. PFC backpressures senders rather than dropping packets.

The problem with PFC: it propagates congestion upstream. A pause frame stops traffic from the upstream device, which may then fill its own buffer, causing it to pause its upstream, and so on. This is called a pause storm or PFC deadlock. In a spine-leaf fabric, a congestion event at one leaf can propagate pauses all the way through the spine to other leaves, causing collateral damage to unrelated traffic flows.

Mitigation: PFC watchdog timers detect links that have been paused for too long and take action (bring the port down, generate an alert). Careful fabric design limits the propagation distance. Buffer tuning at each switch controls the headroom between the PFC trigger threshold and actual buffer overflow.

PFC is necessary for RoCE in DC environments but requires careful operational tuning. It is not a mechanism you deploy and walk away from.

## L3: ECN (Explicit Congestion Notification)

ECN is a more surgical signal. Instead of pausing the sender, it marks packets to tell the receiver that congestion is happening, and the receiver signals the sender to reduce its rate.

ECN uses two bits in the IP header (the ECN field) to signal congestion state:
- 00: Not ECN-capable
- 10 or 01: ECN-capable transport (sender has opted in)
- 11: Congestion experienced (CE: set by a router when it detects congestion)

When a router's queue depth crosses a threshold (typically using RED, Random Early Detection), instead of dropping the packet, it sets the CE bits (11) on ECN-capable packets. The receiving end sees the CE mark and sends an ECN echo back to the sender. The sender reduces its transmission rate.

The key difference from PFC: ECN works proactively before the buffer fills. PFC is reactive (pause when the buffer is nearly full). ECN marks packets at moderate queue depth, giving senders time to reduce rate before the queue overflows and drops occur.

In DC fabrics with RoCE, ECN is used alongside PFC. ECN handles the common case: senders back off when they detect congestion, queues drain, no PFC needed. PFC is the backstop for when ECN is not sufficient (e.g., sudden large bursts).

DCQCN, Data Center Quantized Congestion Notification, is the algorithm used for RoCE congestion control. It was developed by Microsoft and published in 2015, and it uses both ECN at the switch and PFC as a last resort. The sender (RDMA NIC) responds to ECN marks by reducing its rate multiplicatively. It then probes by increasing the rate additively. This is similar to TCP's AIMD (Additive Increase, Multiplicative Decrease) but implemented in hardware in the RDMA NIC and responsive in microseconds rather than milliseconds.

### Congestion signaling and control in the NIC

The ECN loop for RoCE runs largely in the NIC. When a receiver NIC gets a packet whose CE bits are set, it generates a Congestion Notification Packet (CNP) and sends it back to the sender NIC, which then cuts its rate. None of this waits for software. The marking, the CNP, and the rate response all happen in hardware and firmware on the adapter, which is what lets RoCE react in microseconds.

Recent NVIDIA adapters push this further. The time-stamped RTT probes and the congestion telemetry that feed the algorithm run in hardware and firmware on ConnectX-6 Dx and later, ConnectX-7, ConnectX-8, and the BlueField-3 DPUs and SuperNICs. On top of that, Programmable Congestion Control lets you write your own congestion control algorithm and run it on the NIC rather than living with the vendor default, starting with the ConnectX-6 Dx. NVIDIA documents this under [DOCA Programmable Congestion Control](https://docs.nvidia.com/doca/sdk/doca-telemetry-pcc/index.html).

## L4: TCP Congestion Control

TCP has built-in congestion control at the transport layer. This is independent of what happens at L2 and L3, though they interact.

### The original model: TCP Reno

I first worked through this AIMD model and the TCP throughput math in the Georgia Tech OMSCS Computer Networks course. Their notes on [the TCP protocol and throughput](https://www.omscs-notes.com/computer-networks/transport-and-application-layers/#the-tcp-protocol-tcp-throughput) and on [TCP CUBIC](https://www.omscs-notes.com/computer-networks/transport-and-application-layers/#congestion-control-in-modern-network-environments-tcp-cubic) are a clean reference if you want the derivations.

TCP Reno (and its predecessor Tahoe) established the standard AIMD algorithm:

**Slow start.** When a connection opens, the congestion window (cwnd, the amount of unacknowledged data allowed in flight) starts at 1 MSS (maximum segment size, typically 1460 bytes) and doubles with each ACK until it hits ssthresh (slow start threshold) or detects congestion. Despite the name, slow start grows exponentially.

**Congestion avoidance.** Above ssthresh, cwnd grows by 1 MSS per RTT, which is linear. This is the additive increase in AIMD.

**Congestion detection on packet loss.** When a packet is lost, through triple duplicate ACKs or a timeout, TCP assumes the network is congested. ssthresh is set to cwnd/2 and cwnd is reduced, cut in half for a triple dup-ACK in Reno and all the way to 1 for a timeout. This is the multiplicative decrease in AIMD.

**Fast retransmit.** When the sender receives three duplicate ACKs, where the receiver keeps ACKing the last in-order segment to signal a gap, it retransmits the missing segment without waiting for the timeout.

**Fast recovery.** After fast retransmit in Reno, instead of going back to slow start, TCP enters congestion avoidance at the new halved ssthresh, which is faster than timeout recovery.

Reno's limitation: it treats all packet loss as congestion. In noisy wireless networks, packet loss from radio errors triggers AIMD, reducing throughput unnecessarily. It also fills buffers to capacity (buffer bloat) before backing off, adding significant latency on saturated links.

### CUBIC (Linux default)

CUBIC is the default TCP congestion control in Linux. It uses a cubic function to grow cwnd rather than the linear growth of Reno's congestion avoidance phase. This allows faster recovery after congestion and better throughput on high-bandwidth, long-delay links.

CUBIC is the right choice for most general-purpose DC workloads.

### BBR (Bottleneck Bandwidth and RTT)

Google's BBR takes a different approach. Instead of using packet loss as the congestion signal, BBR models the network path: it estimates the bottleneck bandwidth and the minimum RTT, then sets the sending rate to match the bandwidth without overfilling the bottleneck buffer.

BBR does not fill buffers. It targets a sending rate based on measured bandwidth, not buffer capacity. This reduces queueing latency significantly. On Google's internal and public networks, BBR reduced latency by 15-40% and increased throughput on lossy connections (compared to CUBIC on the same path).

BBR is now widely deployed. It is available in Linux, and is used by YouTube, Google Search, and other high-volume Google services. For latency-sensitive workloads or deployments over lossy WAN paths, BBR is worth evaluating.

## AI workloads and congestion

AI training workloads create congestion patterns that are different from web/application traffic.

Why congestion control matters so much for GPUs comes down to cost and synchronization. GPU training is bulk synchronous, so every GPU in a collective waits for the slowest one before the next step can start. A single flow stuck behind a congested queue stalls the whole collective, and the entire fleet of accelerators sits idle while one packet waits. At 400 and 800 Gbps per port, even a few microseconds of queue buildup turn into real lost compute on hardware that costs a fortune per hour. Congestion control is what keeps the expensive part of the cluster, the GPUs, from waiting on the network.

The collective communications are the heart of it. Operations like AllReduce, used in data parallel training to average gradients across GPUs, require every GPU to send and receive at the same time, orchestrated by NCCL, the NVIDIA Collective Communications Library. A poorly tuned fabric with congestion in the collective path makes training throughput collapse.

Incast shows up here too. During a reduce operation, N GPUs all send data to M aggregators at once, so the aggregators' uplinks are suddenly hit with N simultaneous flows. Even brief queue buildups cause tail latency that blocks the GPU and prevents it from starting the next compute step.

There is also a flow distribution problem. Modern AI training with hundreds or thousands of GPUs creates millions of concurrent flows, and ECMP (Equal-Cost Multi-Path) hashing spreads them across parallel paths. When the hashing collides and many flows land on the same path, some paths are overloaded while others sit idle. This is a flow distribution problem, not a total bandwidth problem.

Solutions being deployed: per-packet load balancing (CONGA, DRILL) rather than per-flow ECMP, RDMA with DCQCN for lossless transport, telemetry-based congestion visibility at microsecond granularity, and custom transport protocols developed by hyperscalers (Google's SWIFT, Meta's RDMA implementations).

## Telemetry and out-of-sequence counters

You cannot tune what you cannot see, and at these speeds the only way to see congestion is hardware telemetry. Meta described building telemetry systems that automatically collect RDMA hardware counters across the network switch, the NIC, and the PCIe path to examine what is happening behind a training run. One of the most useful counters they call out is Out-of-Sequence, the number of packets the NIC perceives as arriving out of order. A rising OOS count exposed packet drops caused by unhealthy switches and NIC hardware bugs that nothing else in the stack reported, drops that would otherwise have shown up only as mysteriously slow training. This is described in their [SIGCOMM 2024 paper on RDMA over Ethernet for distributed training at Meta scale](https://cs.stanford.edu/~keithw/sigcomm2024/sigcomm24-final246-acmpaginated.pdf).

## The interaction

PFC, ECN, and TCP/RDMA congestion control are not independent. They interact:

1. Congestion builds at a switch
2. ECN marks packets, RDMA senders (DCQCN) begin to rate limit
3. If ECN is not enough and buffers continue to fill, PFC kicks in and pauses the sender's priority class
4. When PFC is active, ECN is irrelevant (traffic is paused, not flowing)
5. PFC propagates upstream if the paused device's own buffers fill

For TCP traffic: PFC pauses the sender, TCP does not get ACKs (no segments in flight), TCP RTO timer may fire, TCP retransmits and enters slow start on resume. PFC pauses degrade TCP performance significantly.

This is why RoCE fabrics go to great lengths to prevent PFC from firing: every PFC pause is a problem for TCP traffic sharing the same physical fabric. The correct model is to use QoS to separate RoCE/RDMA traffic (lossless class with ECN + PFC as backstop) from TCP traffic (standard class with standard TCP congestion control, no PFC).

The fabric's design goal: keep PFC as a last-resort safety net, use ECN as the primary congestion signal, and use QoS marking to prevent TCP and RDMA traffic from competing for the same queues.

## What working on this taught me

The thing that stuck with me from working on these fabrics hands on is that congestion control is not one mechanism in one place. It runs at several layers of the stack at once, L2 pause, L3 marking, and L4 and NIC transport, and it runs across several different devices, the server NIC, the switch, and back again. No single device holds the whole picture. That is why telemetry and an end-to-end view matter so much. You have to follow the path from server to switch and from switch back to server to understand where a stall actually came from. In a distributed system that is trying to keep high bandwidth links full, that end-to-end visibility is the difference between a fabric you can tune and one that quietly underperforms for reasons nobody can name.
