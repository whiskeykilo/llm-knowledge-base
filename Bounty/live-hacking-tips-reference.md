---
title: "Live Hacking Playbook (Frans Rosén)"
source: "Live Hacking Tips and Tricks - Frans Rosen.md"
topics: ["bug bounty", "live hacking events"]
---

## Overview

- Live hacking events pair invited hackers with onsite engineers for rapid triage and payouts; expect real-time fix verification.
- Companies typically expand scope to include wildcard domains, infrastructure, enterprise tenants, and occasionally acquired brands.
- Direct Slack or in-person communication accelerates clarifications (e.g. “Is `admin.corp.dev` production traffic?”) and unlocks attack chains.
- Pre-submissions, when allowed, reduce day-of pressure and lead to faster bounty processing; teams often ship 5–10 reports before arrival.

## Event Flow

- Receive company briefing with walkthrough slides, demo accounts, and live Q&A to clarify scope and expectations; record answers verbatim.
- Map the widened scope, confirm credentials or enterprise upgrades, and log out-of-scope areas early (e.g. “support portal is read-only, no auth fuzzing”).
- Dedicate the allotted recon window (often 24–72 hours) to asset discovery—subdomain sweeps, integration audits, and credential inventory.
- Leverage collaboration channels onsite to validate hypotheses and avoid duplicate reporting; tag program staff when reproducing tricky steps.
- Automate polished report drafting up front (e.g., `bountyplz -t jwt-tenant-escape.md`) to hit “first valid bug” or “best chain” incentives quickly.
- Participate in the post-hunt show-and-tell to learn peers’ approaches, capture remediation notes, and refine future tactics.

## Team Strategy

- Form small teams (two to four people) with complementary skills (e.g., API testing + mobile reversing) and comparable effort levels.
- Share note repositories (`obsidian`, `hedgedoc`, or plain text in git) so “weird behavior” observations feed collective attack chains.
- Divide surface area by feature, stack, or acquisition lineage instead of by vulnerability class—one person on legacy SOAP APIs, another on OAuth apps.
- Agree on payout splits in advance; default to equal halves for pairs unless large time deltas require weighted splits.
- Schedule sync checkpoints (morning recon standup, afternoon exploitation focus) to keep coverage balanced.

## Recon Priorities

- Exhaust “boring” or labor-intensive areas other hunters skip, such as legacy integrations, PDF generators, or customer support tooling.
- Inspect SDKs, desktop clients, and published code to learn undocumented API behavior; diff `sdk-v2.4` vs production endpoints for mismatched auth.
- Enumerate third-party integrations (Slack, Zapier, Trello) and vet OAuth/webhook flows end to end, including token rotation and scope enforcement.
- Diff company-managed forks or open source projects for security patches missing in production; track `git diff upstream/master` for stale fixes.
- Mine GitHub, gists, and issues for internal indicators, credentials, and cloud asset references; prioritize `org:company filename:.env`.
- Pull archived documentation and URLs from Wayback Machine or Common Crawl to resurrect legacy entry points like `/beta/admin/export`.

## Tooling And Notes

- Maintain per-target directories with plain-text notes, Burp histories, PoC snippets, and screenshots; sync via git for version history.
- Use DNS brute forcing, JS route extraction, and wfuzz lists tailored from newly observed keywords (e.g., `customersuccess` => add to wordlist).
- Stand up an internal-only SSRF test host that serves varied file types without requiring Host headers; include `/metadata`, `/smb`, `/aws-metadata` paths.
- Track potential report chains explicitly (e.g., open redirect + OAuth flow leading to privilege escalation) to assemble multi-bug narratives.
- Prepare quick-reference checklists for day-of execution: login endpoints to retest, GraphQL introspection queries, reusable Burp macros.

```text
notes/
  00_scope.md        # credentials, in/out-of-scope, contact handles
  10_recon.md        # domains, IP ranges, integrations
  20_findings.md     # report drafts with reproduction steps and impact
  artifacts/
    burp-logs/
    screenshots/
```

## Case Study Signals

- Monitor proxy error pages for leaked tokens; one event exposed a tenant-wide admin JWT in Squid error logs, enabling cross-tenant access.
- Probe IPv6-only paths and verbose headers when IPv4 behavior seems sanitized; header bloat can reveal backend IPs or debug credentials.
- Combine Burp Collaborator payload spraying with manual review to confirm blind interaction bugs; log correlation timestamps in shared docs.
- Treat sandbox escapes seriously: XSS inside unprivileged domains can exfiltrate documents via `postMessage` into privileged iframes.
- Document second-order findings (e.g., initial JWT leak followed by RCE four hours later) to show compounding impact.

## Knowledge Base Tips

- Capture recon tactics, collaboration norms, and tooling as reusable recipes (e.g., “How to triage enterprise SaaS targets at live events”).
- Emphasize questions to ask programs: “Which domains are DNS-only?”, “Do mobile builds point at separate APIs?”, “Any legal hold around customer data?”.
- Highlight social dynamics—communication, show-and-tell, pairing—as core differentiators that an LLM assistant should remind hunters about.
- Preserve concrete war stories to ground methodology in memorable patterns, linking each to a bounty value and root cause category.
