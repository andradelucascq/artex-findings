---
doc_id: mmd-072-2026-artex
chunk_id: appendix-c
title: "Appendix C — Change Log"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [changelog]
---

## Appendix C — Change Log

| Version | Changes | 
|---|---|
| 0.1 | Initial draft: document review, gates G1–G8, tiers, routing, exclusions, decision record. | 
| 0.2 | Added Section 3A (ConnectWise / Storm-0940 case review, re-checked against Huntress). Removed source-IP condition from the ConnectWise app-ID T1 rule. Proposed behavioral rule B6. Split Storm-0940 IPs by role and time window. Added hunting window from 2026-10-13, ‘disable, do not delete’ and ‘sign-in ≠ data access’ alert text, former-customer scope, and three new exclusions. azure-toolkit/1.0 upgraded to V2 (still T4). | 
| 0.3 | Added Section 3B (Korea finance case) and 3C (Japan breach wave, JPCERT/CC cases A–E). Expanded B1 (A2B, config/backup/WSUS, SSRF-redirect branches) and B2 (differential probing, NoSQL injection, low-and-slow distinct-record enumeration). JPCERT/CC IPs upgraded V3→V2 and T4→T3 with reassignment warning. 38.244.50.120 marked cross-campaign with T1 escalation on any B-rule hit. Added operator-workspace hunting item, five exclusions (personal data, victim names, unpublished IOCs, unrelated GMO VPS incident, Metabase case), and per-campaign STIX intrusion sets. | 
| 0.4 | FINAL. Editorial corrections by the project lead (source document names and dates, terminology, section titles). Version labels set to 0.4, distribution set to TLP:CLEAR, US spelling applied throughout, andpalmier recorded as a cross-check source. Approved as the baseline for writing the signature set. | 
