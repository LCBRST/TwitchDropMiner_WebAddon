# Management API

The miner's management API listens on `api_host:api_port` (default
`0.0.0.0:8091`, so LAN devices can reach it; set `api_host` to `127.0.0.1` to
restrict it to the miner's machine). Every `/api/*` request needs the token
printed by `api token`:

```
X-API-Token: <token>
```

`Authorization: Bearer <token>` is accepted as well. `GET /` and `GET /assets/*`
are public so the dashboard can load and ask for the token. `GET /auth` returns
`{"required": bool}` without auth, and the dashboard uses it to skip the token
prompt when `api auth off` / `api_auth_enabled false` is set. `GET /healthz`
returns `{"ok": true}` without auth.

Errors are JSON: `{"error": "message"}` with a 4xx status.

`/login`, `/login/{token}`, `/login/state`, `/login/ready`, `/login/cancel` and
the `/login/ws` WebSocket are proxied to the sign-in endpoint, so the embedded
viewer works on the dashboard's own port. They keep the sign-in endpoint's
one-off token and cookie; the proxy does not add authentication.

## Read

| Method | Path | Response |
|---|---|---|
| GET | `/api/status` | engine state, watching channel, current drop progress, counts, login, websockets, version, uptime |
| GET | `/api/channels` | `{"channels": [...]}` — id, login, name, online, game, viewers, ACL/drops flags, watching/selected, `can_watch` |
| GET | `/api/campaigns` | `{"campaigns": [...]}` — campaign metadata, progress and a `drops` array with minutes/progress/claimable |
| GET | `/api/settings` | `{"settings": {...}, "api": {...}}`; `api_token` is `{"configured": bool}`, never the token |
| GET | `/api/login` | sign-in/session/renewal status, including the one-off `urls` of the sign-in page while it is listening |
| GET | `/api/logs?limit=200` | `{"lines": [...]}` — the console output buffer (1–1000 lines) |
| GET | `/api/events` | Server-Sent Events: `event: status` every 2 s, `event: logs` when the buffer changes |

## Write

| Method | Path | Body |
|---|---|---|
| POST | `/api/channels/select` | `{"id": 123}` or `{"login": "name"}` |
| POST | `/api/settings` | partial settings object (see below) |
| POST | `/api/actions/pause` | — (`stop_watching` + IDLE) |
| POST | `/api/actions/resume` | — (re-fetch inventory) |
| POST | `/api/actions/refresh` | — (re-fetch inventory) |
| POST | `/api/actions/unwatch` | — |
| POST | `/api/actions/exit` | — (shut the miner down) |
| POST | `/api/actions/claim` | `{}` (all claimable) or `{"drop_id": "..."}` |
| POST | `/api/actions/login-browser` | — (open the built-in browser sign-in; 409 `{reason, message}` when it cannot start) |
| POST | `/api/actions/login-renew` | — (renew the imported session) |
| POST | `/api/actions/login-cancel` | — (cancel a running sign-in) |

### Writable settings

`priority` (list), `exclude` (list), `priority_mode` (`priority_only`,
`ending_soonest`, `low_avbl_first`), `language`, `proxy`, `connection_quality`
(0–2), `reload_interval` (10–1440), `enable_badges_emotes`,
`available_drops_check`, `login_server_enabled`,
`login_server_host`, `login_server_port`, `renewal_browser_path`.

`api_enabled`, `api_host`, `api_port` and `api_token` are intentionally
read-only over HTTP: changing the listener from one of its own requests would
drop the connection mid-response. Change them from the CLI (`api on`,
`api rotate`, `set api_port 8091`).

Setting `priority`, `exclude` or `priority_mode` immediately rebuilds the wanted
games and re-picks a channel; other values apply at the next reload or restart.
