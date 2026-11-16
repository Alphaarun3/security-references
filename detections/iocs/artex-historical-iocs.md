# ARTEX historical indicators

Defensive reference only. **These are historical indicators from public reporting, not a block list.** Infrastructure is reassigned and
re-registered, so vet every item against the original source, correlate with behaviour, and review before blocking.

Sources:
- FBI FLASH-20261113-001 (public advisory)
- [MalwareMustDie MMD-072-2026](https://blog.malwaremustdie.org/2026/11/mmd-072-2026-artex-detection-signature.html), a community detection pack published 13 Nov 2026. It is a secondary source built on vendor reports, and its author marks several items as unverified. Check the original vendor reports it cites before relying on an item.

How MalwareMustDie tiers indicators (summarised): T1 confirmed compromise, T2 behaviour (validate first), T3 past-incident infrastructure (correlate before blocking),
T4 generic or low-confidence (never alert alone). Their IP indicators alert only until 2027-02-11 and are hunt-only until 2027-11-13; behavioural rules do not expire.

## Domains (defanged)

| Indicator | Note | Tier (MMD) |
|-----------|------|------------|
| xcai[.]pro | LLM reseller gateway; also named in the FBI FLASH | T3 |
| artex[.]lol, staging[.]artex[.]lol | Reported ARTEX hosts | T3 |
| artex[.]tokena2b[.]top, artex[.]ok7777[.]eu[.]cc, artex[.]noog2infra[.]site | Reported ARTEX hosts | T3 |
| artx*[.]dnslog[.]cn | Callback pattern | T2 |
| webhook[.]site, *[.]oast[.]me, *[.]dnslog[.]cn | Common testing/callback services, used by many legitimate users | T4 (enrichment only) |

Sigma rule for the T3 domains: [artex_infrastructure_dns.yml](../sigma/network-dns/artex_infrastructure_dns.yml).

## IP addresses (defanged, T3 historical unless stated)

| Indicator | Note |
|-----------|------|
| 38[.]244[.]50[.]120 | Reported ARTEX host; MMD treats any behavioural hit from it as T1 |
| 144[.]172[.]64[.]60, 144[.]172[.]64[.]99, 144[.]172[.]64[.]128 | Reported Storm-0940 related sign-in and spray sources |
| 178[.]16[.]54[.]129 | Reported Storm-3358 server |
| 207[.]5[.]195[.]93 | Reported in Eye Security reporting |
| 199[.]19[.]77[.]84 | Reported exposed-secret service principal sign-ins |
| 104[.]194[.]150[.]21, 167[.]88[.]172[.]97 | Reported by Wiz |
| 187[.]15[.]88[.]186, 187[.]15[.]89[.]186, 187[.]15[.]90[.]217 | T4: transient VPN exits, alert only |

No IP rule is shipped. Hunt these in your logs, and do not block on an IP match alone.

## Not included
Items MalwareMustDie marks HOLD or unverified (file hashes, user-agent strings, git author emails, JA3/JA4 fingerprints, file-name patterns) are not copied here.
