---
doc_id: mmd-072-2026-artex
chunk_id: 03b
title: "3B. Case Review — South Korean Financial Sector (late Sep – early Oct 2026)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [south-korea, financial-sector, case-study, kill-chain]
---

## 3B. Case Review — South Korean Financial Sector (late Sep – early Oct 2026)

First widely reported real-world ARTEX abuse. Primary source: CrowdStrike (7 Oct 2026). Victim names and counts come from Korean press and regulator notices, not from CrowdStrike. Actor: unknown, likely Chinese-speaking, financially motivated, probably a single operator (CrowdStrike, moderate confidence). Not attributed to Storm-0940 or any named group.

### 3B.1 Kill chain and where each phase is visible

| Phase | What happened | Visible to | Playbook effect | 
|---|---|---|---|
| Operator set-up | ARTEX hosted on 38.244.50.120 (with open directory incl. CLAUDE.md); Hong Kong primary server (IP not published); DeepSeek v4.1-flash via xcai.pro; GLM-5.3 and Grok 4.6 in Claude Code sessions. | Hosting providers, internet scanners; egress logs if inside your network | 38.244.50.120 and xcai.pro stay T3. Operator workspace folders become an osquery/hunting item. | 
| Target choice | Peripheral internet-facing systems: loan-progress inquiry service for brokers, employee mobile work-support app. Core banking not targeted. | Victim web/API logs | B1/B2 scope must include low-profile partner and employee portals, not only flagship apps. | 
| Access | Authentication/access-control gaps exploited; valid records enumerated. | Victim web/API logs | B2 gains an authorization-enumeration branch. | 
| Collection | One service: \~25,000 records exfiltrated over \~30 hours (\~14 records/min). | Victim web/API, DB audit | Low-and-slow: volume thresholds would miss it. Use distinct-object counts per identity/session over long windows. | 
| Proxying | Proxy IPs incl. 101.53.80.20, 205.214.59.31, 124.155.252.63, 154.201.79.246, 23.248.249.90 and others. | Victim edge logs | Proxy IPs T3 with ‘shared proxy’ note; never auto-block on these alone. | 
| Monetization | Claude sessions asked about markets and Telegram groups for selling Korean breach data. | Not visible to defenders | Context only. Personal details in the sessions are excluded (Section 9). | 

### 3B.2 Lessons applied to the playbook

1. **Count distinct objects, not requests.** B2 adds a
rule branch for one identity or session reading an unusually high number
of distinct record IDs (customer, loan, application numbers) over hours,
with no rate condition.
2. **Peripheral systems are first in line.** Rule
documentation tells users to deploy B1/B2 on broker, partner,
employee-mobile and inquiry APIs first.
3. **One host, two campaigns.** 38.244.50.120 appears in
both the Korean campaign and the JPCERT/CC list. It remains T3 on its
own, but any B-rule hit from it is raised to T1.
4. **Keep campaigns separate in STIX.** Shared tooling and
one shared IP do not establish a shared operator. The Korean activity is
modeled as its own unattributed intrusion set.
