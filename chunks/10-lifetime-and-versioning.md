---
doc_id: mmd-072-2026-artex
chunk_id: 10
title: "10. Lifetime and Versioning"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [ioc-lifetime, versioning]
---

## 10. Lifetime and Versioning

- **Infrastructure (Class E):** valid\_from 2026-11-13,
valid\_until 2027-11-13 (one year), per project decision. Each IP keeps
its own first-seen date and source in metadata.
- **Abuse segment 178.16.54.0/24:** included at T4 for
the same one-year window, alert-only (never auto-block) because the /24
is multi-origin hosting with collateral risk.
- **Class A–D string indicators:** one-year review, not
hard expiry; downgrade when forks are shown to change them.
- **Class F behavioral rules:** no expiry; lifecycle
status only.
- **Hunting window for ConnectWise-related content:** from 2026-10-13 (earliest reported activity against the apps) onward,
independent of the one-year IOC window, because planted secrets can
remain valid until 2028.
- **ConnectWise app IDs:** remain T1 until ConnectWise
confirms the old identities are permanently retired; then downgrade to
hunting-only rather than delete.
- **Identifiers:** reserve a dedicated Suricata/Snort SID
block and stable Sigma UUIDs before writing; never reuse an ID after
deletion.
- **Releases:** semantic versions (v1.0.0 = first
public). GitHub tag, MISP event, OTX pulse and social posts always
reference the same tag.
