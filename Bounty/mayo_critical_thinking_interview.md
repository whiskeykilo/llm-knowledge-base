---
title: "John Colston (Mayo) — Data-Driven Bug Bounty Playbook"
source: mayo_interview_transcript.md
created: 2024-02-05
tags: [bug_bounty, automation, data_science, digital_marketing, recon_workflows]
aliases: ["Mayo", "My Own Eyes", "Jon Colston"]
---

## Quick Facts

- `Identity`: John Colston, hacker handle “Mayo” / “My Own Eyes”, ex-digital marketing executive.
- `Focus`: Data-informed recon for ad-tech targets, especially Yahoo; blends automation with selective manual review.
- `Guiding quote`: “What is not measured is not managed.” — Peter Drucker (referenced ~23:31).
- `Current goal`: Treat bug hunting as “practice retirement”; automation surfaces leads so manual time stays limited.
- `Notable metric`: Targets new assets to surface in his review queue within 18 hours of discovery (~01:14:26).

## Background Snapshot

- Georgia Tech graduate (1999) with early-career work on real-time credit decision systems and lead-gen analytics (~03:16–04:14).
- Spent years building national digital brands, owning analytics for multi-channel ad spend.
- Entered bug bounty about five years prior to interview; heavy early focus on Yahoo’s ad-tech stack.
- Brand origin: “Mayo” is a phonetic play on Smashing Pumpkins’ song “Mayonaise” → “My Own Eyes” (~00:50–01:27).

## Core Philosophy & Mindset Hooks

- **Measure everything**: Wraps every process in instrumentation so inputs, outputs, timing, and hit-rates are captured (~24:00–27:31).
- **Lead generation for vulns**: Automation should hand him scored “opportunities,” letting him spend mornings on reports and afternoons on golf (~35:12–35:44).
- **Play to skill set**: Prefers asset discovery and data patterns over deep exploit chains; turns domain nuance into repeatable advantages (~21:19–21:43).
- **Automate to create headspace**: Automation is “creating” while exploitation is “destroying”; the system keeps earning even when he steps away (~01:09:03–01:10:45).

## Automation Architecture Overview

- **Process stack**: ~30 bash-driven subprocesses chained into a conversion funnel; each stage logs run time, records-in/out, and hits to flat-file KPI logs (~34:12–37:41, ~56:21–57:30).
- **Execution target**: Keep per-host processing under 4–5 minutes to clear ~3,000 assets per server per day (~56:12–56:43).
- **Wrapper strategy**: Every tool invocation (fuzzers, crawlers, link finders) is called via scripts that tag program, root, subdomain, wordlist identity, and result (~24:27–27:01).
- **Output format**: Final daily log highlights discovered endpoints with HTTP method, discovery stage, status, and prioritization flags (403 on crawl > 403 via fuzz) (~56:41–57:30).
- **Tech bias**: Heavy bash usage; historically even scripted custom NSE scans in Nmap before Ffuf/Nuclei existed (~01:34:55–01:35:53).

## Data & Wordlist Engineering

- **Ingredient / Recipe model** (~41:29–43:44):
  - Split domains by delimiters (dot, dash) without stemming to capture acronyms and product codenames.
  - Manually group high-frequency tokens into “ingredients” (e.g., API terms, environment markers, geography).
  - Compose “recipes” by slotting ingredients into observed domain patterns; suffix with placeholders when gaps remain.
  - Apply recipes to generate new subdomain permutations and reuse for tailored content discovery.
- **Dynamic enrichment**:
  - Feed words from domain assets, LinkFinder, crawlers, and historic hits back into wordlists in structured combos (~37:10–38:06).
  - Maintain background jobs that test fresh keyword batches (e.g., 1k at a time across 10k domains) to score hit rates before adding to production (~48:18–50:53).
- **List specialization**: Distinct wordlists per intent—API discovery, uploads, directory traversal, etc.—tracked for comparative performance (~27:31–28:56).
- **Anomaly hunting**: Surface recipes that explode in size (e.g., 5M keywords) to debug logic or chase unexpected asset families (~37:41–38:20).

## Data Sources & Scope Governance

- **Sources referenced**: SecurityTrails (primary), Shodan, ZoomEye, BinaryEdge, Assetnote, BigQuery, program-supplied dumps (~45:07–45:30, ~01:05:40–01:07:33, ~01:00:46–01:01:11).
- **Scope file**: Flat manifest enumerating program, asset type (domain/CIDR/etc.), scope status, bug platform, and which automation stages to run. Each data feed earns a frequency weight (1 = daily, 2 = staged, etc.) to control spend (~01:06:18–01:07:33).
- **Cost discipline**: Combined infra + data spend ≈ $1.5k/month; expects a single substantive bug to cover costs (~01:07:19–01:08:24).
- **Temporal heuristics**: Fewer new assets around weekends/holidays; aligns scheduling and expectations accordingly (~01:14:50–01:15:24).

## Credential & Access Strategies (B2B Focus)

- **Demo accounts**: Locate shared demo/test users (e.g., `demo@company`) with predictable passwords for brute-force or password reset attempts (~15:10–15:57).
- **Expired domains**: Track partner/customer domains that lapse, acquire them, and trigger password resets from owned inboxes (~16:26–17:23).
- **Legitimate onboarding**: When possible, pursue official onboarding flows to secure durable access that programs cannot rescind (~18:00–18:01).
- **Two-factor caveat**: Growth of MFA makes brute-force wins rare; requests temporary waivers only after notifying programs (~17:06–18:01).

## Manual Analysis Loop

- **Question-driven digs**: Uses logs to pose questions (e.g., swagger needing query variables) then designs targeted experiments to test across corpus (~59:46–01:01:19).
- **Historical replay**: Because every discovery is archived, he can retroactively inspect where else a pattern might exist (~57:19–58:19).
- **Experiment compartmentalization**: Runs new hypothesis scripts outside production until they prove value, then instruments them like any other stage (~01:01:53–01:02:53).
- **Prioritization signals**: Favors crawl-originated 403s, rare HTTP methods, or unseen endpoint shapes as prompts for manual exploitation (~56:41–57:19).

## Conversion Funnel Metrics

- **Stages translated from marketing**:
  1. **Impressions** → total assets gathered (domains, IPs, docs).
  2. **Clicks** → resolved assets and viable hosts.
  3. **Engagement** → endpoints/content returned by fuzz + crawl.
  4. **Lead** → surfaced opportunity log items.
  5. **Sale** → validated vulnerability.
- **Feedback loop**: Every conversion drop-off is investigated (e.g., data source latency, wordlist mismatch) to tighten the funnel (~01:18:29–01:19:47).
- **Timing goal**: Asset discovery to analyst review within 18 hours on weekdays (~01:14:10–01:14:50).

## Distribution & “Moab” Strategy

- **Moab definition**: Systemic bug type that stays undiscovered, replicates across hosts/features, resists blanket fixes, and carries high impact (~01:31:00–01:31:39).
- **Examples**:
  - Legacy ad platform leaking auction KPIs everywhere; single fix produced wide-ranging payouts (~01:31:39–01:32:10).
  - YQL server API doc revealing endpoints that cloned into dozens of medium findings (~01:32:31–01:34:44).
- **Duplication reality**: Accepts program variance; some teams pay per-instance, others merge to one fix. Requests report IDs to gauge timing gaps (~01:37:22–01:43:08).

## Digital Marketing Parallels

- **Blind auctions**: Advertising bids are sealed; leaking competitor spend, targeting, or creative strategy is high-impact (`bread and butter` argument) (~01:25:31–01:30:21).
- **Competitive intelligence**: Marketers pay for tools (e.g., SpyFu) to infer rivals’ spend; direct access via bugs is uniquely valuable (~01:29:17–01:30:00).
- **Messaging embargo**: Early access to campaign creatives undermines launch strategies, qualifying as sensitive exposure even if ads later go public (~01:28:26–01:29:04).
- **Security insight**: Old ad-tech often lacks secure SDLC; deep archival systems can expose pre-security-era APIs (~13:51–20:58).

## Personal Ops & Burnout Guardrails

- **Lifestyle design**: “Practice retirement” approach—limit work hours, keep routine enjoyable (~35:12–35:44, ~01:09:03–01:10:45).
- **Expectation management**: Treat outcomes as business, not personal; avoid chasing adrenaline highs which previously led to burnout (~01:39:32–01:41:09).
- **Community practice**: Requests transparent timelines from programs when encountering stale duplicates to maintain trust (~01:42:03–01:42:50).
- **Emotional hygiene**: Acknowledge “second place is first loser” sting, but keeps reviewing dupes to learn speed and coverage gaps (~01:41:23–01:43:08).

## Tooling & Infrastructure Notes

- Bash scripts orchestrating fuzzers, crawlers, LinkFinder, Nuclei (implicitly), and custom NSE modules.
- Uses flat files (CSV/log) as the “database” for portability and simplicity (~24:49–27:31, ~33:37–34:34).
- Relies on scheduled servers plus reminder system to ingest new keyword batches or data feeds (~48:18–50:53).
- Monetization target: one bug to cover monthly infra spend; remainder is profit with reduced time pressure (~01:07:19–01:08:24).

## Retrieval Hooks (LLM Prompts)

- “How does Mayo build and score bug bounty wordlists?”
- “What conversion funnel metrics can guide recon automation?”
- “Strategies Mayo used to gain B2B credentials?”
- “Why is leaking ad auction KPIs critical?”
- “How does Mayo mitigate burnout while automating recon?”

## Suggested Q&A Pairs

- **Q**: What turnaround time does Mayo target between discovering an asset and reviewing it?  
  **A**: Approximately 18 hours on weekdays, longer on weekends/holidays due to fewer releases (~01:14:10–01:15:18).
- **Q**: How does Mayo decide which 403 responses deserve manual investigation?  
  **A**: He weights results by discovery stage—403 from crawl is prioritized over fuzz because it signals higher relevance (~56:41–57:30).
- **Q**: Describe Mayo’s “ingredient and recipe” approach.  
  **A**: Tokenize domain components into grouped “ingredients,” then assemble them into repeatable “recipes” for generating subdomains and content discovery paths (~41:29–43:22).
- **Q**: Which data sources does Mayo routinely integrate?  
  **A**: SecurityTrails (primary), plus Shodan, ZoomEye, BinaryEdge, Assetnote, and ad-hoc public datasets (~01:05:40–01:07:33).
- **Q**: What monthly cost does Mayo expect to maintain his automation stack?  
  **A**: Roughly $1,500 across servers and data subscriptions (~01:07:19–01:08:24).
