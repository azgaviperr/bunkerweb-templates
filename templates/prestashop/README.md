# PrestaShop Template

## Overview

Provision a secure baseline for PrestaShop behind BunkerWeb with reverse proxy defaults, anti-bot protection, ModSecurity CRS, and storefront-safe rate limiting.

## Prerequisites

- A running PrestaShop upstream service (for example `http://prestashop`).
- A domain name for your store.
- BunkerWeb UI access (or multisite settings access).

## Files

- `template.json` - BunkerWeb template definition and guided setup steps.

## Setup

1. Import the template in the BunkerWeb `Templates` UI using **Raw** mode.
2. Apply `USE_TEMPLATE=prestashop` to your service.
3. Customize at least `SERVER_NAME` and `REVERSE_PROXY_HOST`.
4. Reload BunkerWeb and validate checkout, customer login, and admin pages.

## Customization Tips

- Increase `MAX_CLIENT_SIZE` if you upload large product images/media.
- Tune `LIMIT_REQ_RATE` if your traffic profile is bursty.
- Switch `USE_ANTIBOT` between `cookie` and `captcha` based on UX tolerance.
- Keep `USE_MODSECURITY_CRS=yes`, then adjust only if specific false positives appear.

## Validation

Run `jq . template.json` before import to ensure the template JSON is valid.
