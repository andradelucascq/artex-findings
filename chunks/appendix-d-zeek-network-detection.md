---
doc_id: mmd-072-2026-artex
chunk_id: appendix-d
title: "Appendix D: (Additional) An idea on using Zeek to detect using released signatures in upper layers networking data-set (experimental stage)."
source_title: "MMD-072-2026 - ARTEX detection signature pack - from IOC w/playbook process"
source_url: https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html
author: "MalwareMustDie (@unixfreaxjp)"
published: "2026-11-13 (playbook v0.4; pack updated through v1.1.4 on 2026-11-16)"
retrieved: "2026-11-16"
tlp: "TLP:CLEAR"
pack_version: v1.1.4
tags: [zeek, network-detection, scripts, sumstats, intel-framework]
---

## Appendix D: (Additional) An idea on using Zeek to detect using released signatures in upper layers networking data-set (experimental stage).

It would be good for NetSec practitioner with AI tooling to use the signature packs for wider range detection, and Zeek is a natural place to start. Suricata and Snort match packets against fixed content, and the Sentinel KQL rules watch cloud identity logs, but there is a layer in between that neither covers well: the network *conversation* as a whole. Zeek turns every connection, DNS query, TLS handshake and HTTP transaction into structured logs, and its scripting language can keep state and count over time. That is exactly what the behavioral rules in this pack need, and the structured, tier-tagged notices it produces are easy for AI-assisted triage tooling to sort, so the confirmed (T1) alerts surface first and the generic (T4) noise stays in the background.

The idea below splits Zeek detection into two layers. Example 1 answers **"Who are you talking to?"** and Example 2 answers **"What are you saying, and how are you acting?"**. Before the details, here is what Zeek can and cannot see from this pack, so nobody expects it to do the job of the KQL or osquery parts.

| Part of the pack | Visible to Zeek? | Where else to detect it | 
|---|---|---|
| Past-incident IPs and FQDNs (T3), callback services (T4) | Yes: conn.log, dns.log, http.log Host, ssl.log SNI | Suricata/Snort IP and DNS rules, KQL watchlist | 
| Webshell header, ARTEX user-agents, webshell key, response marker (T1) | Yes, but only on cleartext HTTP/1.x (behind TLS termination) | Suricata/Snort content rules, Sigma webserver rules | 
| Rule B1 recon chain (T2) | Yes, with SumStats over a time window | Sigma correlation, KQL k06 | 
| Rule B2 auth probing (T2) | Partly (single requests); full chains are easier in KQL | Sigma, KQL k07/k08 | 
| ConnectWise app abuse, password spray against Entra ID (Storm-0940) | No: this traffic goes to Microsoft's cloud, not through your network | KQL k01-k03, Sigma cloud rules | 
| Host artifacts (backdoored images, accounts, dropped scripts) | No | osquery, YARA, Sigma Linux rules | 

### Example 1: Infrastructure & metadata layer (Layers 3, 4 and DNS/SNI)

**Core concept: "Who are you talking to?"** Example 1 uses Zeek's Intel Framework to check network routing details and unencrypted connection metadata (IP addresses in conn.log, DNS queries in dns.log, TLS Server Name Indication in ssl.log, HTTP Host headers and user-agents in http.log) against the pack's indicators, in memory, at line rate.

- **Why it is useful:** even when an operator hides commands inside HTTPS, your network still sees the destination IP, the DNS lookup for `xcai.pro` or `artex.lol`, and the server name in the TLS handshake. Matching IPs and names in memory is cheap and scales to high throughput.
- **Where it pays off most:** inbound traffic to your own web, VPN and API endpoints, and outbound traffic to `xcai.pro`, the LLM gateway used by ARTEX operators. Egress to it from inside your network suggests an operator is already inside.
- **Its limitation (why it is Tier 3):** operators rent shared VPS and route through public proxies. An IP used by ARTEX last week may belong to an innocent company next week, and JPCERT/CC itself warns that listed addresses may already be in legitimate use. Example 1 hits are therefore T3 (correlate before blocking), and the 178.16.54.0/24 abuse segment, VPN exits and generic user-agents are T4 (logged, never alerted on alone).
- **Blind spots:** DNS-over-HTTPS and TLS Encrypted ClientHello hide the name; and several important IOCs (the Storm-0940 spray and ConnectWise sign-ins) never cross your perimeter at all, they only show up in Entra ID logs.

### Example 2: Application & behavioral layer (Layer 7)

**Core concept: "What are you saying, and how are you acting?"** Example 2 uses Zeek's HTTP parser and the SumStats (summary statistics) engine to read inside web requests and track behavior over time.

- **Part A, exact content signatures (Tier 1):** the `X-ARTX-KEY` header, the `### ARTX-AGENT` response marker, the `ArtExVerify/1.0` and `Mozilla/5.0 (ARTEX)` user-agents and the `k=x9f3c2b7e` webshell key. The header and the response marker are framework-level (class A); the key belongs to one operator and the `(ARTEX)` user-agent to one fork (class B). All of them are unique to ARTEX or a known ARTEX operator, so they stay T1, but a different operator on a fresh IP will not necessarily send the class B ones.
- **Part B, behavioral sequencing with SumStats (Tier 2, Rule B1):** one request for a `.js.map` file or one 404 could be a normal user or crawler. One source pulling five or more distinct source maps, config files, Swagger/OpenAPI docs, backup archives (`.bak`, `.sql`, `.env`) and `zz-notexist-*` test paths within two minutes is the automated reconnaissance pattern of an AI pentest agent. Commodity scanners such as nuclei or ZAP produce the same pattern, which is why it is T2 (validate before containment). The same chain from the cross-campaign host 38.244.50\[.\]120 escalates to T1.
- **Its limitation:** Zeek must see cleartext HTTP, so the sensor belongs behind your reverse proxy, load balancer or TLS-decrypting packet broker. Zeek's built-in HTTP analyzer parses HTTP/1.x; the proxy-to-backend leg is usually HTTP/1.1, which makes it the best tap point.

### First draft of the scripts (concept)

These are the first draft scripts of this idea, kept here as they were written because the review notes below explain, line by line, why parts of them would stay silent. Please deploy the corrected version further down, not this draft.

```
#fields	indicator	indicator_type	meta.source	meta.desc
38.244.50.120	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] KR ARTEX host / JPCERT Case E jump server (escalate to T1 on B-rule hit)
144.172.64.99	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Storm-0940 password spray source
144.172.64.128	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Storm-0940 primary app sign-in source
207.5.195.93	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Eye Security ARTEX infrastructure
178.16.54.129	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Storm-3358 ARTEX/C2 server
artex.lol	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX family callback domain
xcai.pro	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] LLM reseller gateway used by ARTEX operators
```
```
@load base/frameworks/intel
@load frameworks/intel/seen
@load frameworks/intel/do_notice
redef Intel::read_files += {
    "/opt/zeek/share/zeek/site/artex_intel.tsv"
};
# Optional: Customize notice behavior so T3 matches log to notice.log
hook Notice::policy(n: Notice::Info)
    {
    if ( n$note == Intel::Notice && /MMD-072-2026/ in n$msg )
        {
        add n$actions[Notice::ACTION_LOG];
        }
    }
```
```
@load base/protocols/http
@load base/frameworks/notice
@load base/frameworks/sumstats
module ARTEX;
export {
    redef enum Notice::Type += {
        ## Tier 1: Confirmed ARTEX HTTP header, User-Agent, or webshell key
        Confirmed_HTTP_Indicator,
        ## Tier 2: Rule B1 automated API/config reconnaissance chain
        Behavioral_Recon_Chain
    };
    # Tier 1 patterns (Class A/B)
    const t1_ua_regex = /ArtExVerify\/1\.0|Mozilla\/5\.0 \(ARTEX\)/;
    const t1_uri_regex = /[\?&]k=x9f3c2b7e(&|$)/;
    # Tier 2 / Rule B1 recon patterns (Class C + B1 stages)
    const b1_recon_uri_regex = /(\.js\.map|\/env\.js|\/config\.json|\/swagger|\/openapi|\/zz-notexist-|\/artx-nonexistent|\.(bak|sql|env))(\?|$)/;
    
    # Threshold for B1 recon hits from a single source within 2 minutes
    const b1_threshold: double = 5.0;
    const b1_window: interval = 2mins;
}
# 1. Inspect HTTP Headers for X-ARTX-KEY (Tier 1)
event http_header(c: connection, is_orig: bool, original_name: string, name: string, value: string)
    {
    if ( is_orig && name == "X-ARTX-KEY" )
        {
        NOTICE([
            $note=Confirmed_HTTP_Indicator,
            $conn=c,
            $msg="[ARTEX-T1 CONFIRMED] Header X-ARTX-KEY detected. Indicator unique to ARTEX; treat as likely compromise.",
            $sub=fmt("Header value: %s", value),
            $identifier=cat(c$id$orig_h)
        ]);
        }
    }
# 2. Inspect HTTP Requests for Tier 1 UAs/URIs and feed Rule B1 SumStats
event http_request(c: connection, method: string, original_URI: string, unescaped_URI: string, version: string)
    {
    local orig = c$id$orig_h;
    # Check Tier 1 User-Agent
    if ( c$http?$user_agent && t1_ua_regex in c$http$user_agent )
        {
        NOTICE([
            $note=Confirmed_HTTP_Indicator,
            $conn=c,
            $msg=fmt("[ARTEX-T1 CONFIRMED] ARTEX User-Agent detected: %s", c$http$user_agent),
            $identifier=cat(orig, c$http$user_agent)
        ]);
        }
    # Check Tier 1 Webshell Key in URI
    if ( t1_uri_regex in unescaped_URI )
        {
        NOTICE([
            $note=Confirmed_HTTP_Indicator,
            $conn=c,
            $msg=fmt("[ARTEX-T1 CONFIRMED] ARTEX webshell key parameter in URI: %s", unescaped_URI),
            $identifier=cat(orig, unescaped_URI)
        ]);
        }
    # Feed Rule B1 Recon Chain tracker
    if ( b1_recon_uri_regex in unescaped_URI )
        {
        SumStats::observe("artex.b1.recon", [$host=orig], [$str=unescaped_URI]);
        }
    }
# 3. Initialize SumStats for Rule B1 Behavioral Correlation
event zeek_init()
    {
    local r1 = SumStats::Reducer($stream="artex.b1.recon", $apply=set(SumStats::UNIQUE));
    SumStats::create([
        $name="artex-b1-recon-detection",
        $epoch=b1_window,
        $reducers=set(r1),
        $threshold=b1_threshold,
        $threshold_val(key: SumStats::Key, result: SumStats::Result) =
            {
            return result["artex.b1.recon"]$unique + 0.0;
            },
        $threshold_crossed(key: SumStats::Key, result: SumStats::Result) =
            {
            NOTICE([
                $note=Behavioral_Recon_Chain,
                $src=key$host,
                $msg=fmt("[ARTEX-T2 BEHAVIOR] Rule B1 recon chain: %s requested %d distinct config/API/probe URIs in %s.",
                         key$host, result["artex.b1.recon"]$unique, b1_window),
                $identifier=cat(key$host)
            ]);
            }
    ]);
    }
```
### Review notes: what was corrected

1. **The Intel notice hook never fired.** The Intel Framework only raises `Intel::Notice` when the feed has a `meta.do_notice` column set to T, and its notice text does not contain `meta.source`, so `/MMD-072-2026/ in n$msg` could never match. The corrected loader handles `Intel::match` directly and raises its own notice, using the tier from `meta.desc`: T1/T3 notify, T4 stays in intel.log only.
2. **Intel DOMAIN matching is exact.** `artex.lol` does not match `staging.artex.lol`. The corrected feed lists every FQDN, and the loader adds a DNS suffix check for the operator domains and for `artx*.dnslog.cn` callbacks.
3. **The user-agent check could not work in `http_request`.** That event fires before headers are parsed, so `c$http$user_agent` is not set yet. It moved into the `http_header` handler (Zeek upper-cases header names, so `"USER-AGENT"`).
4. **The B1 regex missed most real requests.** The trailing `(\?|$)` applied to every alternative, so `/zz-notexist-8812`, `/swagger/`, `/swagger/index.html`, `/openapi.json` and `/zzz-nonexistent-xyz123` never matched (5 of 9 test URLs). Only the file-extension alternatives are anchored now; all 9 match and the benign samples do not.
5. **Response status was not available.** The playbook counts 2xx backup downloads and 404 probes, so B1 is now fed from `HTTP::log_http`, which carries `status_code`.
6. **Additions:** the `### ARTX-AGENT` response marker (server-side body check), T1 escalation for 38.244.50\[.\]120, the complete tiered feed (43 IPs, 7 FQDNs, 19 user-agents) and the 178.16.54.0/24 segment as a separate optional T4 feed.

### Corrected scripts (use these)

The feed is generated from the same tiered IOC list as the pack (`iocs/mmd-artex-iocs.csv`), so tier, role, first-seen date and source travel with every indicator. Fields are separated by literal tab characters, which are preserved in the block below for copy and paste. The bare `Mozilla/5.0` user-agent is left out on purpose: as an exact-match Intel item it would flood intel.log.

```
#fields	indicator	indicator_type	meta.source	meta.desc	meta.url
207.5.195.93	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] VPS hosting ARTEX instances | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
artex.lol	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator domain | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
staging.artex.lol	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator domain | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
artex.tokena2b.top	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator domain | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
artex.ok7777.eu.cc	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator domain | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
artex.noog2infra.site	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator domain | Eye Security observed ARTEX intrusion | first seen 2026-11-12 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
144.172.64.99	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Password-spray source 10-14 Oct (no successful logons); later related activity | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-10 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
144.172.64.128	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Primary ConnectWise app-only sign-in source, 15 Oct 06:13-08:34 UTC; combined with ConnectWise app ID = T1 | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-15 | ConnectWise / Huntress	https://www.huntress.com/blog/vendor-credential-enables-unauthorized-access
144.172.64.60	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Related sign-in and Graph activity | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-15 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
187.15.88.186	Intel::ADDR	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Transient VPN exit 15-16 Oct; shared, high collateral | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-15 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
187.15.90.217	Intel::ADDR	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Transient VPN exit 15-16 Oct; shared, high collateral | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-15 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
187.15.89.186	Intel::ADDR	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Transient VPN exit 15-16 Oct; shared, high collateral | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-15 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
104.194.150.21	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Wiz-observed node targeting IT/software vendors | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-01 | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
167.88.172.97	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Wiz-observed node targeting IT/software vendors | Storm-0940 ConnectWise Cloud Backup campaign | first seen 2026-10-01 | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
38.244.50.120	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX host / jump server (KR); also on JPCERT/CC Case E list. Any behavioral-rule hit from this IP escalates to T1 | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
101.53.80.20	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
205.214.59.31	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy / upstream SOCKS :443 (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
124.155.252.63	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
154.201.79.246	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy / proxy-pool API :443 (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
23.248.249.90	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
23.158.220.98	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
103.248.148.84	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
203.160.133.172	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
209.209.85.38	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator proxy (shared) | Korean financial-sector ARTEX campaign | first seen 2026-09-25 | CrowdStrike	https://www.crowdstrike.com/en-us/blog/unknown-threat-actor-uses-artex-to-target-south-korean-finance/
23.248.226.250	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator work server | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
23.248.226.251	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator work server | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
23.248.226.252	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator work server | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
23.248.226.253	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator work server | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
23.248.226.254	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Operator work server | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
161.8.161.140	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Upstream SOCKS proxy (port 11005) | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
xcai.pro	Intel::DOMAIN	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] LLM API reseller/gateway used as ARTEX model backend; egress from your own network suggests an operator inside | Korean financial-sector ARTEX campaign | first seen 2026-08-04 | Zscaler ThreatLabz	https://www.zscaler.com/blogs/security-research/inside-global-ai-assisted-hacking-campaign
31.59.92.155	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 4-20 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-10-04 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
31.56.206.187	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 27 Sep-20 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-09-27 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
46.8.227.73	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 27 Sep-19 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-09-27 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
64.44.24.195	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed from 15 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-10-15 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
94.183.184.98	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 13-19 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-10-13 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
94.183.184.254	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 3-20 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-10-03 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
95.182.99.112	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 4-19 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-10-04 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
176.65.144.71	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Observed 23 Sep-17 Oct (JST); may now be legitimately used per JPCERT/CC | Japanese breach wave, JPCERT/CC Case E | first seen 2026-09-23 | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
199.19.95.66	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] WISP C2 node | Storm-3358 financial-sector campaign | first seen 2026-11-02 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
199.19.95.67	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] WISP C2 node | Storm-3358 financial-sector campaign | first seen 2026-11-02 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
199.19.95.68	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] WISP C2 node | Storm-3358 financial-sector campaign | first seen 2026-11-02 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
91.92.240.238	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX server | Storm-3358 financial-sector campaign | first seen 2026-11-04 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
85.11.168.45	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX server | Storm-3358 financial-sector campaign | first seen 2026-11-04 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
85.11.167.52	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] C2 server | Storm-3358 financial-sector campaign | first seen 2026-11-04 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
178.16.54.129	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX server (Microsoft IOC table value) | Storm-3358 financial-sector campaign | first seen 2026-10-19 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
31.77.229.35	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] ARTEX server | Storm-3358 financial-sector campaign | first seen 2026-10-28 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
178.15.54.129	Intel::ADDR	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Possible typo variant from Microsoft's hunting query; no threat record | Storm-3358 financial-sector campaign | first seen 2026-10-19 | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/11/06/artex-abuse-expands-as-cybercriminals-target-financial-sector/
199.19.77.84	Intel::ADDR	MMD-072-2026	[ARTEX-T3 PAST-INCIDENT IOC] Service-principal sign-ins with exposed secrets; Microsoft advises blocking /32 via Conditional Access | Exposed-secrets agentic campaign | first seen 2026-10-28 | Microsoft Threat Intelligence (exposed secrets, 5 Nov 2026)	https://www.microsoft.com/en-us/security/blog/
webhook.site	Intel::DOMAIN	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Public webhook capture service; legitimate uses common | Generic ARTEX tradecraft | first seen 2026-10-30 | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
Mozilla/5.0 (ARTEX)	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T1 CONFIRMED] User-Agent: Agent driving webshells (fork-specific per community source analysis) | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
Mozilla/5.0 (compatible; ArtExVerify/1.0)	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T1 CONFIRMED] User-Agent: Scripted WAF-bypass verification | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
Mozilla/5.0 (X11; Linux x86_64; rv:38.0) Gecko/20100101 Firefox/38.0	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Neo-reGeorg default UA; used by many actors | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
Python-urllib/3.14	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic library UA | Eye Security	https://labs.eye.security/how-we-defended-against-an-ai-driven-attack/
Python-urllib/3.12	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic library UA; also seen in ConnectWise persistence script | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
python-requests/2.34.2	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic library UA (Microsoft, Wiz, JPCERT/CC) | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
python-requests/2.31.0	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic library UA | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
python-requests/2.31	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic library UA (ConnectWise advisory form) | ConnectWise / Huntress	https://www.huntress.com/blog/vendor-credential-enables-unauthorized-access
curl/7.88.1	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Generic tool UA (Microsoft, Wiz, JPCERT/CC) | JPCERT/CC	https://www.jpcert.or.jp/at/2026/at260030.html
azure-toolkit/1.0	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Scripted Azure tooling UA | Microsoft Threat Intelligence	https://www.microsoft.com/en-us/security/blog/2026/10/21/storm-0940-uses-artex-agentic-ai-connectwise-supply-chain-compromise/
norma/0.4	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Norma agent SDK; shared by non-ARTEX agents | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
ReconProbe/1.0	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Attribution to Wiz not confirmed; likely model-generated | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64)	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Truncated browser UA | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Truncated browser UA | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/124.0	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Truncated browser UA | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/124.0 Safari/537.36	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Truncated browser UA | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Truncated browser UA | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0 Safari/537.36	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Malformed browser UA (no KHTML) | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/124.0 Safari/537.36	Intel::SOFTWARE	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] User-Agent: Malformed browser UA (no KHTML) | Wiz Research	https://www.wiz.io/blog/artex-in-the-wild-storm-0940-expands-targeting-of-it-and-software-vendors
```
```
#fields	indicator	indicator_type	meta.source	meta.desc	meta.url
178.16.54.0/24	Intel::SUBNET	MMD-072-2026	[ARTEX-T4 GENERIC - WARNING] Abuse segment around 178.16.54.129 (Spamhaus SBL/DROP neighbours, URLhaus lures); multi-origin hosting: alert only, never auto-block | Storm-3358 financial-sector campaign | first seen 2026-07-09 | Spamhaus / URLhaus (segment reputation)	https://urlhaus.abuse.ch/host/178.16.54.12/
```
```
# MalwareMustDie.org - ARTEX misuse detection pack - Zeek Intel loader (MMD-072-2026)
# Tiers come from meta.desc: T1/T3 raise a notice, T4 is logged to intel.log only.
@load base/frameworks/intel
@load base/frameworks/notice
@load frameworks/intel/seen
module ARTEX;
export {
    redef enum Notice::Type += {
        ## Intel feed hit (tier taken from the feed description)
        Intel_Hit,
        ## DNS lookup of an ARTEX callback name (suffix match, not covered by Intel exact matching)
        Callback_Domain
    };
    const feed_source = "MMD-072-2026" &redef;
}
redef Intel::read_files += {
    "/opt/zeek/share/zeek/site/artex/artex_intel.tsv",
    # Optional T4 abuse segment; needs Intel::SUBNET support in your Zeek version
    "/opt/zeek/share/zeek/site/artex/artex_intel_subnet_t4.tsv",
};
event Intel::match(s: Intel::Seen, items: set[Intel::Item])
    {
    for ( item in items )
        {
        if ( ! item$meta?$source || item$meta$source != feed_source )
            next;
        local desc = item$meta?$desc ? item$meta$desc : "";
        if ( /^\[ARTEX-T4/ in desc )
            next;   # T4 = enrichment only: stays in intel.log, no notice
        local n: Notice::Info = [$note=Intel_Hit,
                                 $msg=fmt("%s (seen in %s)", desc, s$where),
                                 $sub=s$indicator,
                                 $identifier=cat(s$indicator, s$where)];
        if ( s?$conn )
            n$conn = s$conn;
        NOTICE(n);
        }
    }
# Intel DOMAIN matching is exact, so subdomains and callback services need a suffix check.
event dns_request(c: connection, msg: dns_msg, query: string, qtype: count, qclass: count)
    {
    local q = to_lower(query);
    if ( /(^|\.)(artex\.lol|artex\.tokena2b\.top|artex\.ok7777\.eu\.cc|artex\.noog2infra\.site|xcai\.pro)$/ in q )
        NOTICE([$note=Callback_Domain, $conn=c,
                $msg=fmt("[ARTEX-T3 PAST-INCIDENT IOC] DNS lookup of ARTEX operator/LLM gateway name %s. May be reassigned; correlate with T1/T2 before blocking.", q),
                $identifier=cat(c$id$orig_h, q)]);
    else if ( /\.dnslog\.cn$/ in q && /artx/ in q )
        NOTICE([$note=Callback_Domain, $conn=c,
                $msg=fmt("[ARTEX-T2 BEHAVIOR] ARTEX diagnostic callback %s. Validate source and scope before containment.", q),
                $identifier=cat(c$id$orig_h, q)]);
    }
```
```
# MalwareMustDie.org - ARTEX misuse detection pack - Zeek HTTP detection (MMD-072-2026)
# Needs cleartext HTTP/1.x: tap behind the TLS-terminating reverse proxy / load balancer.
# Event signatures as in Zeek 5.x-7.x (older Zeek: drop original_name from http_header).
@load base/protocols/http
@load base/frameworks/notice
@load base/frameworks/sumstats
module ARTEX;
export {
    redef enum Notice::Type += {
        ## T1: ARTEX webshell header, ARTEX user-agent, operator webshell key or response marker
        Confirmed_HTTP_Indicator,
        ## T2 (T1 from known cross-campaign hosts): Rule B1 automated recon chain
        Behavioral_Recon_Chain
    };
    # T1 patterns (class A/B per playbook)
    const t1_ua_regex = /ArtExVerify\/1\.0|^Mozilla\/5\.0 \(ARTEX\)$/ &redef;
    const t1_uri_regex = /[?&]k=x9f3c2b7e(&|$)/ &redef;
    const t1_body_marker = "### ARTX-AGENT" &redef;
    # B1 recon stages: only the file-extension alternatives are anchored
    const b1_recon_uri_regex = /\.js\.map|\/env\.js|\/config\.json|\/swagger|\/openapi|\/api-docs|\/zz-?notexist|\/artx-nonexistent|\/zzz-nonexistent|\.(bak|old|sql|env|zip|7z)(\?|$)/ &redef;
    const b1_threshold: double = 5.0 &redef;
    const b1_window: interval = 2mins &redef;
    # Any B1 hit from these hosts escalates to T1 (playbook 3B.2)
    const escalate_hosts: set[addr] = { 38.244.50.120 } &redef;
}
# 1. Headers: X-ARTX-KEY and ARTEX user-agents (headers are parsed after http_request)
event http_header(c: connection, is_orig: bool, original_name: string, name: string, value: string)
    {
    if ( ! is_orig )
        return;
    if ( name == "X-ARTX-KEY" )
        NOTICE([$note=Confirmed_HTTP_Indicator, $conn=c,
                $msg="[ARTEX-T1 CONFIRMED] X-ARTX-KEY webshell header. Indicator unique to ARTEX; treat as likely compromise.",
                $sub=fmt("value=%s", value),
                $identifier=cat(c$id$orig_h, "x-artx-key")]);
    else if ( name == "USER-AGENT" && t1_ua_regex in value )
        NOTICE([$note=Confirmed_HTTP_Indicator, $conn=c,
                $msg=fmt("[ARTEX-T1 CONFIRMED] ARTEX user-agent: %s", value),
                $identifier=cat(c$id$orig_h, value)]);
    }
# 2. URI: operator webshell key
event http_request(c: connection, method: string, original_URI: string, unescaped_URI: string, version: string)
    {
    if ( t1_uri_regex in unescaped_URI )
        NOTICE([$note=Confirmed_HTTP_Indicator, $conn=c,
                $msg=fmt("[ARTEX-T1 CONFIRMED] ARTEX webshell key in URI: %s", unescaped_URI),
                $identifier=cat(c$id$orig_h, "k-param")]);
    }
# 3. Response body: webshell output marker (may be missed if split across chunks)
event http_entity_data(c: connection, is_orig: bool, length: count, data: string)
    {
    if ( ! is_orig && t1_body_marker in data )
        NOTICE([$note=Confirmed_HTTP_Indicator, $conn=c,
                $msg="[ARTEX-T1 CONFIRMED] ARTEX webshell response marker ### ARTX-AGENT",
                $identifier=cat(c$id$resp_h, "artx-agent")]);
    }
# 4. Rule B1: feed SumStats once the response status is known (2xx downloads, 404 probes)
event HTTP::log_http(rec: HTTP::Info)
    {
    if ( ! rec?$uri || ! rec?$status_code )
        return;
    if ( b1_recon_uri_regex !in rec$uri )
        return;
    if ( rec$status_code == 404 || (rec$status_code >= 200 && rec$status_code < 300) )
        SumStats::observe("artex.b1.recon", [$host=rec$id$orig_h], [$str=rec$uri]);
    }
event zeek_init()
    {
    local r1 = SumStats::Reducer($stream="artex.b1.recon", $apply=set(SumStats::UNIQUE));
    SumStats::create([
        $name="artex-b1-recon-detection",
        $epoch=b1_window,
        $reducers=set(r1),
        $threshold=b1_threshold,
        $threshold_val(key: SumStats::Key, result: SumStats::Result) =
            {
            return result["artex.b1.recon"]$unique + 0.0;
            },
        $threshold_crossed(key: SumStats::Key, result: SumStats::Result) =
            {
            local tier = key$host in escalate_hosts ? "[ARTEX-T1 CONFIRMED]" : "[ARTEX-T2 BEHAVIOR]";
            NOTICE([$note=Behavioral_Recon_Chain,
                    $src=key$host,
                    $msg=fmt("%s Rule B1 recon chain: %s requested %d distinct config/API/probe URIs in %s. Commodity scanners also trigger this; validate before containment.",
                             tier, key$host, result["artex.b1.recon"]$unique, b1_window),
                    $identifier=cat(key$host)]);
            }
    ]);
    }
```
### Deployment notes

1. Copy the four files to `/opt/zeek/share/zeek/site/artex/`, add `@load ./artex/artex_intel.zeek` and `@load ./artex/artex_http_detection.zeek` to `local.zeek`, then run `zeekctl check` and `zeekctl deploy`.
2. Run the HTTP script on a sensor that sees decrypted traffic (behind TLS termination). The Intel part works on any sensor.
3. Tune `ARTEX::b1_threshold` and `ARTEX::b1_window` with `redef` for your traffic; allow-list your own scanners and any authorized ARTEX testing.
4. The scripts use Zeek 5.x-7.x event signatures (on older Zeek, drop `original_name` from `http_header`). SumStats works the same on a Zeek cluster.
5. Status: experimental. The regex logic was tested against sample URLs, but these scripts have not yet been run on every Zeek version. Please test them on your own build and traffic first, and report false or true positives on the GitHub repository so they can be tuned like the rest of the pack.

malwaremustdie.org - @unixfreaxjp 20261113v0.4rk
