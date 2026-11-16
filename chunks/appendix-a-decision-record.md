---
doc_id: mmd-072-2026-artex
chunk_id: appendix-a
title: "Appendix A — Decision Record for the Current IOC Set"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [decision-record, iocs]
---

## Appendix A — Decision Record for the Current IOC Set

Application of Gates G2–G6 to the merged v2 + v3 baseline. “Formats” uses: Sg = Sigma, K = Sentinel KQL, Su = Suricata 7, Sn = Snort 3, O = osquery, Y = YARA, X = STIX.

| Indicator group | Source | Class | Tier | Formats | Notes | 
|---|---|---|---|---|---|
| Webshell: X-ARTX-KEY header, ### ARTX-AGENT response | V1 Eye | A | T1 | Su Sn Sg Y X | Response marker also in YARA for webroot scans | 
| Shell key k=x9f3c2b7e | V1 Eye | B | T1 | Su Sn Sg X | Operator-specific; may rotate | 
| UA Mozilla/5.0 (ARTEX), ArtExVerify/1.0 | V1 Eye | A/B | T1 | Su Sn Sg X | (ARTEX) noted as fork-specific in v3 | 
| Probe markers ARTEX\_WRITE\_PROBE, ARTX\_PERSIST\_OK, artex\_control\_marker\_zz9q, ARTEX-PROBE-0001 | V1 Eye | C | T2 | Sg Su Sn Y X | Fire with a second signal | 
| File patterns artx\_\*, artx-\*, zzartx0000.php, artex-probe-\* | V1 Eye / V2 GTIG | C | T2 | Sg O Y X | Anchored regex + write context | 
| Control-404 probes zz-notexist-\*, artx-nonexistent, zzz-nonexistent-xyz123 | V1 Eye/Wiz | C | T2 | Sg Su Sn | Combine with recon chain B1 | 
| Accounts qa\_probe\_\*, qa.audit29a; runners sink-\* | V1 Eye | B/C | T2 | Sg O K X | B5 component | 
| DB: INTO OUTFILE /tmp/x.php5, LOAD DATA LOCAL into artx\_\* | V1 Eye | B | T1 | Sg O X |  | 
| Backdoored images tomicdd/artex, congdonglinux/artex | V2 THN/Badbyte | A | T1 | O Sg X | Defends red teams too | 
| Other Docker namespaces (mianhuatang11/artex etc.) | V2 | B | T4 | O X | Not confirmed backdoored | 
| Image tag artex-local:perf2-20261003; ports 8890/5435 | V1 Zscaler | B | T1/T3 | O X | Operator infra; ports as context | 
| Console default ports 8787/8790 + console page title | V2 Censys | A | T3 | Su Sn O X | Port alone = T4; title+port = T3 | 
| Favicon hashes, realm, health JSON | V4 | A | HOLD | — | Awaiting Strand Intelligence original | 
| ConnectWise app IDs (7): role self-assignment or credential addition | V1 CW/Huntress/MS | B | T1 | K Sg X | Any source IP; text: disable, do not delete | 
| ConnectWise app IDs (7): SP sign-in from 144.172.64.0/24 or other non-vendor IP | V1 | B | T1 | K Sg X | Text: sign-in ≠ data access; check Graph logs | 
| ConnectWise app IDs (7): consent present only | V1 | B | T4 | K X | Hunting context; covers former customers | 
| Credential added to any app by a third-party SP within minutes of a role grant (B6) | V1 Huntress | F | T2 | K Sg | Pending approval of B6 | 
| 144.172.64.99 (spray source, 10–14 Oct) | V1 MS | E | T3 | K Su Sn Sg X | Role recorded in metadata | 
| 144.172.64.128 (app sign-ins, 15 Oct 06:13–08:34 UTC) | V1 MS/CW/Huntress | E | T3 | K Su Sn Sg X | Combined with app ID → T1 | 
| 144.172.64.60 (related sign-in / Graph) | V1 MS/CW | E | T3 | K Su Sn Sg X |  | 
| Transient VPN 187.15.88.186/.90.217/.89.186 (15–16 Oct) | V2 CTI/colleague | E | T4 | K Su Sn X | VPN exits; high collateral | 
| 38.244.50.120 (KR ARTEX host / jump server; JPCERT/CC list) | V1 CS/Zscaler + V2 JPCERT | E | T3 | K Su Sn Sg X | Cross-campaign; any B-rule hit from it → T1 | 
| Korean campaign proxy IPs (9) | V1 CS | E | T3 | K Su Sn X | Shared proxies; alert, do not auto-block | 
| Zscaler work servers 23.248.226.250–254, proxies | V1 Zscaler | E | T3 | K Su Sn X |  | 
| Wiz nodes 104.194.150.21, 167.88.172.97 | V1 Wiz | E | T3 | K Su Sn X |  | 
| JPCERT/CC Case E IPs (8 besides 38.244.50.120) | V2 CTI/staff | E | T3 | K Su Sn X | Text: may now be legitimately used (per JPCERT/CC) | 
| Storm-3358 IPs (WISP C2, ARTEX servers, C2) incl. 178.16.54.129 | V2 MS mirror | E | T3 | K Su Sn X | Upgrade to V1 when MS original read | 
| 178.16.54.0/24 abuse segment | V2 Spamhaus/URLhaus | E | T4 | K Su Sn X | Alert-only, no block | 
| 178.15.54.129 typo variant | V3 | E | T4 | K X | Explicit typo note | 
| 199.19.77.84 (exposed-secrets SP sign-ins) | V2 | E | T3 | K Su Sn X |  | 
| Eye infra 207.5.195.93; artex\[.\]lol family domains | V1 Eye | E | T3 | Su Sn K X | DNS + HTTP Host | 
| xcai.pro LLM gateway | V1 Zscaler/CS | E | T3 | Su Sn K X | Egress from your own network = operator inside | 
| Callback \*.dnslog.cn, \*.oast.me, webhook.site | V1 Eye/GTIG | D | T4 | Su Sn Sg | Second signal for T2 combos | 
| Neo-reGeorg UA Firefox/38.0 | V1 Eye | D | T4 | Su Sn Sg | Used by many actors | 
| Generic UAs (curl, python-requests, Python-urllib, Boto3, aws-cli, Wiz browser set, norma/0.4, ReconProbe/1.0, azure-toolkit/1.0) | V1–V3 | D | T4 | Sg K Su Sn X | WARNING text mandatory | 
| JA3/JA4 (JPCERT) | V3 | D | T4 | Su X | Snort 3 lacks JA4 | 
| Zscaler env vars, scripts, workspace files, server accounts | V1 Zscaler | B | T2 | O Y X | For hunting your own servers / seized hosts | 
| Storm-3358 tooling WISP, vshell, AdaptixC2 | V2 MS/Censys | B | T2 | X (+ link to existing rules) | Reference existing family rules | 
| Git authors [artx@example.local](mailto:artx@example.local)/.com | V4 | B | HOLD | — | Re-verify | 
| v3 Windows/JSP/aspx artifacts, disposable mail domains, fork UAs | V4 | C | HOLD | — | Section 2.3 | 
| Operator workspace folders (.claude/, .codex/, .qwen/, CLAUDE.md) on hosts also showing ARTEX ports | V2 CS/JPCERT | B | T2 | O X | Hunting on own servers; single folder alone = T4 | 
| Open directory listing exposing CLAUDE.md or ARTEX config | V1 CS | B | T4 | Su X | For hosting providers / CSIRTs | 
| Metabase-GHSA-vwf4/2.0, python-requests/2.33.1 (JPCERT Case C) | V2 | — | Out of scope | — | Optional companion rule | 
| Behavioral B1–B5 (B1/B2 expanded in v0.3) | V1 multi | F | T2 | Sg K | Section 8.1 | 
