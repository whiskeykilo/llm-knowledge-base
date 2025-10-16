---
title: "Zseano’s Bug Bounty Methodology"
source: "Zseano's Bug Bounty Methodology.md"
topics: ["bug bounty", "web hacking", "mindset"]
---

## Philosophy

- Treat hacking as structured curiosity: question how every feature works, which assumptions it makes, and how it fails under edge cases.
- Build relationships with application security teams; consistent collaboration (status updates, repro videos) strengthens payouts and trust.
- Balance self-reliance with community support, sharing payloads or bypass techniques when possible to elevate newcomers.
- Remember the ethics: target only authorized programs, practice responsible disclosure, avoid extortion, and document intent clearly in reports.

## Getting Started

- Pair this methodology with foundational resources like Andy Gill’s “Breaking into Information Security” to cover tooling basics.
- Develop enough HTML, JavaScript, and CSS literacy to craft PoCs (e.g., DOM clobbering, fetch-based exfiltration) and understand developer intent.
- Expect iterative learning: Zseano logged 600+ bugs over years by combining coding background with disciplined recon notes.
- Track how developers patch issues to anticipate regressions; tag reports with categories (authz, SSRF, stored XSS) for trend analysis.

## Program Approach

- Select programs where you can invest time, understand feature roadmaps, and observe release cadence (weekly pushes imply frequent regressions).
- Maintain detailed notes, screenshots, and PoCs per asset; include auth state, payloads used, and server responses for reproducibility.
- Use BugBountyHunter training apps (Barker, Kreative, FirstBlood) to simulate real-world program workflows before touching production targets.
- Treat each recon session as puzzle-solving; map features end to end (create, edit, delete flows) before diving into exploit classes.
- Keep a “questions log” for each feature: “What if user uploads SVG?”, “Can I reuse invite links?”, “Does rate limiting apply to API key creation?”.

```text
notes/program/
  features/
    invoices.md      # endpoints, roles, risky parameters
    messaging.md
  payloads/
    xss.md
    ssrf.md
  findings/
    2023-10-15-xss-team-chat.md
```

## XSS Testing Flow

- Start with benign HTML tags (`<h2>hello</h2>`, `<img src=x onerror=alert(1)>`) to gauge reflection and encoding behavior.
- Escalate to double-encoded payloads (`%253Cscript%253E`, `%26lt;img%20src=x%26gt;`) to probe how filters normalize inputs.
- Reverse engineer filters by enumerating blacklisted tags/attributes (`onerror`, `javascript:`) and case variations (`<ScRiPt>`).
- Experiment with partial tags (`<script src=//x>`, `<%00iframe srcdoc=...>`) and alternative contexts (JSON, Markdown, HTML attributes).
- Capture bypass attempts with context: endpoint, parameter, payload, response snippet, verdict (blocked/reflects/raw HTML).

## Knowledge Base Notes

- Capture repeatable checklists (encoding tests, blacklist probes, context mapping) for LLM-guided audits to walk hunters through XSS triage.
- Emphasize mindset coaching—natural curiosity, patience, questioning—as much as technical payloads when designing helper prompts.
- Highlight success metrics (relationships, recognition, sustained effort) to motivate long-term practice and discourage quick wins-only thinking.
- Document responsible-use reminders prominently so automated agents reinforce legal boundaries, especially when suggesting exploit paths.
- Store sample responsible disclosure templates referencing program rules and safe proof-of-concept guidelines.
