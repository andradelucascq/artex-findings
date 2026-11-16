---
doc_id: mmd-072-2026-artex
chunk_id: 03
title: "3. Decision Workflow Overview"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [playbook, workflow, decision-process]
---

## 3. Decision Workflow Overview

Every indicator passes through the same eight gates, in order. An indicator that fails a gate stops there and is recorded with the reason.

| Gate | Name | Question answered | 
|---|---|---|
| G1 | Intake & normalize | Is the value clean, de-duplicated, refanged for rules, and tied to a source, date and URL? | 
| G2 | Source verification | How strong is the best source we have actually read? (Section 4) | 
| G3 | Uniqueness class | Is it ARTEX-specific, operator/fork-specific, model-generated, generic, infrastructure or behavioral (Section 5) | 
| G4 | Exclusion check | Does it hit a legitimate service, private data, attacker-enabling detail or an unrelated namesake? (Section 9) | 
| G5 | Tier assignment | Which tier, severity and alert wording follow from G2 × G3? (Section 6) | 
| G6 | Format routing | Which of the seven formats can observe this indicator in real logs? (Section 7) | 
| G7 | Combination design | Must it co-occur with other signals to fire? Which regex or keyword set? (Section 8) | 
| G8 | Lifetime & test | What are valid from / valid until, and has it passed false-positive testing? (Sections 10–11) | 
