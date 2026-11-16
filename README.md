# security-references

A public reference library from my security videos on product security and AI security: detections, notes and supporting material that go with each video. Everything here is defensive.

- YouTube (Tamil): [@Arunkumar_R](https://www.youtube.com/@Arunkumar_R)
- YouTube (English): [@Arunkumar_R_EN](https://www.youtube.com/@Arunkumar_R_EN)

## Index

| Folder | What is inside |
|--------|----------------|
| [detections/](detections/README.md) | Sigma detection rules, grouped by category, and example pipelines for converting them to Splunk and other SIEMs |
| [detections/iocs](detections/iocs/artex-historical-iocs.md) | Historical indicator lists with sources and caveats (hunting leads, not block lists) |
| [videos/](videos/) | One page per video: topic, sources and the content it uses |

### Videos

| Video page | Topic | Content |
|------------|-------|---------|
| [AI pentest defence](videos/ai-pentest-defense.md) | Defending against AI-driven penetration testing tools | 8 Sigma rules in [detections/sigma](detections/sigma/) and ARTEX indicators in [detections/iocs](detections/iocs/) |

### Detection categories

| Category | Folder |
|----------|--------|
| Web and WAF | [detections/sigma/web-waf](detections/sigma/web-waf/) |
| CI/CD and tokens | [detections/sigma/cicd-tokens](detections/sigma/cicd-tokens/) |
| Cloud and identity | [detections/sigma/cloud-identity](detections/sigma/cloud-identity/) |
| Network and DNS | [detections/sigma/network-dns](detections/sigma/network-dns/) |

## Using the content

Content is a starting point, not a drop-in solution. Thresholds and field names are examples, so baseline your own environment and run anything new in log-only mode before you alert or block. How to convert and tune the rules is in [detections/README.md](detections/README.md).

## License

MIT. See [LICENSE](LICENSE).
