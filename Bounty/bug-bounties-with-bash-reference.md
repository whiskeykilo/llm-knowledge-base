---
title: "Bug Bounties With Bash (TomNomNom)"
source: "bug_bounties_with_bash_tomnomnom.md"
topics: ["bug bounty", "automation", "bash"]
---

## Core Principles

- Treat Bash as glue for chaining simple UNIX tools into custom reconnaissance workflows; e.g. `cat scope.txt | xargs -I{} curl -s https://{}` to fan out requests.
- Remember stdin, stdout, and stderr redirection (`cmd > out.log 2>&1`) to persist transcripts while keeping the console clean.
- Use command substitution `domain_count=$(wc -l live_hosts.txt)` or process substitution `diff <(sort a.txt) <(sort b.txt)` to compare evolving datasets.
- Prefer lightweight shell scripts for rapid iteration, then promote stable snippets into version-controlled tooling when complexity grows.

```bash
# Sample transcript-safe wrapper for any recon command
run() {
  echo "[+] $*" | tee -a recon.log
  "$@" 2>&1 | tee -a recon.log
}
run subfinder -d example.com
```

## Essential Utilities

- `grep`, `sed`, and `awk` extract or transform patterns larger tools miss (e.g. `grep -F "apiKey" responses/*.body`).
- `find`, `sort`, `uniq`, and `xargs` traverse assets, dedupe results, and distribute workloads: `find screenshots -name '*.png' | sort | uniq`.
- `tee` mirrors output to disk while streaming (`httpx -l hosts.txt | tee httpx.out`) so reruns start with known-good data.
- `dtach -A recon.dtach bash -lc './massdns.sh'` keeps long-running scans alive without tying up a terminal session.
- Combine utilities: `rg -i token logs | awk '{print $2}' | sort -u` surfaces unique secrets from past captures.

## Subdomain Workflow

- Blend third-party data sources (crt.sh, certspotter, hackertarget) with brute-force DNS enumeration to avoid blind spots.
- Use wordlists plus Bash loops to test resolution and log successes:
  `while read sub; do host "$sub.$root" >/dev/null && echo "$sub.$root"; done < wordlist`.
- Capture CNAME chains (`dig +short CNAME store.example.com`) to uncover dangling records and takeover opportunities.
- Parallelize request bursts with `xargs -P 20` or `gnu parallel` once preliminary scripts behave as expected.
- Persist subdomain intel per target: append findings into `recon/example.com/subdomains/live.txt` for downstream steps.

```bash
root=example.com
cat wordlist.txt |
  sed "s/$/.$root/" |
  xargs -P 25 -I{} sh -c 'dig +short {} >/dev/null && echo {}'
```

## Fetching And Filtering

- After enumeration, script bulk fetching (`httprobe < live.txt | xargs -L1 -I{} curl -s {} > "responses/{}.body"`) so analysis runs offline.
- Scan for titles, server headers, takeover fingerprints, secrets, or error traces to prioritize review (`rg -n "S3 Bucket" responses`).
- Extract JavaScript URLs (`grep -oE 'https?://[^\"'\'' ]+\.js' responses/*.body | sort -u`) and recursively gather linked assets.
- Flag base64-like patterns (`rg -o 'eyJ\w+' responses`) and immediately pipe into `base64 -d` to confirm whether credentials leak.
- Track HTTP status codes alongside body hashes to spot subtle content drift between releases.

## Tuning Pipelines

- Favor generic shell functions with parameters (`fetch_js() { local host=$1; ... }`) over brittle ad-hoc loops.
- Wrap reconnaissance loops into `.sh` scripts once prototypes stabilize; place them under `scripts/` with inline usage comments.
- Keep outputs timestamped (`date +%F` suffixes) so regressions stand out when diffing new runs against archives.
- Reach for Go tooling such as `meg`, `waybackurls`, `gf`, `httprobe`, and `concurl` when concurrency or parsing exceeds Bash comfort.
- Store environment metadata (command versions, wordlist sources) at the top of each script to help future operators reproduce results.
