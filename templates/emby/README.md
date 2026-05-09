# Emby — BunkerWeb hardening template

Profile adapté depuis `anti-claw-bot` pour les serveurs de médias Emby.

## Différences clés vs anti-claw-bot

| Paramètre | anti-claw-bot | emby |
|---|---|---|
| `USE_ANTIBOT` | `captcha` | `no` — les clients Emby (TV, mobile) ne supportent pas le captcha |
| `LIMIT_REQ_RATE` | `5r/s` | `20r/s` — le streaming nécessite plus de requêtes |
| `LIMIT_CONN_MAX_HTTP1` | `5` | `20` — multi-client sur un même foyer |
| `BAD_BEHAVIOR_THRESHOLD` | `5` | `10` — évite les faux positifs sur les 4xx de range-requests |
| `BAD_BEHAVIOR_STATUS_CODES` | inclut `404` | sans `404` — les médias manquants génèrent naturellement des 404 |
| WebSocket | `/socket` | `/embywebsocket` |
| ModSec exclusions | — | API Emby (`/Items`, `/Users`, `/web/config.json`) |

## Prérequis

- `REVERSE_PROXY_HOST` doit pointer vers votre instance Emby (ex. `http://host-gateway:8096` si Emby est sur l'hôte Docker).
- `SERVER_NAME` doit être votre domaine public.

## Utilisation sur z4g4

Voir `z4g4/compose.yaml` — les variables d'environnement dans `bw-scheduler` appliquent ces paramètres directement.
