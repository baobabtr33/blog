---
title: "Data Center Planning: Site Location and High Availability"
date: 2026-04-21 10:00:00 -0500
categories: [Data Centers]
tags: [data-centers, infrastructure, planning, site-selection, high-availability, series]
permalink: /posts/dc-planning-02-site-location/
description: >-
  How geography locks in latency, cost, and risk for a decade. The 3ms / 300km
  rule for synchronous replication, active-active vs active-passive trade-offs,
  power economics, disaster exposure, data residency, and carrier-neutral
  connectivity.
---

Previously, [what a data center actually is](/blog/posts/dc-planning-01-intro/).

Picking a data center location feels like a real estate problem. It is partly that. But the decisions you make about geography lock you into latency, cost, and risk profiles for the next decade. Getting it wrong is expensive to unwind.

There are five constraints that drive site selection. Latency between sites. Power cost and availability. Natural disaster exposure. Data residency and legal jurisdiction. Network connectivity. They all pull in different directions.

## The 3ms rule for synchronous replication

If you care about zero data loss (RPO of zero), you need synchronous replication. The database writes a transaction, and that transaction must be confirmed on a second site before the write is acknowledged to the application. The application waits for both sites to confirm.

The constraint is physics. Light travels through fiber at roughly 200,000 km per second (the refractive index of glass slows it down from the vacuum speed). A 3ms round-trip time gives you 1.5ms one way. At 200 km per ms, that is about 300 km of fiber distance.

300 km is roughly the maximum separation between your primary and secondary sites if you want synchronous replication at 3ms RTT.

Why 3ms specifically? It is a practical number. Most synchronous replication systems and database clusters can tolerate 1 to 3 ms without significant application-level performance impact. Beyond 3ms, write latency compounds enough to affect user-facing response times on write-heavy workloads. Some systems push to 5ms, but 3ms is the conservative design target.

At 3ms, your site options are already constrained. If your primary is in Chicago, your secondary can be in Indianapolis or Milwaukee, not Miami or Seattle. This is why cloud providers announce that AZs are "within 60 miles" of each other. The 60-mile number corresponds roughly to the sub-3ms RTT constraint.

For asynchronous replication, where some data loss is acceptable, distance is not constrained. Your DR site can be across the country or on another continent. But async means if the primary site burns down before the last replication cycle completes, you lose whatever had not replicated yet. The RPO is hours, minutes, or seconds depending on how often you replicate, not zero.

The decision falls out cleanly. Active-active with synchronous replication requires geographic proximity. True DR (active-passive with async) can sit anywhere.

## Active-active vs active-passive

Active-active runs production traffic on both sites at the same time. Traffic is split between them. If one site fails, the other absorbs the full load, which is why each site typically runs at 50 to 60 percent capacity rather than 100. Synchronous replication keeps both sides current. Failover is automatic and fast.

Active-passive has one site running everything and the other sitting as a standby. The standby may be warm (powered on, software running, data replicated) or cold (infrastructure ready but not running). Failover is manual or semi-automated. It is cheaper to operate because the standby is not serving live traffic, so you are not paying for double the compute capacity at normal utilization.

Most enterprises run active-passive for cost reasons and accept the failover time (RTO). Financial services and large consumer platforms tend to run active-active because downtime at their scale is more expensive than the extra hardware.

There is also active-active-active. Three sites, each serving a third of traffic, with majority quorum for decisions. This eliminates split-brain scenarios and tolerates one full site loss without degradation. It is expensive and complex to build, but some platforms require it.

## Power, cost, and availability

Power is typically 40 to 60 percent of a data center's operating cost. The electricity rate per kWh varies significantly by location.

The US Pacific Northwest (Oregon, Washington) has historically cheap hydroelectric power, which is most of why Google, Amazon, and Microsoft put major facilities there. Rates can be under $0.03/kWh for industrial users. Compare that to parts of the Northeast or California where industrial rates can run $0.08 to $0.15 per kWh. At 10 megawatts of IT load running 24/7 for a year, that price difference is millions of dollars annually.

Iceland and the Nordic countries get renewable geothermal and hydro power at low cost with naturally cold ambient temperatures, which gives you free cooling for much of the year. This is why hyperscalers have built there.

Nuclear power provides stable, high-density electricity without weather dependency. Some facilities negotiate directly with nuclear utilities for baseload contracts.

Power availability matters as much as cost. A site with cheap power that only has one utility feed from one substation is a reliability problem. Good sites have multiple utility feeds from geographically separate substations, ideally from different transmission lines.

## Natural disaster exposure

You do not want your primary and secondary sites to share a failure mode.

Earthquake fault lines are the first thing to map. The Pacific Coast (Cascadia subduction zone) affects the Pacific Northwest. The New Madrid Seismic Zone runs through the central US. Putting both sites in Seattle is not HA design.

Flooding deserves its own check, both from river flood plains and from coastal flood zones. FEMA flood maps are a useful starting point.

Hurricane paths are the next concern. Gulf Coast and Atlantic Coast sites need to be designed for wind loads and have flood mitigation built in.

Tornado Alley sits in the central US (Kansas, Oklahoma, Texas) and sees frequent severe weather. A tornado can destroy a facility. Most data centers in these areas are hardened with reinforced concrete and no above-ground windows in the critical areas, but a direct hit is a direct hit.

The practical approach is to put sites in different natural disaster regions. Oregon (earthquake, mild weather) paired with Northern Virginia (hurricane risk but different fault profile) is a common hyperscaler choice because the failure modes are largely uncorrelated.

## Data residency and legal jurisdiction

This is increasingly a hard constraint. GDPR requires EU citizen data to stay in the EU with specific exceptions. China has strict data localization requirements. Some industries (government, healthcare, finance) have regulatory rules that specify which countries or regions data can reside in.

If you operate globally, you often do not get to choose the optimal location. You build where your users are and where regulations allow, then design around those constraints.

Jurisdiction also affects law enforcement access. Data in the US is subject to US legal process. Data in Germany is subject to German legal process. Sovereignty concerns have driven some organizations to build in specific locations specifically to limit which governments can compel access.

Finance has a clean rule of thumb for handling conflicting regimes. When jurisdictions disagree, you design to the strictest one. The CFA Institute's first standard, knowledge of the law, says that when applicable laws or regulations conflict, you comply with the more strict requirement. Applied to a data center build, you do not try to thread the needle between regimes. You assume the tightest residency and access rule applies everywhere your data might land and you architect to satisfy that, because meeting the strictest rule by definition satisfies the looser ones.

## Network connectivity and carrier neutrality

You want your data center to connect to multiple network carriers. If your facility only has one carrier and that carrier has an outage, your entire DC goes offline from a network perspective even if all the hardware is running fine.

Carrier-neutral co-location facilities like Equinix, Digital Realty, and CoreSite host equipment from dozens of network providers. You can bring in multiple carriers and connect to them on the same cage floor. You can also peer directly with other companies at these locations without traffic going over the public internet.

If you are building a private facility, you have to negotiate with multiple carriers for physical fiber runs into the building. This is called a diverse entry, where fiber enters from geographically separate paths so that one construction crew cutting a duct does not sever all your connectivity. The fiber should enter the building from different sides through different conduits, tracing different physical routes back to the carrier's POP.

## Putting it together

There is no perfect site. You are balancing a tradeoff surface that pulls in opposite directions on every axis.

- Close enough for low-latency sync replication vs. far enough to survive a regional failure
- Cheap power vs. network connectivity close to your users
- Low disaster risk vs. regulatory compliance with where you have to be
- Carrier-neutral flexibility vs. cost of building in a major metro

Most practical site selection ends up filtering on regulatory requirements first, then power cost, then connectivity, then optimizing for distance between primary and secondary within those constraints.

The 3ms / 300km rule is the stake in the ground. Everything else negotiates around it.

## Sources

- [Calculating optical fiber latency](https://www.m2optics.com/blog/bid/70587/calculating-optical-fiber-latency), for the roughly 200,000 km per second figure and the per-kilometer latency math behind the 3ms rule.
- [The speed of light as the ultimate limit to network latency](https://notes.suhaib.in/docs/tech/physics/how-the-speed-of-light-bounds-network-latency/), for why this is a physics constraint rather than an engineering one.
- [CFA Institute Standard I(A), Knowledge of the Law](https://www.cfainstitute.org/standards/professionals/code-ethics-standards/standards-of-practice-i-a), for the comply-with-the-stricter-rule convention applied to conflicting jurisdictions.

Next, [capacity planning](/blog/posts/dc-planning-03-capacity-planning/).
