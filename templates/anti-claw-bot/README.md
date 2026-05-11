# Anti-claw / Anti-bot Template

## Overview

Apply a strict BunkerWeb hardening profile focused on AI/LLM crawlers, scrapers, and abusive bot traffic. The template enables CAPTCHA challenge mode, DNSBL checks, aggressive request throttling, and automatic bad-behavior bans.

This template is the companion to the PrestaShop template. Use it standalone for blogs, media sites, and content platforms where full anti-bot hardening is the priority, with no e-commerce-specific tuning.

## Prerequisites

- A domain name for the protected service.
- BunkerWeb deployed with templates support.
- Awareness that stricter limits (`5r/s`) may affect legitimate burst traffic.

## Files

- `template.json` — Hardened anti-bot profile and guided configuration steps.

## Setup

1. Import `template.json` from the BunkerWeb `Templates` page in **Raw** mode.
2. Apply `USE_TEMPLATE=anti-claw-bot` to the target service.
3. Set `SERVER_NAME` for your domain.
4. Reload and monitor BunkerWeb logs for false positives.

---

## Consolidated User-Agent Blocklist Reference (2026)

The `BLACKLIST_USER_AGENT` value in this template is a space-separated list of substring/regex patterns matched against the HTTP `User-Agent` header. Each entry below is documented with its source and risk category.

### Declared AI / LLM Training Crawlers

These bots have published official documentation or `robots.txt` entries. They should be blocked on any site that has not explicitly consented to data extraction.

| Pattern | Operator | Purpose | Source |
|---------|----------|---------|--------|
| `GPTBot` | OpenAI | Training data collection | platform.openai.com/docs/gptbot |
| `ChatGPT-User` | OpenAI | Real-time inference browsing | platform.openai.com |
| `OAI-SearchBot` | OpenAI | ChatGPT Search index | openai.com |
| `ClaudeBot` | Anthropic | Training data / retrieval | anthropic.com |
| `Claude-Web` | Anthropic | Inference-time web access | anthropic.com |
| `anthropic-ai` | Anthropic | Generic Anthropic agent token | anthropic.com |
| `Google-Extended` | Google DeepMind | AI training opt-out signal | developers.google.com/search |
| `Bytespider` | ByteDance | TikTok/Douyin AI products | bytedance.com |
| `CCBot` | Common Crawl | Open corpus (used in most LLM training runs) | commoncrawl.org |
| `Diffbot` | Diffbot | Knowledge graph / structured extraction | diffbot.com |
| `Applebot-Extended` | Apple | Apple Intelligence training | support.apple.com |
| `PetalBot` | Huawei | Search index / AI products | aspiegel.com |
| `meta-externalagent` | Meta | Meta AI training | meta.com |
| `meta-webindexer` | Meta | Meta AI indexing | meta.com |
| `PerplexityBot` | Perplexity AI | Search / answer engine | docs.perplexity.ai |
| `cohere-ai` | Cohere | AI model training | cohere.com |
| `YouBot` | You.com | AI search engine | you.com |
| `AI2Bot` | Allen Institute for AI | Research corpus | allenai.org |

### Undeclared / Media Intelligence Crawlers

These bots scrape content for media monitoring, data brokerage, or AI pipelines. Most do not honour `robots.txt` consistently.

| Pattern | Operator | Notes |
|---------|----------|-------|
| `DataForSeoBot` | DataForSEO | Data reseller, aggressive crawling |
| `omgilibot` | Brandwatch / M-Brain | Social/media monitoring, AI pipeline |
| `Meltwater` | Meltwater | Media monitoring service |
| `peer39_crawler` | Peer39 / Sizmek | Advertising intelligence |
| `magpie-crawler` | Brandwatch | Content aggregation |
| `AwarioSmartBot` | Awario | Social listening / brand monitoring |

### Security Scanners & Reconnaissance Bots

These are not AI crawlers but perform mass infrastructure scanning. Blocking them prevents reconnaissance data about your stack from being indexed by threat-intelligence platforms.

| Pattern | Operator | Notes |
|---------|----------|-------|
| `CensysInspect` | Censys | Continuous internet-wide port/service scanning |
| `RecordedFuture` | Recorded Future | Threat intelligence feed crawler |
| `Infrawatch` | Unknown | Infrastructure enumeration |
| `InternetMeasurement` | CAIDA / research orgs | Academic/commercial scanning |
| `Keydrop` | Unknown | Key/credential discovery scanning |

### Generic HTTP Clients (Bare Scrapers)

These are not declared bots — they are raw HTTP client library defaults used by scrapers, automation scripts, and exploit tools. Legitimate applications set a meaningful `User-Agent`; blocking these patterns has very low false-positive risk on public-facing websites.

| Pattern | Examples |
|---------|---------|
| `python-requests` | Python `requests` library (default UA) |
| `python-urllib` | Python `urllib` / `urllib2` |
| `go-http-client` | Go `net/http` default UA |
| `libwww-perl` | Perl LWP library |
| `okhttp` | Android/Java OkHttp default |
| `Java/` | Generic Java `HttpURLConnection` |

### PHP Injection Strings

These match injection probes commonly embedded in `User-Agent` headers by vulnerability scanners and exploit attempts.

| Pattern | Meaning |
|---------|---------|
| `x22` | Hex-encoded double quote — SQL/code injection probe |
| `{|}` | PHP-style expression delimiter — RCE probe |
| `mb_ereg_replace` | PHP function name — PHP injection payload |
| `file_put_contents` | PHP function name — file write injection payload |

---

## Customization Tips

- Start with `LIMIT_REQ_RATE=5r/s`, then relax to `10r/s` if legitimate users are throttled.
- Keep `USE_ANTIBOT=captcha` for strongest challenge; switch to `cookie` if CAPTCHA UX is too aggressive.
- Raise `BAD_BEHAVIOR_THRESHOLD` (default 5) if your API endpoints naturally produce more 4xx responses.
- Keep ModSecurity CRS enabled; add exclusion rules only for confirmed false positives.
- The `mitchellkrogza-bad-user-agents` community list is updated regularly — no manual maintenance needed.
- Review new AI crawler announcements quarterly: each major model release typically introduces a new crawler token.

## nginx / reverse proxy equivalent

For operators not using BunkerWeb, the equivalent nginx `map` block:

```nginx
map $http_user_agent $block_bot {
    default             0;
    "~*GPTBot"          1;
    "~*ChatGPT-User"    1;
    "~*OAI-SearchBot"   1;
    "~*ClaudeBot"       1;
    "~*Claude-Web"      1;
    "~*anthropic-ai"    1;
    "~*Google-Extended" 1;
    "~*Bytespider"      1;
    "~*CCBot"           1;
    "~*Diffbot"         1;
    "~*Applebot-Extended" 1;
    "~*PetalBot"        1;
    "~*meta-externalagent" 1;
    "~*meta-webindexer" 1;
    "~*PerplexityBot"   1;
    "~*cohere-ai"       1;
    "~*YouBot"          1;
    "~*AI2Bot"          1;
    "~*DataForSeoBot"   1;
    "~*omgilibot"       1;
    "~*Meltwater"       1;
    "~*peer39_crawler"  1;
    "~*magpie-crawler"  1;
    "~*AwarioSmartBot"  1;
    "~*CensysInspect"   1;
    "~*RecordedFuture"  1;
    "~*Infrawatch"      1;
    "~*python-requests" 1;
    "~*python-urllib"   1;
    "~*go-http-client"  1;
    "~*libwww-perl"     1;
    "~*okhttp"          1;
    "~*Java/"           1;
}

server {
    # ...
    if ($block_bot) { return 403; }
}
```

> **Note:** Use `return 403` rather than `return 444` (connection drop) on public sites; `444` can cause browser retries that amplify load. Use `444` only in BunkerWeb/nginx for confirmed abusive IPs after bad-behavior banning.

## Validation

```bash
jq . template.json
```
