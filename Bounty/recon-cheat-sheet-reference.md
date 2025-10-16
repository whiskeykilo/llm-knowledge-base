---
title: "Recon Cheat Sheet 2019 Highlights"
source: "Recon Cheat Sheet 2019 - HackerOne.md"
topics: ["bug bounty", "reconnaissance"]
---

## Digital Dumpster Diving

- Pivot on umbrella companies and acquisitions to surface forgotten assets or credentials (e.g., query `"Acme Holdings" ftp password`).
- Search paste sites, code snippets, and public notes for leaked FTP details, API keys, or passwords; log context to support responsible disclosure.
- Track hidden endpoints and cloud instances, then validate findings responsibly for high payouts—$10k+ FTP compromises are documented outcomes.
- Combine manual review with gitrob, git-all-secrets, truffleHog, git-secrets, and repo-supervisor; store suspicious results with commit SHAs for validation.
- Example workflow:
  `gh api repos/{org}/{repo}/contents | jq -r '.[].download_url' | grep -E '.env|.json' | xargs -n1 curl -s`.

## Content Discovery

- Run directory brute-force tools (`dirbuster`, `dirsearch`, `gobuster`, `gograbber`) across every exposed port; log 200/302 responses to CSV.
- Always archive screenshots and raw findings; `webscreenshot.py -i hostlist.txt -o screenshots/$(date +%F)` enables quick before/after comparisons.
- Let robots.txt hints, 403 responses, and nonstandard ports guide deeper probing of admin paths; queue follow-up fuzzing for `/admin/*`.
- Normalize nmap runs over extended port lists (e.g., 3868, 8443, 8080, 15672) with `nmap -Pn -sV -p $(cat ports.txt | tr '\n' ',') target`.
- Pivot on service banners (Jenkins, Nexus, RabbitMQ) to craft tailored exploit checklists or password reuse attempts.

## Asset Discovery

- Brute-force subdomains with sublist3r, massdns, altdns, brutesubs, knockpy, dns-parallel-prober, and dnscan; merge outputs via `sort -u`.
- Explore environment naming conventions (`dev`, `stage`, `uat`, hyphen/period variants) to extend scope; cross-check against TLS SAN entries.
- Treat WHOIS and RIR data as a seed list for netblocks tied to acquisitions or regional subsidiaries; `whois example.com | rg -i "netrange"`.
- Document every permutation tested so automation can revisit high-yield branches; store wordlists and execution commands per target directory.
- Snapshot DNS records (`dnsx -l domains.txt -o resolved.txt -json`) so coverage gaps are obvious on later reruns.

## Certificate Transparency

- Automate certspotter and crt.sh queries to pull SAN entries that DNS brute force misses (`curl https://certspotter.com/api/v1/issuances?domain=example.com`).
- Pair Censys and Shodan searches with filters on ports, products, and titles, e.g. `services.service_name:jenkins AND parsed.names:example.com`.
- Check for SSL certificates referencing corporate brands across vendor infrastructure to uncover unmanaged third-party deployments.
- Script aliases (`certspotter`, `dirbruteforce`, `screenshot`) to chain discovery, fuzzing, and visual triage; store scripts alongside usage docs.
- Correlate certificate issue dates with acquisition timelines to identify subsystems that may not share central security controls.

## Knowledge Base Notes

- Encourage LLM prompts that ask for acquisition assets, environment variants, and leaked metadata to mimic expert triage questions.
- Reinforce the habit of coupling manual review with automation; advise rotating tools when results plateau.
- Highlight trophy stories (e.g., $10k FTP leak after pastebin credentials, $5k Jenkins takeover via `hostname:target.com title:"Dashboard [Jenkins]"`) to illustrate payoff patterns.
- Keep reusable port lists, wordlists, and helper scripts versioned for rapid recon onboarding; note last refresh date for each resource.
- Capture sample disclosure narratives so future agents understand how to translate recon findings into impactful reports.
