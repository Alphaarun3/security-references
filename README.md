# security-detections

Defensive detection content from my security videos on product security and AI security. Each video adds new detections here, as Sigma rules you can convert to your own SIEM.

## Layout

```
detections/
  sigma/<category>/   Sigma rules (web-waf, cloud-identity, cicd-tokens, network-dns, ...)
  pipelines/          Example Sigma pipelines to map index and field names per SIEM
videos/               One page per video: which rules it covers and the sources
```

- YouTube (Tamil): [@Arunkumar_R](https://www.youtube.com/@Arunkumar_R)
- YouTube (English): [@Arunkumar_R_EN](https://www.youtube.com/@Arunkumar_R_EN)

> These are starting points, not drop-in rules. Field names differ per log source, and every threshold is an example.
> **Baseline your own traffic, then run each rule in log-only mode for a few days before you alert or block.**

## Rules

New rows are added with each video. Video pages are in [videos/](videos/).

| Rule | Purpose | Type | Video |
|------|---------|------|-------|
| [Burst cadence](detections/sigma/web-waf/burst_cadence.yml) | One client sends far more requests in 10 s than a real user | correlation | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Path enumeration](detections/sigma/web-waf/path_enumeration.yml) | Many distinct 403/404 paths from one client in a minute | correlation | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Data pull](detections/sigma/web-waf/data_pull.yml) | One token runs many export/dump/backup actions in an hour | correlation | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Token from a new place](detections/sigma/cicd-tokens/token_used_from_new_place.yml) | GitHub token used outside your CI runner ranges | base rule | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Token creates resources](detections/sigma/cicd-tokens/token_creates_resources.yml) | One identity creates many functions, keys or roles quickly | correlation | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Servers calling LLM APIs](detections/sigma/network-dns/servers_calling_llm_apis.yml) | DNS lookups for LLM API hosts from hosts with no need | base rule | [AI pentest defence](videos/ai-pentest-defense.md) |
| [Permission probing](detections/sigma/cloud-identity/permission_probing.yml) | Many distinct AWS DryRun permission checks | correlation | [AI pentest defence](videos/ai-pentest-defense.md) |

Correlation files hold a base rule plus a Sigma correlation rule (`event_count` / `value_count`), which need a recent pySigma backend.

## Convert to your SIEM

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install sigma-cli
sigma plugin install splunk elasticsearch azure
sigma check detections/sigma
```

```bash
# Splunk
sigma convert -t splunk --without-pipeline detections/sigma/web-waf/path_enumeration.yml
# Elastic (Lucene query)
sigma convert -t lucene --without-pipeline detections/sigma/network-dns/servers_calling_llm_apis.yml
# Microsoft Sentinel / Log Analytics (KQL)
sigma convert -t kusto --without-pipeline detections/sigma/cicd-tokens/token_used_from_new_place.yml
```

Not every backend supports every correlation type.

## Make it fit your environment (Splunk example)

Sigma rules are generic. They do not know your Splunk index names or your field names, so you tell the converter with a small **pipeline file**. An example is in [detections/pipelines/splunk-example.yml](detections/pipelines/splunk-example.yml). Copy it and edit three things:

1. **Index and sourcetype**: the `index:` and `sourcetype:` values (for example `web`, `dns`, `aws_cloudtrail`) to the ones in your Splunk.
2. **Field names**: the right side of the `mapping:` (for example `src_ip`, `uri_path`, `status`) to the field names your data really uses.
3. **Allow-lists**: for rules 4 and 6, put your own CI runner ranges and approved AI hosts in the rule's filter.

Then convert with your pipeline:

```bash
sigma convert -t splunk -p detections/pipelines/my-splunk.yml detections/sigma/web-waf/path_enumeration.yml
```

Example output with the pipeline applied:

```
index="web" sourcetype="access_combined" (status IN (403, 404))
| bin _time span=1m
| stats dc(uri_path) as value_count by _time src_ip
| search value_count >= 40
```

To find your real field names, run a search on your own data and look at the fields (`index=web | head 5`). Run the converted query as a saved search in log-only mode before you alert. Sentinel and Elastic work the same way with their own pipelines.

## Scope

Defensive content only. No exploits or attack payloads. Rules are `experimental`: expect false positives and tune them.

## License

MIT. See [LICENSE](LICENSE).
