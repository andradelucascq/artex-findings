---
doc_id: mmd-072-2026-artex
chunk_id: G
chunk_type: supplementary-research
title: "External Source Verification — Case Review Citations (Supplementary, Not Part of the Original Post)"
derived_from: "Links and plain-text source URLs embedded in the MMD-072-2026 post (IOC tables, case reviews, Section 2 source review), verified by live fetch on 2026-11-16"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
verification_date: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [external-sources, verification, case-review-support, storm-0940, storm-3358, connectwise, south-korea, japan, jpcert, eye-security, wiz, crowdstrike, zscaler, microsoft, huntress, urlhaus, github]
---

# External Source Verification — Case Review Citations

**This chunk is not part of MalwareMustDie's original post.** It is a
supplementary research artifact: every external URL that appears as a
plain-text citation inside the source document (mostly in the IOC CSV
excerpts, the case reviews, and the Section 2 source-review table) was
fetched live on 2026-11-16 and cross-checked against what the MMD post says
about it. The goal is to let an investigating analyst or agent jump
straight from a case review or IOC row to the underlying vendor reporting,
and to flag where the vendor source says something more specific, more
hedged, or slightly different from the MMD summary — so correlations built
on this pack rest on primary sources, not just a secondary retelling.

Thirteen distinct external URLs were found embedded as plain text in the
post (separate from the ~9 GitHub signature-repository links, which all
point to subfolders of the same repo). Ten were fetched successfully; one
(URLhaus) returned a bot-check interstitial with no data; one (the Reddit
discussion thread) was blocked by the fetch tool itself.

## 1. Sources backing Case Review 3A (ConnectWise Cloud Backup / Storm-0940)

| Source | Date | What it independently confirms |
|---|---|---|
| [Microsoft Security Blog — "Storm-0940 uses ARTEX, agentic AI, ConnectWise supply-chain compromise"](https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/) (21 Oct 2026, Microsoft Threat Intelligence) | 21 Oct 2026 | Storm-0940 (Microsoft's own China-state-aligned attribution) ran a cross-tenant spray from 144.172.64[.]99 — 327 accounts, 8 orgs, 151,000+ failed logons, **no successful authentication** — and separately attempted ARTEX-driven sign-ins against **>2,250 tenants**. Confirms "<3 hours" initial-access-to-privileged-access in multiple instances. Confirms the OAuth-app-credential-add technique (B6) and self-assignment of Global Administrator. |
| [Huntress — "ConnectWise (SkyKick) Vendor Credentials Abused in M365 Attacks"](https://www.huntress.com/blog/vendor-credential-enables-unauthorized-access) (16 Oct 2026, updated through 27 Oct) | 16 Oct 2026 | Confirms the ConnectWise Cloud Backup API secret compromise independently of Microsoft. Gives its own timeline: last confirmed sign-in with the credential at 08:44 UTC on 15 Oct; a Global-Admin-grant-to-secret-addition gap of **~2 minutes** in one tenant pair — faster than the "sub-3-hour" aggregate Microsoft cites. A 17 Oct update states **~1,000 Entra tenants** saw sign-ins from 144.172.64[.]128, with **300+ succeeding**. |
| [Wiz — "ARTEX in the Wild: Storm-0940 Expands Targeting of IT and Software Vendors"](https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors) (22 Oct 2026, Sapir Federovsky & Rami McCarthy) | 22 Oct 2026 | Supplies the specific B1/B2 probe paths the pack's rules are modeled on: `/zzz-nonexistent-xyz123` (the "baseline probe" before fuzzing), `/swagger/?format=openapi`, `/sso?jwt=eyJhbGciOiJub25lIn0...` (unsigned-JWT abuse), `/api/users/generate_redirect_link?role=admin`. Confirms >60 SSO route variants probed in two minutes and >15 rotating user-agents, including `norma/0.4` and `ReconProbe/1.0`. Wiz itself warns **"many of the provided user agents are not unique to this actor"** — reinforcing the pack's own D0 durability warning. |

**Nuance for the analyst:** Microsoft's "2,250 tenants" is **sign-in attempts**, not breaches — the majority failed. Huntress's "~1,000 tenants / 300+ successes" and Microsoft's "327 accounts / 0 successes" are **two different measurements** (general ConnectWise-credential sign-ins vs. one specific cross-tenant spray) and should not be merged into a single number when briefing stakeholders. Also notable: **Huntress's own article never names Storm-0940 or ARTEX** — that attribution was added only in Huntress's 27 Oct update, citing Microsoft. If you're tracing attribution lineage, Microsoft is the primary source; Huntress is a corroborating secondary one.

## 2. Sources backing Case Review 3B (South Korean financial sector)

| Source | Date | What it independently confirms |
|---|---|---|
| [CrowdStrike — "Unknown Threat Actor Uses AI-Driven ARTEX to Target South Korean Finance"](https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/) (7 Oct 2026, Counter Adversary Operations) | 7 Oct 2026 | Activity window "late September to early October 2026." Actor **unnamed**, assessed "likely Chinese-speaking, financially motivated" at **moderate** confidence only. Confirms the DeepSeek v4.1-flash backend and the `xcai[.]pro` LLM-reseller domain, plus the open-directory ARTEX host 38.244.50[.]120 (port 18899 serving `CLAUDE.md`) and nine proxy IPs. **CrowdStrike's own text states "the number of organizations affected remains unconfirmed"** — any victim count elsewhere does not come from this source. |
| [Zscaler ThreatLabz — "Inside a Global AI-Assisted Hacking Campaign"](https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign) (21 Oct 2026, Seongsu Park) | 21 Oct 2026 | Confirms the work-server IPs (23.248.226.250–254), jump server 38.244.50.120, proxy-pool API 154.201.79.246:443, and upstream SOCKS 205.214.59.31:443 / 161.8.161.140:11005. **Important scope nuance: Zscaler frames this as a global campaign across 19 countries and multiple sectors (dating, SMS, crypto, retail), not a Korea-only operation** — the "South Korean Financial Sector" framing in Case 3B is MMD's narrower slice of a broader Zscaler finding. Zscaler's prompt-history window (4 Aug–4 Oct 2026) also starts earlier than CrowdStrike's "late Sep" estimate. |

**Nuance for the analyst:** the two vendors only overlap on three IPs (38.244.50.120, 154.201.79.246, 205.214.59.31) and **disagree on that shared IP's role** — CrowdStrike calls 38.244.50[.]120 the "ARTEX host," Zscaler calls it the "jump server." Treat vendor-assigned infrastructure *roles* as interpretive, not as settled fact, even when the IP itself is corroborated. Also: neither vendor gives a confirmed victim-organization or affected-record count — the pack's own "~7–8 institutions / ~67–68k people" figures (Appendix F) trace to Korean press reporting cited by CrowdStrike, not to CrowdStrike's or Zscaler's own counts.

## 3. Sources behind Case Review 3C (Japanese breach wave) and the Section 2 input documents

| Source | Date | What it independently confirms |
|---|---|---|
| [JPCERT/CC Advisory JPCERT-AT-2026-0030](https://www.jpcert.or.jp/at/2026/at260030.html) (published 8 Oct 2026, updated 23 Oct 2026; Japanese-language) | 8/23 Oct 2026 | This is the direct primary source for Case 3C. Its **Case E, added in the 23 Oct update**, explicitly names three AI-pentest frameworks — **"Cairn, CyberStrikeAI, ARTEX"** — and states the attack technique targeted **internet-facing WSUS servers**, with 9 listed IPs (31.59.92.155, 31.56.206.187, 38.244.50.120, 46.8.227.73, 64.44.24.195, 94.183.184.98/.254, 95.182.99.112, 176.65.144.71). Note: **38.244.50.120 is the same IP CrowdStrike and Zscaler attribute to the Korea campaign** — either shared/reassigned multi-origin infrastructure (consistent with the pack's own "never auto-block, multi-origin hosting" warning) or a genuine cross-campaign link worth flagging to the analyst as a specific pivot. Cases A–D in the same advisory (vulnerability scanning, API abuse, a Metabase SQLi CVE, a JSP webshell) predate the AI-agent framing and are not ARTEX-specific. |
| [Eye Security Labs — "Machine speed: how we defended against an AI-driven attack" (Part 1)](https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/) (12 Nov 2026, Bas van den Berg) | 12 Nov 2026 | This is **not** a Japan-specific source — the MMD post actually cites it as one of the six **Section 2 input documents** ("MMD_artex_iocs.docx... Eye Security + CTI report"), not as Case 3C evidence. It is independently valuable: a full incident narrative naming ARTEX explicitly, with a distinct, non-overlapping IOC set — domains `artex.lol`, `staging.artex.lol`, `artex.tokena2b.top`, `artex.ok7777.eu.cc`, `artex.noog2infra.site`; UAs `Mozilla/5.0 (ARTEX)` and `Mozilla/5.0 (compatible; ArtExVerify/1.0)`; webshell header `X-ARTX-KEY`; markers `### ARTX-AGENT` and `ARTEX_WRITE_PROBE`. TTPs include reGeorg/Neo-reGeorg tunnels, a Git-triggered malicious Kubernetes pod, an `nsenter` host-namespace escape, and self-hosted-runner registration — the `nsenter`-based escape specifically echoes the DIVD/Zammad case in the pack's Appendix F chronology (24 Sep 2026 entry), suggesting this host-escape technique recurs across unrelated ARTEX incidents rather than being tied to one actor. It is explicitly marked **"Part 1" of a series** — a Part 2/3 may exist or be published later and would be worth checking again. |

## 4. Repository and community links

| Link | Status | Notes |
|---|---|---|
| [GitHub — unixfreaxjp/malwaremustdie, artex-stix-osquery-sigma-kql-snort-suricata-yara](https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara) | **Confirmed live**, matches the pack's self-description | Top-level folders: `iocs`, `kql`, `osquery`, `sigma`, `snort3`, `stix`, `suricata7`, `tests`, `tools`, `yara`; files `CHANGELOG.md`, `LICENSE`, `MANIFEST.sha256`, `README.md`, `Testing.md`. README confirms version **1.1.4**, dated 16 Nov 2026, first release 1.0.0-rc1 on 13 Nov 2026 — matches this package's own metadata. The repo's listed T3 Storm-3358 indicator is **178.16.54.129**, not .12 (see next row). |
| [URLhaus host page for 178.16.54.12](https://urlhaus.abuse.ch/host/178.16.54.12/) | **Inconclusive** — returned a bot-verification interstitial, no data retrievable by automated fetch | The MMD post links to this exact `.12` address, but the pack's own repo README and Microsoft's Nov 6 blog both list **178.16.54.129** (first seen 2026-10-19, "ARTEX server," Storm-3358) as the actual T3 indicator — a different host in the same `/24`. Both addresses fall inside the `178.16.54.0/24` segment the pack already flags as **alert-only, never auto-block (multi-origin hosting)**, so this is consistent with — not contradictory to — the pack's own shared-infrastructure warning, but the two specific addresses should not be treated as interchangeable. An analyst who needs the actual URLhaus tags/history for `.12` should query the URLhaus API or bulk feed directly rather than the browser-walled page. |
| [Reddit — r/netsec discussion thread](https://old.reddit.com/r/netsec/comments/1xv53bm/sharing_artex_detection_signature_pack_from_ioc/) | **Fetch blocked** by the tool (`old.reddit.com` is refused) | Could not verify comment count, upvotes, or whether the community raised corrections/false-positive reports. If community feedback on the pack matters to your investigation, check this thread manually. |

## 5. Key corrections and nuances to carry into the investigation

- **Attempted ≠ compromised.** Microsoft's 2,250-tenant figure and the 327-account spray both failed in the majority/entirety of attempts. Don't let a large number in a vendor headline inflate your read of actual impact.
- **Different vendors measure different things on the same incident.** Huntress's sign-in/success counts and Microsoft's spray counts are not reconcilable into one number — they describe different activity windows and different data sources.
- **The same IP can be labeled differently by different vendors** (38.244.50[.]120: "ARTEX host" per CrowdStrike, "jump server" per Zscaler, and also present in JPCERT/CC's Japan advisory). Treat vendor-assigned infrastructure *roles* as interpretation, and treat the **recurrence of one IP across three unrelated advisories** as exactly the kind of infrastructure-pivot signal Section 7 of the main briefing asks you to look for — but confirm independently before assuming it's the same campaign rather than reassigned shared hosting.
- **Zscaler's Korea-linked report is explicitly global in scope** (19 countries); the pack's "South Korean Financial Sector" case review is a deliberately narrowed slice of it.
- **JPCERT/CC's Case E adds a target surface not spelled out in the pack's Case 3C narrative: internet-facing WSUS servers.** This is a concrete, actionable pivot worth hunting for directly.
- **Eye Security's report is a sourcing document, not Case 3C evidence** — don't assume its domains/UAs apply to the Japan case; they're a separate, additional incident with their own fork-specific markers.
- **Primary vs. secondary attribution:** Huntress did not originally attribute the ConnectWise compromise to Storm-0940 or ARTEX; that link was added later, citing Microsoft. When tracing who-said-what-first, Microsoft is the anchor source for that specific attribution.
- Two items could not be independently verified with the tools available here (URLhaus's `.12` host data, and the Reddit thread's community feedback) — flagged above as open follow-ups rather than treated as confirmed either way.
