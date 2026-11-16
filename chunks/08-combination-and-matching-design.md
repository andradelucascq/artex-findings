---
doc_id: mmd-072-2026-artex
chunk_id: 08
title: "8. Combination and Matching Design"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [correlation-rules, behavioral-detection]
---

## 8. Combination and Matching Design

Regex and keyword sets are acceptable starting points, provided each Class C or D signal is paired with at least one other signal. Design rules:

- **Anchor:** filenames as ^artx\[\_-\] or ^zzartx\d+\.php$,
not substring artx.
- **Case:** list case variants explicitly (ARTX, artx)
rather than /i over long patterns, to keep Snort/Suricata fast-pattern
efficient.
- **Co-occurrence:** Class C requires a second element,
e.g. artx\_ file write + .php extension + ?k= parameter, or artex-probe-\*
+ webhook.site callback.
- **Windows and thresholds:** every Class F rule declares
its window and count (e.g. ≥60 SSO route variants in 120 s; spray median
≤1 s across ≥20 accounts).
- **Allow-lists:** each rule ships an empty, documented
allow-list for authorized ARTEX testers and internal scanners.

### 8.1 Behavioral rules (Class F)

B1–B5 are the five agreed rules. B6 is proposed from the ConnectWise case review and needs the project lead’s approval before it joins the set. It is listed before B5 only for readability.

| ID | Name | Signal chain | Data sources | 
|---|---|---|---|
| B1 | Recon & API-discovery chain | \*.js.map or env.js/config.json → swagger/openapi → previously unseen API route, same source, short window. Branches: (a) A2B — internal/admin API routes never called by the official app, from a client lacking the app’s normal headers; (b) config/backup downloads (.bak, .zip, .sql, .env) and WSUS probing; (c) SSRF probes with URL parameters pointing to hosts that redirect (302) to internal/metadata addresses | Web server, WAF, reverse proxy, API gateway | 
| B2 | Auth & authorization probing | alg:none JWT, malformed OAuth callback, SSO route-variant burst, reset/username-check burst. Branches: (a) differential probing — same endpoint with headers removed or tokens malformed, then response class changes; (b) NoSQL-injection operators in auth/search parameters; (c) low-and-slow enumeration — one identity/session reading an unusually high count of distinct record IDs over hours, no rate condition (Korea case: \~25,000 records in \~30 h) | Web, API gateway, DB audit, Okta, Entra, Cognito | 
| B3 | Distributed spray | ≈1 s cadence across many accounts; UA rotates per burst; IP rotation after block. Calibration from the Storm-0940 case: 327 accounts / 8 orgs, >151,000 failures over 5 days, peaks \~2,900/min | Entra, Okta, Cognito | 
| B4 | Agentic cloud retry | SP sign-in without MFA → ARM 4xx → corrected retry in 28–80 s → Key Vault/Storage read | Entra SP sign-ins, Azure Activity, Key Vault | 
| B6 (proposed) | Vendor app-identity abuse & persistence | Third-party service principal adds itself to a privileged directory role (Global / Exchange / Privileged Role Administrator) → within minutes adds a client secret or certificate to a different application (often with multi-year expiry). Fires regardless of source IP. Seed list: the 7 ConnectWise app IDs; generic form covers any vendor app. | Entra audit log, PIM alerts, SP sign-in logs, Graph activity | 
| B5 | Webshell foothold | ARTX-marker write → INTO OUTFILE / LOAD DATA LOCAL → qa\_\* or ArtexProbe account → nsenter or runner registration | Linux audit, DB logs, web, GitHub audit | 
