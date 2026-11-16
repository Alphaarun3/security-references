# security-detections

Defensive detection content from my security videos (product security and AI security): Sigma rules you can convert to your own SIEM.

- YouTube (Tamil): [@Arunkumar_R](https://www.youtube.com/@Arunkumar_R)
- YouTube (English): [@Arunkumar_R_EN](https://www.youtube.com/@Arunkumar_R_EN)

> These are starting points, not drop-in rules. Field names differ per log source, and every threshold is an example.
> **Baseline your own traffic, then run each rule in log-only mode for a few days before you alert or block.**

## Rules

Video: [Defending against AI pentest tools](VIDEO_LINK_TODO) (long09). Details in [videos/long09-ai-pentest-defense.md](videos/long09-ai-pentest-defense.md).

| # | Rule | Purpose | Type |
|---|------|---------|------|
| 1 | [Burst cadence](rules/web-waf/burst_cadence.yml) | One client sends far more requests in 10 s than a real user | correlation |
| 2 | [Path enumeration](rules/web-waf/path_enumeration.yml) | Many distinct 403/404 paths from one client in a minute | correlation |
| 3 | [Data pull](rules/web-waf/data_pull.yml) | One token runs many export/dump/backup actions in an hour | correlation |
| 4 | [Token from a new place](rules/cicd-tokens/token_used_from_new_place.yml) | GitHub token used outside your CI runner ranges | base rule |
| 5 | [Token creates resources](rules/cicd-tokens/token_creates_resources.yml) | One identity creates many functions, keys or roles quickly | correlation |
| 6 | [Servers calling LLM APIs](rules/network-dns/servers_calling_llm_apis.yml) | DNS lookups for LLM API hosts from hosts with no need | base rule |
| 7 | [Permission probing](rules/cloud-identity/permission_probing.yml) | Many distinct AWS DryRun permission checks | correlation |

Correlation files hold a base rule plus a Sigma correlation rule (`event_count` / `value_count`), which need a recent pySigma backend.

## Convert to your SIEM

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install sigma-cli
sigma plugin install splunk elasticsearch azure
sigma check rules
```

```bash
# Splunk
sigma convert -t splunk --without-pipeline rules/web-waf/path_enumeration.yml
# Elastic (Lucene query)
sigma convert -t lucene --without-pipeline rules/network-dns/servers_calling_llm_apis.yml
# Microsoft Sentinel / Log Analytics (KQL)
sigma convert -t kusto --without-pipeline rules/cicd-tokens/token_used_from_new_place.yml
```

Use the processing pipeline for your log source in place of `--without-pipeline` where one exists, then map field names. Not every backend supports every correlation type.

## Scope

Defensive content only. No exploits or attack payloads. Rules are `experimental`: expect false positives and tune them.

## License

See [LICENSE](LICENSE).
