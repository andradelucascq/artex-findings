---
doc_id: mmd-072-2026-artex
chunk_id: 09
title: "9. Exclusions and HOLD Queue"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [hold-queue, do-no-harm, exclusions]
---

## 9. Exclusions and HOLD Queue

| Item | Decision | 
|---|---|
| IlyassCODEX/ARTEX | Exclude: unrelated namesake project. | 
| api.deepseek.com, Backblaze B2, webhook.site, \*.oast.me, \*.dnslog.cn | Legitimate or widely used testing services. webhook.site / oast.me / dnslog.cn enter only as T4 enrichment or as the second signal in a T2 combination. api.deepseek.com and Backblaze excluded as indicators. | 
| xcai.pro | Include as T3 (LLM reseller used by operators); note that other customers may use it. | 
| 127.0.0.1:5435, 127.0.0.1:8788 | Not network IOCs. osquery listening-port context only. | 
| Backdoor secret marker | Never published (unknown, and would enable abuse). | 
| Telegram handle YY520CN | Exclude from the public pack: identifies an individual and CrowdStrike could not tie it definitively to the operator. | 
| Git authors [artx@example.local](mailto:artx@example.local) / .com | V4 until re-verified; HOLD. | 
| All Section 2.3 items | HOLD until a reviewer upgrades the source level. | 
| 178.15.54.129 | Publish as T4 typo variant with explicit note; never as T3. | 
| Microsoft-internal private IPv6 seen in the Global Admin escalation | Exclude as an indicator (Microsoft cloud infrastructure). Mention in rule notes only, to explain why source-IP filtering is not used. | 
| Huntress and MSP tool app IDs that received planted secrets | Exclude: victims, not indicators. | 
| New ‘ConnectWise Cloud Backup for M365’ app | Never alert on it; document it as a legitimate replacement once its ID is published. | 
| Personal details in the Korean operator’s sessions (name, phone, Telegram handle) | Exclude entirely from rules, STIX and README. Not definitively tied to the operator; private-person data. | 
| Victim organization names (Korean banks, Japanese companies) | Exclude from rules and STIX. Mention only as published by regulators, if at all, in the README. | 
| Unpublished Hong Kong primary server; GMO’s law-enforcement-only IOCs | Nothing to include; do not infer or reconstruct. | 
| GMO Internet Group Onamae.com Desktop Cloud / FX VPS incident | Distinct event. Do not merge its indicators into the ARTEX pack. | 
| JPCERT/CC Case C (Metabase) indicators | Not ARTEX-attributed. Out of scope; optional companion rule with its own label. | 

**HOLD queue handling:** HOLD items live in a private
tracking file with the missing evidence stated. Review weekly; promote
when the source level reaches V2 or better, otherwise drop at the next
major release.
