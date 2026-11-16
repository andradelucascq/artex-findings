---
doc_id: mmd-072-2026-artex
chunk_id: appendix-e
title: "Appendix E: Addendum (14 Nov 2026) — updates from further and deeper internal tests, pack v1.1.0"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [v1.1.0, addendum, update]
---

## Appendix E: Addendum (14 Nov 2026) — updates from further and deeper internal tests, pack v1.1.0

Everything above this appendix stays as published on 13 November 2026. This appendix adds what we learned after running the pack against our own deeper internal simulation, and it is the only place where corrections are made. The pack is now **v1.1.0**. The detection logic of every 1.0.0-rc1 rule is unchanged (same SIDs, same `rev`, same Sigma IDs); the only rc1 text change is the B6 release below. New content is appended to the existing files and marked `Added in v1.1.0`.

### What the new dataset tests showed

1. **Markers are easy to rename.** We renamed the ARTEX strings (`artx_*`, `X-ARTX-KEY`, `k=x9f3c2b7e`, probe and account names) in every positive test case. All 13 marker-based (D0) Sigma rules went silent, which is what we expected. The rules exposed to the same rename that kept firing were the D1 and D2 ones. That is the point of this release: the IOCs and markers are context with a short life, and the behavior is the detection.
2. **Windows was a gap.** The rc1 webshell foothold chain (B5) was Linux-only. When we ran the simulated agent against Windows, the files it dropped in `C:\Windows\Temp\` and the PowerShell it ran were not detected.
3. **Upload-then-execute did not need the ARTEX file name.** Matching on `artx_*.php` misses the same attack under a random name. Matching on "an upload was accepted, then a script was requested from an upload folder by the same client" does not miss it.
4. **Cloud coverage was Azure-only.** Arctic Wolf has since reported **AntiFox**, a fork of ARTEX 0.1.1 that abuses valid AWS IAM users and assumed EC2 roles to steal credentials. The playbook already listed AWS as a target for KQL; rc1 had no AWS rule. That is corrected now.
5. **OAuth consent and Graph reconnaissance** by an app identity were not covered outside the ConnectWise app IDs.
6. **An IP spray rotated across many source IPs** stays below the per-IP thresholds of both Sigma B3 and KQL k03. This is not fixed in v1.1.0; see "Known gaps" below.

### What changed in v1.1.0

| Area | Update | 
|---|---|
| Durability axis | New label for every rule: **D0** renamable string or IOC, **D1** needs a code change to evade, **D2** behavior-inherent. All v1.1.0 rules are D2. The durability of the rc1 rules is declared in the linter, so the rc1 rules did not need a new `rev`. `python3 tools/mmd_artex-linter.py --coverage` prints a telemetry-layer × durability matrix, so you can see where your coverage depends on strings. | 
| Rename-proof B5 | Sigma: upload accepted → server-side script requested from an upload folder (any name, nested folders) within 2 minutes. Suricata 7: SID 4260120/4260121, an xbits pair that keeps the upload/execute state per client. | 
| Windows webshell | Sigma and osquery: IIS / PHP-CGI / Apache / Tomcat worker spawns cmd, PowerShell or a common LOLBin. Sigma also covers a web worker writing a script into the web root or `C:\Windows\Temp\`, and the B5w chain (the drop, then a shell on the same host within 15 minutes). There is a Linux equivalent (web server spawns a shell, interpreter or downloader). | 
| AWS (AntiFox) | B4 ported to CloudTrail: one IAM identity reads ≥10 distinct secrets in 10 minutes, makes ≥10 distinct IAM/STS discovery calls in 5 minutes, or touches ≥15 buckets in 10 minutes. Sigma value\_count correlations plus a KQL section in k04. Allow-list your IaC and CSPM roles. | 
| Identity | OAuth consent with high-privilege Graph scopes, for any app and from any IP (Sigma and KQL k01). Service-principal Graph directory-recon burst (KQL k02, MicrosoftGraphActivityLogs). | 
| LLM API egress policy | T4 policy rules for servers that resolve or connect to public LLM API endpoints (Suricata/Snort SID 4260400–4260419, Sigma DNS, KQL k10). An agent harness needs an LLM backend whatever it calls itself. These domains are policy categories, **not IOCs**: allow-list your sanctioned AI clients and watch your servers. | 
| IOC lifetime (correction) | The main post says the IOCs expire within one year. That is still the hunting window, but IP indicators now **alert only until 2027-02-11 (90 days)** and stay **hunt-only until 2027-11-13**. They are reassigned too quickly to page on for a full year. The IOC CSV has new `alert_until`, `hunt_until` and `durability` columns, and k05 has a watchlist variant that applies the split. STIX `valid_until` is unchanged. | 
| STIX | AntiFox tool object ( `derived-from` ARTEX), an unattributed AntiFox intrusion set and campaign, and indicators for the new rules. The report is renamed v1.1.0 and its `modified` date is 2026-11-14. The rc1 objects are unchanged. | 
| B6 released | B6 (a vendor service principal joins a privileged role, then adds credentials to another application within 30 minutes, from any source IP) was PROPOSED in rc1 and is now **RELEASED**. It caught the service-principal credential step in our simulation. Detection logic is unchanged; the Sigma files stay in `sigma/proposed/` so existing paths do not break, with `modified: 2026-11-14`. KQL k11 and the STIX B6 indicators are updated to match. | 
| Tools and tests | The Sigma harness now evaluates correlation rules on event sequences. It reads every `tests/sigma_events*.json` file, reports documented expected misses, and has a `--mutate` mode that renames the markers and checks each rule's declared durability. New test files only: `sigma_events_v1.1.0.json` (79 cases), `sigma_events_simulation_v1.1.0.json` (17 steps of our simulation scenario, synthetic data) and `nids_samples_v1.1.0.json`. The rc1 test files are unchanged. | 


### The Results

| Check | Result | 
|---|---|
| Linter (7 formats) | PASS: Suricata 101 rules, Snort 99, Sigma 56 documents, YARA 4, STIX 452 objects, osquery 14, KQL 11 files / 16 queries | 
| Sigma harness | 157/157 pass, including 34 correlation/sequence cases; 43/43 single-event rules have a positive test; 4 documented expected misses | 
| Mutation test | 15 rules mutated: all 13 D0 rules died (expected), the D1 and D2 rules survived; 0 durability violations | 
| NIDS content | 33/33 pass | 

These are offline structure and logic checks, the same as for rc1. Native engine runs, PCAP replay (especially the xbits pair) and a 7-day benign-data run (playbook Section 11) are still required before production use.

### Known gaps (documented, not hidden)

1. A password spray rotated across many source IPs (one or two accounts per IP) stays below the per-IP thresholds of Sigma B3 and KQL k03. Rely on IdP smart lockout and identity-protection risk detections for now. A per-tenant spray rule is a candidate for a later version.
2. `MARKER123` / `ARTX123` came up in our simulation, but we found no public source for them. They are unverified and are not shipped.
3. `C:\Windows\Temp\artx*` and `i<intentID>_*.ps1` remain playbook HOLD items, so there is no name-based rule. When a web worker writes or runs them, the v1.1.0 behavior rules catch them; when a file is written outside a web worker, nothing catches it yet.

### Where to get it

The same [GitHub folder](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara), same directory structure. See `CHANGELOG.md` (1.1.0) and README section 12 for which existing file received which rule. Verify the files with `MANIFEST.sha256`. Please break the rules and send false- and true-positive reports; that is how this pack grows.

malwaremustdie.org - @unixfreaxjp 20261114v1.1.0 (addendum)
