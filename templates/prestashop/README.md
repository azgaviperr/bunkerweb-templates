# PrestaShop Template

## Overview

Provision a secure baseline for PrestaShop behind BunkerWeb with:

- Reverse proxy defaults for a containerised PrestaShop backend
- Anti-bot cookie challenge (non-intrusive for shoppers)
- **AI/LLM crawler blocking** — prevents catalog scraping that causes `ps_connections` / `ps_guest` / `ps_page_viewed` database bloat
- ModSecurity CRS WAF
- Storefront-safe rate limiting (20 r/s — high enough for checkout flows)
- Auto bad-behavior banning on repeated 4xx/403

> **Why AI crawler blocking matters for PrestaShop specifically:**
> PrestaShop writes every visit to tracking tables (`ps_connections`, `ps_guest`, `ps_page_viewed`) rather than flat access logs. Multi-threaded AI catalog crawlers can generate gigabytes of database rows per hour, causing admin dashboards to freeze, cron optimisation scripts to time out, and search to degrade. Blocking at the proxy layer prevents these rows from ever being written.

## Prerequisites

- A running PrestaShop upstream service (for example `http://prestashop`).
- A domain name for your store.
- BunkerWeb UI access (or multisite settings access).

## Files

- `template.json` — BunkerWeb template definition and guided setup steps.

## Setup

1. Import the template in the BunkerWeb `Templates` UI using **Raw** mode.
2. Apply `USE_TEMPLATE=prestashop` to your service.
3. Customize at least `SERVER_NAME` and `REVERSE_PROXY_HOST`.
4. Reload BunkerWeb and validate checkout, customer login, and admin pages.

---

## AI Crawler Blocklist (PrestaShop-safe subset)

This template blocks **declared AI crawlers and recon scanners only**. Generic HTTP clients (`python-requests`, `go-http-client`, `libwww-perl`, `Java/`, etc.) are intentionally **not** blocked here because PrestaShop's ecosystem depends on them:

- Payment gateways (PayPal IPN, Stripe webhooks) use generic Java/HTTP clients
- Shipping carrier APIs (Colissimo, Chronopost, DHL) use various HTTP libraries
- Module update checks, marketplace integrations, and PrestaShop Addons use standard HTTP clients

For a site that does **not** need these integrations to pass through, combine this template with the `anti-claw-bot` template settings.

### Blocked AI crawlers

| Pattern | Operator |
|---------|----------|
| `GPTBot` | OpenAI (training) |
| `ChatGPT-User` | OpenAI (inference browsing) |
| `OAI-SearchBot` | OpenAI (search index) |
| `ClaudeBot` | Anthropic (training) |
| `Claude-Web` | Anthropic (inference) |
| `anthropic-ai` | Anthropic (generic) |
| `Google-Extended` | Google DeepMind (training opt-out) |
| `Bytespider` | ByteDance / TikTok AI |
| `CCBot` | Common Crawl (used in most LLM training sets) |
| `Diffbot` | Diffbot knowledge graph |
| `Applebot-Extended` | Apple Intelligence |
| `PetalBot` | Huawei AI products |
| `meta-externalagent` | Meta AI training |
| `meta-webindexer` | Meta AI indexing |
| `PerplexityBot` | Perplexity AI search |
| `cohere-ai` | Cohere |
| `YouBot` | You.com AI search |
| `AI2Bot` | Allen Institute for AI |
| `DataForSeoBot` | DataForSEO (data reseller) |
| `omgilibot` | Brandwatch / M-Brain |
| `Meltwater` | Meltwater media monitoring |
| `peer39_crawler` | Peer39 / Sizmek |
| `magpie-crawler` | Brandwatch |
| `AwarioSmartBot` | Awario brand monitoring |
| `CensysInspect` | Censys internet scanner |
| `RecordedFuture` | Recorded Future threat intel |
| `Infrawatch` | Infrastructure enumeration |
| `InternetMeasurement` | CAIDA / research scanning |
| `Keydrop` | Unknown recon scanner |

---

## Customization Tips

- Increase `MAX_CLIENT_SIZE` if you upload large product images or media files.
- Tune `LIMIT_REQ_RATE` down to `10r/s` for brochure-only sites; keep at `20r/s` if your checkout generates bursts.
- Switch `USE_ANTIBOT` from `cookie` to `captcha` for stronger challenge if bot traffic persists after blacklisting.
- Keep `USE_MODSECURITY_CRS=yes`; add exclusion rules only for confirmed false positives on specific PrestaShop modules.
- Monitor `ps_connections` and `ps_guest` row counts before and after deploying — a successful block is immediately visible as a flat or decreasing row count during previously high-traffic periods.

## Validation

```bash
jq . template.json
```
