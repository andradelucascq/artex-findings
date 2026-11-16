---
doc_id: mmd-072-2026-artex
chunk_id: 05
title: "5. Uniqueness Classes"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [classification, methodology]
---

## 5. Uniqueness Classes

| Class | Description | Examples | Default treatment | 
|---|---|---|---|
| A | ARTEX-unique, hard-coded or framework-default | ### ARTX-AGENT, X-ARTX-KEY, ArtExVerify/1.0, console title, backdoored image names | Single-signal alert allowed | 
| B | Operator- or fork-specific | Mozilla/5.0 (ARTEX), k=x9f3c2b7e, artex-local:perf2-20261003, sink-\* runners | Single-signal alert; note fork/operator scope | 
| C | LLM- or tool-generated naming patterns | artx\_\*, artex-probe-\*, qa\_probe\_\*, zz-notexist-\*, ARTEX-PROBE-0001 | Alert only with a second signal (Section 8) | 
| D | Generic tool or library strings | curl/7.88.1, python-requests/\*, Boto3, browser UAs, norma/0.4 | Never alerts alone; enrichment only | 
| E | Infrastructure (IP, domain, segment, port) | Campaign IPs, artex\[.\]lol, 178.16.54.0/24, 8787/tcp | Tier by source; time-boxed | 
| F | Behavioral sequence | Recon chain, spray cadence, ARM retry, webshell foothold | Sigma correlation / KQL; no expiry | 
