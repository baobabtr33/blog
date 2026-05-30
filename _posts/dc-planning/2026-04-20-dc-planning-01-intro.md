---
title: "Data Center Planning 1: What a Data Center Actually Is"
date: 2026-04-20 10:00:00 -0500
categories: [Data Centers]
tags: [data-centers, infrastructure, planning, series]
permalink: /posts/dc-planning-01-intro/
description: >-
  How I got into data center engineering, and a working definition of what a
  data center actually is. The physical layer that everything in the cloud
  rides on.
---

I first became interested in Data Centers during my undergraduate years. While I was trying to learn about NLP, I heavily relied on Google's Colab and Google's infrastructure. I always wondered how these GPUs were allocated to me and how it was always available to me (even though it was free!). Even when I was trying to host a simple webserver on EC2, I always wondered how a server in the data center was virtualized so that even a user like me, that needs a 4GB RAM and 1v CPU to host a side project could actually get the server from AWS.

After graduation, I decided to dig deeper into the realm of cloud computing. I decided to join Hyundai Motor Company, which was just building its private data center. Although there were bigger conglomerates that already had private data centers, and other tech companies that already were providing public cloud services in Korea, I wanted to start in a place where I could build from the ground up. Where I could truly be hands-on and learn the whole process of building a cloud infrastructure.

Luckily, I have achieved what I was looking for. Starting out as a cloud network engineer, I was able to become part of planning and building a data center, and also setting up the monitoring infrastructure, and take responsibility in the network that ties all the servers and storage together.

For this topic, I will focus on the lessons learned and the myriad of considerations needed to build a redundant, scalable, efficient data center from the ground up. This will be more about the physical layer of the cloud, rather than the software associated.

## What a data center actually is

Before going into the planning details, it helps to set the picture. A data center, at the most physical level, is a room (usually a very large one) that exists to keep computers running reliably. Everything else is in service of that one goal.

The room sits on land that has cheap, abundant, reliable power, and enough water or air to carry heat away. Inside, you find a few main pieces.

- Rows of racks holding servers, switches, and storage.
- Power distribution that brings megawatts from the utility feed down to each rack, with backup generators and battery banks for when the grid drops.
- Cooling, which means CRAH units, chilled water loops, hot-aisle/cold-aisle layouts, and increasingly direct-to-chip liquid loops for the GPU racks.
- A network fabric of switches connected in a topology that lets any server talk to any other with predictable latency.
- Cabling, both fiber and copper, routed through overhead trays or under raised floors, and terminated at structured panels.
- Security and operations, covering physical access control, fire suppression, monitoring, and on-site staff.

When we use a "cloud" service, this is the thing we are using. The instance is a slice of one of those servers. The storage is one of those disks in one of those racks. The network path from your browser to that VM crossed several of those switches.

## Why the physical layer matters

It's tempting to treat the data center as a solved problem, a commodity you rent. For most users, it is. But every decision at this layer locks in capacity, cost, and reliability for years. Power density is set when the building is poured. Cable pathways are set when the floor is built. Cooling capacity is set when the chillers are installed. Software has very little ability to walk those decisions back.

That's why the people designing the room think about it the way an architect thinks about a building. They work in decades, not sprints.

## What this series covers

For the rest of this series I'll walk through the layers of decisions that go into building one of these rooms from the ground up.

1. Site location, and what you actually look for in land and power.
2. Capacity planning, sizing for today against the workloads you don't yet have.
3. Floor planning, covering racks, aisles, weight loading, and service corridors.
4. Cables, including pathways, lengths, polarity, and slack management.
5. Transceivers, picking modules at 100G, 400G, 800G and the trade-offs that matter.

None of this is glamorous. But it's the foundation everything else rides on, and getting it right is most of why a data center ends up reliable, dense, and economical (or doesn't).

Next, [site location](/blog/posts/dc-planning-02-site-location/).
