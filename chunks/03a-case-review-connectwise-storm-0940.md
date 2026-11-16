---
doc_id: mmd-072-2026-artex
chunk_id: 03a
title: "3A. Case Review — ConnectWise Cloud Backup / Storm-0940 (Oct 2026)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [connectwise, storm-0940, case-study, kill-chain, cloud-backup]
---

## 3A. Case Review — ConnectWise Cloud Backup / Storm-0940 (Oct 2026)

This is a well-documented ARTEX-assisted intrusion, so it is used to test the playbook end to end. Sources: Microsoft (21 Oct 2026), Huntress, ConnectWise advisories, and the colleague case summary. The incident was a compromise of a vendor’s multi-tenant app credential; it was not a compromise of Microsoft systems.

### 3A.1 Kill chain and where each phase is visible

| Phase (2026) | What happened | Visible to | Playbook effect | 
|---|---|---|---|
| 10–14 Oct Password spray | From 144.172.64.99: 327 accounts across 8 orgs, >151,000 failed logons, peak \~2,900/min on 11 Oct. No success, no tokens, no Graph activity. | Each targeted tenant (Entra sign-in logs) | Calibrates B3 thresholds. .99 role = spray source. | 
| \~13–15 Oct Vendor credential theft | Client assertion/secret of ConnectWise Cloud Backup and related Cloud Management apps obtained from ConnectWise’s own tenant. | Vendor only | Not detectable by downstream tenants. No rule; context in STIX. | 
| 15 Oct from 01:58 UTC Role self-assignment | Cloud Management service principals (Exchange Online, Teams, Graph) assigned themselves directory roles, mostly Exchange Administrator, in \~50 Huntress-monitored tenants. | Each tenant (Entra audit log, PIM alerts) | New behavioral rule B6 (proposed). | 
| 15 Oct 03:44–03:49 UTC Global Admin + persistence | Cloud Management – MS Teams (24205314-…) made itself Global Administrator, then a Python-urllib/3.12 script added client secrets (expiring 2028) to an MSP tool and to Huntress’s own app. Source was a Microsoft-internal private IPv6 address. | Each tenant (Entra audit log) | B6 core chain. Source-IP allow-filters would miss it. | 
| 15 Oct 06:13–08:34 UTC Tenant sweep | Cloud Backup app-only sign-ins from 144.172.64.128 to obtain Graph tokens, tenant after tenant in sorted GUID order, seconds apart. | Per tenant: one SP sign-in. Sorted-order pattern: vendor, Microsoft, multi-tenant MSSPs only | Per tenant: T1 on app ID + non-vendor IP. Cross-tenant pattern: guidance for MSSPs, not a public rule. | 
| 15–16 Oct Follow-on | Graph reconnaissance; activity from transient VPN IPs 187.15.88.186 / .90.217 / .89.186. | Each tenant (Graph activity logs) | VPN IPs stay T4 (shared exits). | 
| 16–17 Oct Containment | Advisory published; 7 app identities disabled, certificates revoked, services suspended. | — | App-ID rules stay active until vendor confirms rotation. | 
| Late Oct Restoration | New hardened app ‘ConnectWise Cloud Backup for M365’ requiring re-consent. | — | Do not alert on the new app ID; add it to the documented allow-list once published. | 

### 3A.2 Lessons applied to the playbook

1. **Act on what the identity does, not where it comes
from.** The Global Administrator grant came from
Microsoft-internal private IPv6, so the ConnectWise app-ID rule now
fires on role self-assignment or credential addition regardless of
source IP. Source IP is enrichment only.
2. **Persistence outlives the IOC window.** Secrets
planted with 2028 expiry survive disabling the original apps.
Credential-addition detections have no expiry, and the published hunting
window starts at 2026-10-13, which is earlier than the one-year IOC
window.
3. **Authentication is not exfiltration.** T1 alerts on
app-only sign-ins must say that a successful sign-in alone does not
prove data access, and must direct responders to Graph activity and
audit logs.
4. **Disable, do not delete.** Response text for app-ID
alerts must say to disable the service principal; deleting it destroys
consent records needed for investigation.
5. **Scope includes former customers.** Tenants with
residual consent were hit. Rule documentation must tell users to check
for the app IDs even if they no longer use the product.
6. **Separate IP roles.** 144.172.64.99 = spray (no
success); 144.172.64.128 = primary app sign-in source on 15 Oct
06:13–08:34 UTC; 144.172.64.60 = related sign-in/Graph activity. Each
IP’s metadata records its role and time window.
7. **Some detections belong to vendors and MSSPs.** The
sorted-GUID multi-tenant sweep is only visible across tenants. It goes
into the README as guidance for MSSPs and CSIRTs, not into the
per-tenant rule set.
8. **Do not publish Microsoft-internal addresses.** The
private IPv6 range seen in the escalation is Microsoft cloud
infrastructure; publishing it as an IOC would generate false positives
on legitimate traffic.
