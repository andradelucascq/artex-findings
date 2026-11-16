---
doc_id: mmd-072-2026-artex
chunk_id: 07
title: "7. Format Routing"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [sigma, kql, suricata, snort, yara, stix, osquery]
---

## 7. Format Routing

| Format | Takes | Does not take | 
|---|---|---|
| Sigma | Web/WAF strings and paths, identity events, Linux process/file events, Class F sequences (correlation rules) | Raw IP lists longer than \~20 (use STIX/KQL watchlists) | 
| Sentinel KQL | Entra/M365/Azure/AWS behaviors, app IDs, IP watchlists, retry-timing logic, SOAR hooks | Packet-level content | 
| Suricata 7 | HTTP headers/UA/URI/body markers, DNS for callback domains, JA3/JA4, IP reputation, console fingerprints (title, port) | Identity-provider events; TLS payloads without inspection | 
| Snort 3 | Same content markers and IP lists as Suricata; no JA4 dependency | JA4 (not supported natively) | 
| osquery | Files, accounts, containers/images, listening ports (8787/8790/8890/5435), runner names on servers | Network content | 
| YARA | Webshell and dropped-file content markers; backdoored image layers once a hash is V1 | Infrastructure; behavior | 
| STIX 2.1 | Every published indicator plus Tool, Intrusion-Set, Campaign and relationship objects; embedded Sigma/Suricata/YARA patterns | HOLD items; private or attacker-enabling data | 
