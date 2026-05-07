# Anti-claw / Anti-bot Template

## Overview

Apply a strict BunkerWeb hardening profile focused on crawler, scraper, and abusive bot traffic. The template enables CAPTCHA challenge mode, DNSBL checks, aggressive request throttling, and automatic bad-behavior bans.

## Prerequisites

- A domain name for the protected service.
- BunkerWeb deployed with templates support.
- Awareness that stricter limits may affect legitimate burst traffic.

## Files

- `template.json` - Hardened anti-bot profile and guided configuration steps.

## Setup

1. Import `template.json` from the BunkerWeb `Templates` page in **Raw** mode.
2. Apply `USE_TEMPLATE=anti-claw-bot` to the target service.
3. Set `SERVER_NAME` for your domain.
4. Reload and monitor BunkerWeb logs for false positives.

## Customization Tips

- Start with `LIMIT_REQ_RATE=5r/s`, then relax if legitimate users are throttled.
- Keep `USE_ANTIBOT=captcha` for strongest challenge mode.
- Raise `BAD_BEHAVIOR_THRESHOLD` if your API endpoints naturally produce more 4xx traffic.
- Keep ModSecurity CRS enabled and tune only if needed.

## Validation

Run `jq . template.json` before import to ensure the JSON is valid.
