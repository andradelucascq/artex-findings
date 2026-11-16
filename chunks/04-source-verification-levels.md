---
doc_id: mmd-072-2026-artex
chunk_id: 04
title: "4. Source Verification Levels"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [verification, methodology]
---

## 4. Source Verification Levels

| Level | Definition | Effect on decision | 
|---|---|---|
| V1 | Confirmed: read in the primary vendor/CERT publication, or two independent primary sources agree | Eligible for any tier | 
| V2 | Corroborated: one primary source read via a faithful mirror, or two independent secondary sources agree | Eligible for Tier 1–3; reviewer note required | 
| V3 | Single secondary source, or primary page cited but not retrievable | Maximum Tier 4; flagged "unconfirmed" in rule metadata | 
| V4 | No public source located, or internal-only | HOLD — not published until upgraded | 

**Hash rule:** file hashes are never searched on the
open web. A hash moves above V4 only after a VirusTotal lookup approved
by the project lead.
