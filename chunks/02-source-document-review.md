---
doc_id: mmd-072-2026-artex
chunk_id: 02
title: "2. Source Document Review (Input Baseline)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [sources, cti-reports, verification]
---

## 2. Source Document Review (Input Baseline)

Six inputs were compared by stages until 13 November 2026. The comparison sets the baseline for the decision record in Appendix A.

| Document | Role | Assessment | 
|---|---|---|
| ARTEX CTI Report v1.0 (18 Oct 2026, Internal) | Primary internal analysis; 47 reference URLs | Authoritative for campaign context, ConnectWise app IDs, Korean/JPCERT/Zscaler/Wiz IPs. Two attributions could not be confirmed against the cited vendor page (see 2.2). | 
| MMD\_artex\_iocs.docx (original sheet) | Format template; Eye Security + CTI report | Accurate extraction. Git author emails cannot be re-verified (masked by Cloudflare on the live page). | 
| MMD\_ARTEX\_IOCs\_v2.docx (10 Nov, review) | Superset of the original + referenced vendor reports | Adds Microsoft Storm-3358 and exposed-secrets IPs, full Wiz UA and probe-path set, Zscaler operator artifacts, backdoored Docker images. Carries verification caveats per item. | 
| ConnectWise / Storm-0940 case summary (colleague, 11 Nov) | Timeline and kill chain of the Oct 2026 ConnectWise Cloud Backup compromise | Consistent with Microsoft and Huntress reporting. Re-checked against Huntress’s original analysis, which adds two details that change rule logic (see Section 3A). Note: the incident was a vendor (ConnectWise) app-credential compromise; Microsoft stated its own systems were not compromised. | 
| Korea finance & Japan breach-wave summary (staff, 12 Nov) | Timelines, mechanics and IOCs of the first two ARTEX-linked campaigns | Consistent with CrowdStrike, GMO and JPCERT/CC coverage. Re-checked against Japanese reporting of the JPCERT/CC advisory, which adds attack-technique detail used in B1/B2 (see Sections 3B, 3C). Victim counts differ by measure (records vs individuals); not detection-relevant. | 
| MMD\_ARTEX\_IOCs \_v3.docx (colleagues checks, 13 Nov) | Merge of v1/v2 + Strand, cross-checked with andpalmier, Censys/SOCRadar, Google/Mandiant, Sophos | Most valuable addition: console fingerprints and the source-confirmed vs model-generated split. Several new items have no public source found yet (see 2.3). Some v2 detail needed by the behavioral rules was dropped (see 2.4). | 

### 2.1 Items verified during review

- **Console default port 8787, with 8790 and 443
common:** confirmed in Censys coverage (5 Nov 2026). Zscaler’s
8890 is an operator customization, not a contradiction. Both are
kept.
- **Censys matches consoles by page title plus Next.js static
assets:** confirmed; supports using the title as a durable
console fingerprint.
- **Google/Mandiant artifacts:** artex-probe-\* files and
webhook.site callbacks confirmed via secondary coverage of the 30 Oct
2026 GTIG post.
- **178.16.54.129 vs 178.15.54.129:** no public ARTEX
mention of either. 178.16.54.0/24 (OMEGATECH) carries Spamhaus SBL/DROP
listings on neighboring hosts and URLhaus records of ConnectWise-themed
lures (Jul 2026). 178.15.54.129 has no threat record. Decision by the
project lead: include 178.16.54.129 and the abuse segment; treat
178.15.54.129 as a typo variant.

### 2.2 Attribution conflicts found

- **ReconProbe/1.0:** the CTI report attributes it to
Wiz, but Wiz’s published IOC table does not list it. v3 classes it as
model-generated. Decision: Tier 4.
- **norma/0.4:** listed by Wiz, but v3 notes the Norma
SDK is shared by other agents. Decision: Tier 4, not Tier 1.
- **azure-toolkit/1.0:** now attributed to Microsoft by
two independent internal summaries (CTI report, colleague case summary):
upgraded to V2. Still a generic-looking string, so it stays Class D /
Tier 4.
- **JA3/JA4 hashes:** still not seen in any JPCERT/CC
material available to this project, including the staff summary. Hold at
Tier 4 until a reviewer confirms them in the original.
- **JPCERT/CC IP list (9 addresses):** matched by the MMD
CTI report and the summary of the JPCERT/CC Case E list: upgraded V3 →
V2. JPCERT/CC itself warns that listed addresses may be in legitimate
use by the time of investigation, which is reflected in the alert
text.
- **curl/7.88.1 and python-requests/2.34.2:** also
published by JPCERT/CC as example user-agents for its API-abuse case.
Corroborated, but still generic: Tier 4.

### 2.3 v3 items with no public source located yet

Searches for Strand Intelligence (10 Nov) returned no matching publication. Until a reviewer obtains the original reports, the following v3 items enter the HOLD queue (Section 9), not the pack:

- Favicon SHA256 and mmh3 values; HTTP Basic Auth realm “ARTEX”; health JSON "service": "artex"; recorder proxy 127.0.0.1:8788
- Binary SHA256 of the backdoored build (needs VirusTotal confirmation with project-lead approval before any lookup)
- Fork user-agents: ARTEX-Nuclei-Fingerprint/1.0, ARTEX-CredHunter/2.0, artex-selfupdate, artex-enrich/1.0
- Windows host artifacts (i\<intentID>\_\<purpose>.\<ext>, C:\ProgramData\artx5v\, artx\* under C:\Windows\Temp), zz\<id>.aspx, ZZ\<n>$ accounts
- JSP shells cl1217.jsp, f1219a.jsp, z1219.jsp; disposable mail domains (maxxspace.com, yopmail.com, fldubai.com)

### 2.4 v2 detail to restore in the merged baseline

The behavioral rules depend on items v3 summarized away. These must be carried into the merged IOC baseline from v2:

- Wiz account-recovery, registration, OAuth and SSO probe endpoints, the alg:none JWT test, and AWS Cognito event names (Rule B2)
- Microsoft exposed-secrets ARM retry pattern (28–80 s, corrected api-version) and Storm-3358 tooling WISP / vshell / AdaptixC2 (Rules B4, B5)
- ConnectWise guidance that legitimate Cloud Backup egress is Azure private IPv6 — kept as context only. It must NOT be used as an allow-filter: Huntress saw the Global Administrator escalation itself come from a Microsoft-internal private IPv6 address (see Section 3A).
- Per-item verification caveats (git authors unverified; date first-seen per IP)
