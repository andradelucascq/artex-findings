---
doc_id: mmd-072-2026-artex
doc_type: index
title: "MMD-072-2026 — ARTEX detection signature pack (from IOC w/ playbook process)"
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
blog: "Malware Must Die!"
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
github_repo: "https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara"
license_note: "Content copyright MalwareMustDie; archived here for personal/defensive reference under TLP:CLEAR public-release terms. Not for redistribution as original work."
---

# MMD-072-2026 — ARTEX detection signature pack

Local archive of the **MalwareMustDie** (TLP:CLEAR) blog post about the
behavior-first detection pack against misuse of **ARTEX**, an LLM-operated
autonomous pentest framework. Content saved in full and reorganized into
blocks (chunks) with YAML metadata for LLM indexing/RAG.

- **Original source:** https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
- **Author:** MalwareMustDie (@unixfreaxjp)
- **Published:** 13 Nov 2026 (playbook v0.4) — pack updated through v1.1.4 on 16 Nov 2026
- **Retrieved:** 16 Nov 2026
- **Classification:** TLP:CLEAR (public release)
- **Signature repository:** https://github.com/unixfreaxjp/malwaremustdie/tree/master/etc/artex-stix-osquery-sigma-kql-snort-suricata-yara

## Directory structure

```
artex_mmd072_2026/
├── README.md                        <- this file (index + document metadata)
├── ARTEX_investigation_briefing.md  <- RECOMMENDED ATTACHMENT: CSIRT-style intro
│                                        for the agent/analyst + the full source
│                                        content (verbatim). Single file, ready to
│                                        attach as investigation context.
├── _chunk_index.md                   <- simple auto-generated index (relative links)
├── chunks/                           <- full content, split by section, 1 file per chunk
│   └── NN-slug.md                      each with YAML frontmatter (doc_id, chunk_id, title, tags...)
└── raw/                               <- unedited raw material
    ├── artex.html                      original HTML downloaded via curl
    ├── artex_full_raw.md               full markdown extraction (trafilatura), unsplit
    └── briefing_intro_source.md        editable source for the briefing's intro (see below)
```

### Which file should I use?

- **Attaching context for an agent/LLM investigating an incident?**
  Use [`ARTEX_investigation_briefing.md`](ARTEX_investigation_briefing.md).
  It has: (1) an English intro written as a CSIRT briefing, covering
  what ARTEX is, why the document matters, a signal→rule→tier→durability
  correlation map (treated as canonical examples, not an exhaustive list),
  the attack-chain narrative common to the case reviews, how to explain a
  correlation when the agent finds one (heuristics, not a rigid checklist —
  containment/action logic stays with the human analyst), a **critical-
  thinking and pivoting question bank** (Section 7: bias-check questions,
  infrastructure/identity/tooling pivoting, kill-chain reconstruction,
  cross-campaign attribution, lateral-movement blast radius, concrete next
  places to look), and the caveats to communicate to the analyst; followed
  by (2) the **complete, verbatim English source content**, untranslated,
  delimited by `<document>`/`<source>`/`<document_content>` tags.
  The intro's standalone, editable source lives at
  [`raw/briefing_intro_source.md`](raw/briefing_intro_source.md).
- **Indexing into a RAG / vector store?** Use `chunks/` (one chunk per
  section, with YAML metadata per block).
- **Want the plain text with no additions at all?** Use
  `raw/artex_full_raw.md`.

- `raw/artex_full_raw.md` has the full text in a single file, exactly as
  extracted (no summarizing, no trimming) — use this if you want the whole
  document at once with nothing added.
- `chunks/*.md` has the **same full content**, split by section (the `##`
  level of the original post) with per-block YAML frontmatter, meant for
  ingestion into a vector index / RAG (each chunk is self-contained, with
  `doc_id`, `chunk_id`, `title`, `tags`, and the source URL).
- Nothing was summarized or rewritten: splitting into chunks only cuts the
  text at section boundaries and adds metadata; each chunk's body is a
  literal copy of the text extracted from the post.

## Section index (original document order)

| # | Section | File |
|---|---|---|
| 00 | Start here (front matter, "at a glance" table, prologue, signature-download table) | [chunks/00-front-matter.md](chunks/00-front-matter.md) |
| 01 | Purpose and Principles (the playbook's 5 principles) | [chunks/01-purpose-and-principles.md](chunks/01-purpose-and-principles.md) |
| 02 | Source Document Review (the 6 compared sources, attribution conflicts) | [chunks/02-source-document-review.md](chunks/02-source-document-review.md) |
| 03 | Decision Workflow Overview | [chunks/03-decision-workflow-overview.md](chunks/03-decision-workflow-overview.md) |
| 3A | Case Review — ConnectWise Cloud Backup / Storm-0940 | [chunks/03a-case-review-connectwise-storm-0940.md](chunks/03a-case-review-connectwise-storm-0940.md) |
| 3B | Case Review — South Korean financial sector | [chunks/03b-case-review-south-korea-financial.md](chunks/03b-case-review-south-korea-financial.md) |
| 3C | Case Review — Japanese breach wave (JPCERT/CC) | [chunks/03c-case-review-japan-breach-wave.md](chunks/03c-case-review-japan-breach-wave.md) |
| 04 | Source Verification Levels | [chunks/04-source-verification-levels.md](chunks/04-source-verification-levels.md) |
| 05 | Uniqueness Classes | [chunks/05-uniqueness-classes.md](chunks/05-uniqueness-classes.md) |
| 06 | Tier Assignment and Alert Wording (T1–T4) | [chunks/06-tier-assignment-and-alert-wording.md](chunks/06-tier-assignment-and-alert-wording.md) |
| 07 | Format Routing (Sigma/KQL/Suricata/Snort/osquery/YARA/STIX) | [chunks/07-format-routing.md](chunks/07-format-routing.md) |
| 08 | Combination and Matching Design (rules B1–B6) | [chunks/08-combination-and-matching-design.md](chunks/08-combination-and-matching-design.md) |
| 09 | Exclusions and HOLD Queue | [chunks/09-exclusions-and-hold-queue.md](chunks/09-exclusions-and-hold-queue.md) |
| 10 | Lifetime and Versioning | [chunks/10-lifetime-and-versioning.md](chunks/10-lifetime-and-versioning.md) |
| 11 | Fine-Tuning and Testing | [chunks/11-fine-tuning-and-testing.md](chunks/11-fine-tuning-and-testing.md) |
| 12 | Packaging and Publication Checks | [chunks/12-packaging-and-publication-checks.md](chunks/12-packaging-and-publication-checks.md) |
| A | Appendix A — Decision Record for the Current IOC Set | [chunks/appendix-a-decision-record.md](chunks/appendix-a-decision-record.md) |
| B | Appendix B — Reviewer Checklist (per indicator) | [chunks/appendix-b-reviewer-checklist.md](chunks/appendix-b-reviewer-checklist.md) |
| C | Appendix C — Change Log | [chunks/appendix-c-change-log.md](chunks/appendix-c-change-log.md) |
| D | Appendix D — Zeek-based detection (full scripts, Intel Framework + SumStats) | [chunks/appendix-d-zeek-network-detection.md](chunks/appendix-d-zeek-network-detection.md) |
| E | Appendix E — Addendum, 14 Nov 2026 (pack v1.1.0: results, known gaps) | [chunks/appendix-e-addendum-v1-1-0.md](chunks/appendix-e-addendum-v1-1-0.md) |
| F | Appendix F — Activity Chronology (Jul–Nov 2026, all public campaigns) | [chunks/appendix-f-activity-chronology.md](chunks/appendix-f-activity-chronology.md) |
| G | **Supplementary, not part of the original post** — External Source Verification: every externally cited link (Microsoft, Huntress, Wiz, CrowdStrike, Zscaler, JPCERT/CC, Eye Security, the GitHub repo, URLhaus, Reddit) fetched and cross-checked against the case reviews, with discrepancies/nuances flagged | [chunks/appendix-g-external-source-verification.md](chunks/appendix-g-external-source-verification.md) |

## Notes on the content (not indicators, just archival notes)

- The post **does not publish a binary hash** (the backdoored-binary hash
  is on HOLD) and **does not show YARA rule text**, only describes
  coverage — that is faithful to the original, not an omission by this
  copy.
- IPs and domains (`artex.lol`, `xcai.pro`, T3 feeds) are short-lived
  context indicators (D0), not the detection itself — the author
  repeatedly warns against auto-blocking shared infrastructure (proxies,
  VPS, VPN).
- The only complete code listings in the post are the **Zeek scripts**
  (Appendix D); the Sigma/Suricata/Snort/YARA/KQL/osquery/STIX rules live
  in the linked GitHub repository, not in the post body.
- `raw/artex.html` is kept as a raw backup for future re-checking or
  re-conversion.
