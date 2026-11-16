---
doc_id: mmd-072-2026-artex
chunk_id: 12
title: "12. Packaging and Publication Checks"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [qa, publication]
---

## 12. Packaging and Publication Checks

1. Every rule carries: tier tag, source name and URL, source level (V1–V3), first-seen date, valid\_until, author, license, MITRE ATT&CK IDs.
2. License chosen once for the whole pack (Detection Rule License 1.1 is the Sigma community norm).
3. Indicators are refanged inside rules and STIX patterns, defanged in README, MISP/OTX descriptions and social posts.
4. STIX bundle: Tool (ARTEX), Intrusion-Sets kept separate per campaign (Storm-0940, Storm-3358, unattributed KR finance operator, unattributed JP breach-wave operator(s)); shared indicators such as 38.244.50.120 linked to each campaign without merging actors; Indicators with pattern\_type stix/sigma/suricata/snort/yara, confidence mapped from tier, TLP:CLEAR marking, external references to every source.
5. Credits: Eye Security, Microsoft, Wiz, Zscaler, CrowdStrike, JPCERT/CC, ConnectWise, Censys, Google/Mandiant, Check Point, Badbyte; andpalmier (cross-check); Sophos X-Ops, FBI IC3, ExtraHop Reveal Labs (v1.1.4 ScopeWeaver WinRM/PsExec lab and fork user-agents), SOCRadar (v1.1.4 Ayven fork and exposed-console mapping), GBHackers (secondary coverage), plus Strand Intelligence once its report is confirmed.
6. Final review against Section 9 exclusions immediately before tagging a release.
