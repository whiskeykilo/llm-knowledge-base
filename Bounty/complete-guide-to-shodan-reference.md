---
title: "Complete Guide to Shodan – Key Takeaways"
source: "complete_guide_to_shodan.md"
topics: ["internet scanning", "shodan"]
---

## Overview

- Shodan indexes Internet-exposed services by collecting textual banners instead of website content; think of it as port+protocol telemetry.
- Every service on an IP is treated as a separate record, so multi-service hosts yield multiple results visible via `shodan host <ip>`.
- Meta-data such as geolocation, hostnames, and operating systems accompany banners for filtering (fields documented in Appendix A).
- IPv6 crawling produces millions of banners monthly, supplementing the larger IPv4 corpus, though coverage remains thinner.

## Banner Anatomy

- Web banners expose HTTP status, server version, and response headers for fingerprinting; e.g. `Server: nginx/1.1.19`.
- Industrial protocols (e.g., Siemens S7) return firmware, module IDs, and serial numbers worth cataloging for asset owners.
- Appendices outline property names for HTTP, SSL, SMB, SSH, Elastic, and more; mirror those keys in parsing scripts.
- Hosts typically reveal enough detail in banners to map software stacks without active exploitation—ideal for exposure inventories.

```json
{
  "ip_str": "198.51.100.10",
  "port": 443,
  "http": {
    "status": 200,
    "title": "Admin Portal",
    "server": "nginx/1.14.0"
  },
  "ssl": {
    "versions": ["TLSv1.2", "-SSLv3"]
  }
}
```

## Crawling Model

- Distributed scanners operate continuously from the USA, China, Iceland, France, Taiwan, Vietnam, Romania, and Czech Republic.
- Random IP and port selection ensures uniform Internet coverage and mitigates regional blocking bias; results include timestamps per capture.
- Port testing draws from a curated list of supported services (see Appendix D) to balance breadth with banner richness.
- Real-time ingestion means new scans refresh search results instantly; `shodan stream` and the real-time API expose live data feeds.

## SSL Deep Dive

- Shodan stores full certificate chains plus negotiated protocol versions in the `ssl` structure for each HTTPS/TLS service.
- Explicit probes test SSLv2, SSLv3, TLSv1.0, TLSv1.1, TLSv1.2 (and newer) marking unsupported versions with `-` (e.g., `"-SSLv2"`).
- Collected Diffie-Hellman parameters surface weak primes and export-grade cipher support; field `ssl.dhparams.bits` flags 1024-bit keys.
- Heartbleed, FREAK, and Logjam checks populate `opts.vulns`; a leading `!` indicates the crawler confirmed the host is *not* vulnerable.

## Leveraging Results

- Filter by vulnerability IDs (`vuln:CVE-2014-0160`), product names, or protocol versions (`ssl.version:sslv2`) to triage exposures rapidly.
- Combine filters: `org:"Example Corp" product:"Apache httpd" country:US` narrows high-value assets; facets (`--facets country,port`) summarize.
- Use Shodan Maps, Exploits, and Images (per table of contents) to visualize findings and link them to public exploit references.
- Export data via `shodan download results.json.gz <query>` for offline enrichment, or integrate with Maltego, CLI scripts, or dashboards.
- Example API usage:

```bash
shodan search --fields ip_str,port,hostnames "ssl.cert.subject.cn:example.com"
shodan host 198.51.100.10 | jq '.data[] | {port,product:.product,updated:.timestamp}'
```

## Knowledge Base Notes

- Frame Shodan as telemetry first; emphasize responsible disclosure when sensitive systems appear in search results.
- Encourage building playbooks that join Shodan outputs with asset inventories and patch workflows; store sample join logic (`asset_id` ↔ `ip_str`).
- Document banner parsing patterns so an LLM can transform raw JSON into analyst-ready summaries, including key-value translations.
- Track recurring weak SSL configurations to monitor remediation progress; compare `ssl.versions` and `opts.vulns` snapshots across weekly exports.
