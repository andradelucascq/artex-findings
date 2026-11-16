---
doc_id: mmd-072-2026-artex
chunk_id: 00
title: "Start here (front matter)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [overview, at-a-glance, tldr, navigation]
---

## Start here (front matter)

This is a public, **behavior-first detection pack** against misuse of the ARTEX AI pentest framework, maintained by MalwareMustDie. It is self-published under **TLP:CLEAR**, it is not a CERT advisory, and it is aimed at defenders, xSOCs, CERTs and CSIRTs. The playbook below (v0.4) sets the method, the signatures live on GitHub, and every change since the first release is appended in the version updates and in [Appendix F](#appendix-f-chronology). The current pack is **v1.1.4**.

| At a glance | What you need to know | 
|---|---|
| Status | Every rule ships **experimental**. [The Results](#theresults) tables in this post are internal structure and logic checks on synthetic data, with limited field efficacy and datasets. Not all rules have completed the [Section 11](#section11) seven-day benign-data run or an independent true-positive confirmation yet. Deploy the rules into monitor-only and run them against your own 7 days first, validate in your own environment before you rely on it. | 
| Do NOT auto-block | The T3 IP feed is shared proxies [(Section 6)](#section6), VPS and VPN exits: alert and correlate, never auto-block. The 178.16.54.0/24 segment is alert-only. JPCERT/CC notes its listed addresses may already be in legitimate use. | 
| On HOLD (not shipped) | Console favicon hashes, the health endpoint, the backdoored-binary hash, git-author emails, the self-funding LLM-credential loop, and the deepfake / KYC item. See [Section 9](#9-exclusions-and-hold-queue). | 
| Changed in the last 72 hours | v1.1.0 (durability axis D0/D1/D2; IP indicators now **alert 90 days**, hunt one year, not a flat year; B6 released), v1.1.1 (fork user-agents, Cognito/CloudWatch, Unit 42 patterns), v1.1.2 (FBI FLASH absorption), v1.1.3 (Sophos HOLD items promoted), v1.1.4 (ScopeWeaver WinRM/PsExec gated pivot, two fork user-agents). | 
| If you only do one thing | Deploy the behavior rules (B1 to B6 and the correlations) on your exposed web, identity and cloud logs. Treat every IP and string as short-lived context, not as the detection. | 

**Quick navigation:**
[Playbook: purpose and principles](#1-purpose-and-principles) ·
[Tier matrix (T1 to T4)](#6-tier-assignment-and-alert-wording) ·
[Exclusions and HOLD queue](#9-exclusions-and-hold-queue) ·
[Lifetime and versioning](#10-lifetime-and-versioning) ·
[Testing and validation status](#11-fine-tuning-and-testing) ·
[Appendix D: Zeek](#additional-zeek) ·
[Appendix F: activity chronology](#appendix-f-chronology)

**Version updates:**
[v1.1.0](#update1) ·
[v1.1.1](#update2) ·
[v1.1.2](#update3) ·
[v1.1.3](#update4) ·
[v1.1.4](#update5)

### Prologue

This post leans a bit more into my CTI discipline and I'm definitely putting my CTI hat on to write this. While it still carries cyber threat urgency, non-malware.. and it focuses more on emerged attack tooling that I felt was important to share soon.

I am now releasing custom ARTEX detection signatures pack to the public under **TLP:CLEAR** because active, autonomous threat campaigns leveraging this framework are currently escalating out of control, compressing traditional intrusion timelines from weeks into mere hours, and its intrusion can affect our lifeline.

The purpose is, to help defenders (from xCERT/xSIRT) to help or give idea how to counter these machine-speed identity compromises, API sweeps, and cloud persistence operations. The links for the rules can be found in following table and I have included the analytical playbook detailing the behavioral logic behind this ruleset.

All associated detection tools and signatures are open-sourced and available in the official MalwareMustDie GitHub repository and shared under TLP:CLEAR. In the signature folder there are also several python scripts I made for testing on those signatures. Read and check the following playbook before you decide to use or modify, and I hope this helps!

1. Additionally, the tested OpenIOC can be found here => ([here](https://github.com/unixfreaxjp/malwaremustdie/blob/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/iocs/mmd-artex-iocs.csv)),
2. The tools I build for testing offline can be seen here => ([here](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/tools)),
3. The test results conducted can be seen here => ([here](https://github.com/unixfreaxjp/malwaremustdie/blob/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/tests/)).

The details of the current released detection pack can be found in the repository README.md here => ([here](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara)).

Again, please read, assess and perform tests beforehand. You can decide what to do afterward, it's you to decide. Hope helps!

After 1st version released, I have updated minor versions into v1.1.0 after recent dataset traffic tests, read here => ([here](#update1)), and then there was also minor updates v1.1.1 with more detection to indicators per Nov 14, 2026, here => ([here](#update2)), and since this post and FBI Artex Flash Report was released at the same time so I updated again to v1.1.2 with their published data, with the detail right here => ([here](#update3)), and lastly v1.1.3 update was released here since Sophos X-Ops HOLD indicators finally can be internally confirmed => ([here](#update4)), in the v1.1.3 also I added the Testing.md documentation about using script in /tools to check the signatures to be implemented in your own environment or using its logic for further automation in your own framework, and how to deploy the rules safely in your filtration environments. And one more minor update v1.1.4 was released to add the ExtraHop Reveal Labs ScopeWeaver lateral-movement (WinRM/PsExec) findings and two fork user-agents, read here => ([here](#update5)).

I have also added an activity chronology of public ARTEX reporting (Jul–Nov 2026), compiled at a reviewing CERT's suggestion, as **Appendix F**, read here => ([here](#appendix-f-chronology)).

PS1: I have also included an approach for detecting the released signatures in upper-layer network flows using Zeek, which you can find at the bottom of this post under the ***"Appendix D: (Additional) An idea on using Zeek to detect using released signatures in upper layers networking data-set"*** part here => ([here](#zeek)). This section demonstrates how to automate signature detection from network flows and combine it with behavioral detection across the upper network layers (L3/4). (Thanks to Aaron Kaplan for raising this question.)".

PS2: Readers reading this post might miss the main point. Because Artex is LLM-powered, the post suggests you to use LLMs to stop these behaviors in the future with playbooks built from IOC patterns.

- Many read my ARTEX signature release as "an IOC list". It isn't. The IOCs expire within a year by design. The real release is the workflow: how to turn the next ARTEX-style attack into detections as fast as the attacker moves.
- ARTEX is an LLM harness. Swap the model, prompts or infra and every IOC changes. What changes much less is how an agent behaves: machine-speed recon chains, \~1/s Password Sprays with rotating UAs, self-correcting API retries, systematic record enumeration.
- So the pack is behavior-first. Those patterns live in Sigma correlations, Sentinel KQL and Zeek SumStats on the upper network layers, not in packet filters. IPs are clearly labelled T3/T4 context for retro-hunting what already happened.
- **"The Playbook"** is the core of this sharing: to verify the source, classify how specific an indicator really is, assign a tier, route it to the right format, expire it. We applied it to four campaigns with an LLM assisting and humans gating every verification step. Next trail, same loop.
- That's the big picture: attackers use LLMs to compress weeks into hours. Defenders can use LLMs to compress "new report → tested detection" into hours too, as long as humans keep the verification gates. We must defend attacker's powered LLM/#AI with AI.
- The Zeek section shows how to carry the same behavioral logic into network sensors you already run. Playbook, rules and tests: \[link\]. Break them, send FP/TP reports. That's how this grows.

For any constructive discussion I welcome you to do so here => ([here](https://old.reddit.com/r/netsec/comments/1xv53bm/sharing_artex_detection_signature_pack_from_ioc/)).

FYI, the quick scanning for more or less the currently existing Artex services online I sought at shodan.io is showing below data, per time this blog released day.. seems scary right?


### Signatures Download Table

| Format | Takes | 
|---|---|
| [Sigma](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/sigma) | Web/WAF strings and paths, identity events, Linux process/file events, Class F sequences (correlation rules) | 
| [Sentinel KQL](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/kql) | Entra/M365/Azure/AWS behaviors, app IDs, IP watchlists, retry-timing logic, SOAR hooks | 
| [Suricata 7](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/suricata7) | HTTP headers/UA/URI/body markers, DNS for callback domains, JA3/JA4, IP reputation, console fingerprints (title, port) | 
| [Snort 3](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/snort3) | Same content markers and IP lists as Suricata; no JA4 dependency | 
| [osquery](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/osquery) | Files, accounts, containers/images, listening ports (8787/8790/8890/5435), runner names on servers | 
| [YARA](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/yara) | Webshell and dropped-file content markers; backdoored image layers once a hash is V1 | 
| [STIX 2.1](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara/stix) | Every published indicator plus Tool, Intrusion-Set, Campaign and relationship objects; embedded Sigma/Suricata/YARA patterns | 

### ARTEX Signature Detection Playbook, project definition

**Project:** MalwareMustDie.org public detection pack
against misuse of the ARTEX AI pentest framework

**Version:** Final 0.4 — 13 November 2026.

This is the playbook baseline. The signature pack itself is at v1.1.4: see the version updates ([v1.1.0](#update1) onward) and [Appendix F](#appendix-f-chronology) for every change since, including the v1.1.0 correction that IP indicators alert for 90 days, not one year.

**Audience:** Rule authors and reviewers; downstream
users, CERTs and CSIRTs

**Distribution:** TLP:CLEAR — public release.

**Covered formats:** Sigma (incl. 5 behavioral
correlation rules), Microsoft Sentinel KQL, Suricata 7, Snort 3,
osquery, YARA, STIX 2.1 bundle – Lifetime: 1 year for IOCs; behavioral
rules do not expire.

### Signature development playbook v0.4 (final)
