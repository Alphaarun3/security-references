# Detections

Sigma rules and the pipelines used to convert them. Back to the [repo index](../README.md).

## Rules

New rows are added with each video. Video pages are in [videos/](../videos/).

| Rule | Purpose | Type | Video |
|------|---------|------|-------|
| [Burst cadence](sigma/web-waf/burst_cadence.yml) | One client sends far more requests in 10 s than a real user | correlation | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Path enumeration](sigma/web-waf/path_enumeration.yml) | Many distinct 403/404 paths from one client in a minute | correlation | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Data pull](sigma/web-waf/data_pull.yml) | One token runs many export/dump/backup actions in an hour | correlation | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Token from a new place](sigma/cicd-tokens/token_used_from_new_place.yml) | GitHub token used outside your CI runner ranges | base rule | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Token creates resources](sigma/cicd-tokens/token_creates_resources.yml) | One identity creates many functions, keys or roles quickly | correlation | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Servers calling LLM APIs](sigma/network-dns/servers_calling_llm_apis.yml) | DNS lookups for LLM API hosts from hosts with no need | base rule | [AI pentest defence](../videos/ai-pentest-defense.md) |
| [Permission probing](sigma/cloud-identity/permission_probing.yml) | Many distinct AWS DryRun permission checks | correlation | [AI pentest defence](../videos/ai-pentest-defense.md) |

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

Sigma rules are generic. They do not know your Splunk index names or your field names, so you tell the converter with a small **pipeline file**. An example is in [pipelines/splunk-example.yml](pipelines/splunk-example.yml). Copy it and edit three things:

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
