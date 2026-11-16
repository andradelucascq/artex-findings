---
doc_id: mmd-072-2026-artex
chunk_id: 11
title: "11. Fine-Tuning and Testing"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [testing, validation, benign-data]
---

## 11. Fine-Tuning and Testing

| Stage | Requirement | 
|---|---|
| experimental | Rule compiles/validates (sigma-cli check, suricata -T, snort -T, osqueryi, yara -C, STIX validator). Positive test sample exists (synthetic log line or crafted PCAP). | 
| test | Run against ≥7 days of benign data per log type (web, identity, cloud, PCAP). False positives documented in the rule’s falsepositives field. T1 target: zero FP; T2: ≤1 FP/day per 10k users or 1M requests; T3/T4: no target, but volume recorded. | 
| stable | Two independent confirmations from users or CSIRTs (GitHub issue, MISP sighting), or one confirmed true positive in a real incident. | 
| deprecated | Source retracted, indicator reassigned, or FP rate unacceptable; kept in the repo with reason for one release, then removed. | 

**Feedback loop:** GitHub issue templates for false
positive, true positive and new indicator; triaged weekly; changes land
in the next patch release with a changelog entry citing the issue.
