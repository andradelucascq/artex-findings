---
doc_id: mmd-072-2026-artex
chunk_id: 06
title: "6. Tier Assignment and Alert Wording"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [tiers, t1-t4, alerting]
---

## 6. Tier Assignment and Alert Wording

| Tier | Eligibility | Severity | Alert title and mandatory text | 
|---|---|---|---|
| T1 | Class A or B and V1/V2 | critical / high | \[ARTEX-T1 CONFIRMED\] \<indicator>. Text: "Indicator unique to ARTEX or a known ARTEX operator. Treat as likely compromise; start incident response." | 
| T2 | Class F at V1/V2, or Class C with a second confirming signal | high | \[ARTEX-T2 BEHAVIOR\] \<pattern>. Text: "Sequence consistent with automated ARTEX activity. Validate source and scope before containment." | 
| T3 | Class E at V1/V2 (historical incident infrastructure) | medium | \[ARTEX-T3 PAST-INCIDENT IOC\] \<value> (\<source>, first seen \<date>). Text: "Infrastructure from a published incident; may be reassigned. Correlate with T1/T2 before blocking." | 
| T4 | Class D at any level; any class at V3; abuse segments; typo variants | low / informational | \[ARTEX-T4 GENERIC – WARNING\] \<value>. Text: "WARNING: generic agent or unconfirmed indicator. Also produced by legitimate tools and other attackers. Do not act on this alert alone." | 

**Escalation rule:** a T4 hit that co-occurs with a T1
or T2 hit on the same source, host or identity within the correlation
window raises the combined alert to the higher tier. A T3 hit plus a T2
hit raises to T1.

**Priority rule:** rule metadata must sort confirmed
(T1) above everything else in every format: Suricata/Snort priority 1–4
mapped to T1–T4; Sigma level critical/high/medium/low; Sentinel severity
High/High/Medium/Informational.
