---
doc_id: mmd-072-2026-artex
chunk_id: 03c
title: "3C. Case Review — Japanese Breach Wave (Sep – Oct 2026)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [japan, jpcert, case-study, kill-chain]
---

## 3C. Case Review — Japanese Breach Wave (Sep – Oct 2026)

Sources: GMO Cybersecurity by Ierae (19 Oct 2026, confirmed ARTEX on attack infrastructure) and the JPCERT/CC advisory (first issued 8–9 Oct, updated through 23 Oct). GMO observed one infrastructure targeting \~20 Japanese companies (9 in one industry) plus six further AI-enabled attack servers by sector; it shared full IOCs with law enforcement only. JPCERT/CC describes several cases; the staff summary reports a later Case E naming ARTEX, Cairn and CyberStrikeAI. Actor(s) unknown; not linked publicly to the Korean campaign or Storm-0940.

### 3C.1 JPCERT/CC cases and detection relevance

| Case | Technique (per JPCERT/CC reporting) | Playbook effect | 
|---|---|---|
| A | Scanning each target for many different known vulnerabilities and management misconfigurations, not one common bug. | Supports B1; no single-CVE rule. | 
| B | App-to-Backend (A2B): public smartphone apps analyzed to find API endpoints and keys; internal APIs not reachable from the UI attacked to change user privileges or create accounts; responses compared while toggling headers or sending malformed tokens; blind NoSQL injection to find accounts; API keys stolen elsewhere reused. Example UAs curl/7.88.1, python-requests/2.34.2. | B1 gains an A2B branch; B2 gains header-toggle / malformed-token differential probing and NoSQL-injection probes. | 
| C | Exploitation of a Metabase SQL-injection vulnerability via API (UAs python-requests/2.33.1, Metabase-GHSA-vwf4/2.0). | Part of the same wave but not ARTEX-attributed. Out of scope for the ARTEX pack; may ship as a separately labeled companion rule. | 
| D | Web shells planted on application servers reachable from public web servers. | Supports B5. | 
| E (staff summary) | AI pentest platforms (ARTEX, Cairn, CyberStrikeAI); attempts against internet-facing WSUS servers and downloads of configuration/backup files; IP list of nine addresses incl. 38.244.50.120. | B1 gains WSUS and config/backup download stages. IP list T3 at V2. | 

### 3C.2 Operator tooling seen on attacker servers

- Agent configuration folders for Claude Code, OpenAI Codex CLI and Qwen
- Android/Flutter reverse engineering: jadx, Blutter
- SSRF test scripts based on HTTP 302 redirects to internal targets
- Web shells and tools for MySQL, Jenkins and mail gateways

### 3C.3 Lessons applied to the playbook

1. **Mobile apps expose the API map.** B1’s A2B branch
fires when internal or admin API routes, never called by the official
app or web UI, are hit by a client lacking the app’s normal headers or
signing, especially after many 401/403 responses.
2. **Differential probing is a strong signal.** The same
endpoint requested repeatedly with headers removed, tokens malformed or
alg changed, followed by a different response class, is added to
B2.
3. **SSRF via redirect.** Requests whose URL-type
parameters point to an external host that answers 302 to internal or
metadata addresses (e.g. 169.254.169.254, localhost, RFC1918) are added
to B1 as a late-stage probe.
4. **Config, backup and WSUS exposure.** Downloads of
backup/config artifacts (.bak, .zip, .sql, .env, config.\*) and probing
of internet-facing WSUS endpoints join B1. Exact paths to be taken from
the JPCERT/CC original during rule writing.
5. **Rate limits are a control, not a rule.** JPCERT/CC
recommends per-function rate limits on login, password reset, SMS and
search. These go into the README’s mitigation section.
6. **Listed IPs may already be clean.** JPCERT/CC states
that published addresses may be legitimately used at investigation time.
T3 alert text for this list must say so.
7. **No novel zero-days.** All cases exploited known
issues and misconfigurations, which supports behavior-first detection
over exploit signatures.
