# TwitchDropMiner Web Add-on

A small, dependency-free web dashboard for
[TwitchDropsMiner-CLI](https://github.com/LCBRST/TwitchDropsMiner-CLI).

The miner serves these files itself from its management API, so there is no
separate server, no build step and no CORS configuration: open the miner's URL
in a browser and sign in with the management token.

## Enable it

In the miner's console:

```
api on          # start the API on :8091 (LAN) and generate a token
api token       # print the token
```

Then either let the miner open the page already signed in:

```
api open       # opens http://127.0.0.1:8091/#token=... in your browser
```

or copy the token to the clipboard and paste it into the page:

```
api copy
```

`api` (or `api status`) shows the address, `api off` stops it, and
`api rotate` issues a new token. The token fragment is read and removed from
the address bar by the page; it is never sent to the server or in a Referer.

The listener binds all interfaces by default (`0.0.0.0:8091`), so a phone or
another computer on the LAN can open it. Every API call still needs the token.
On a network you do not trust, set `api_host` back to `127.0.0.1` to keep the
dashboard on the miner's own machine:

```
set api_host 127.0.0.1
```

Anyone with the token can pause mining,
select channels, claim drops, change settings and exit the miner.

## What it does

- Live engine state, login, watching channel, uptime and websocket status
- Current drop and campaign progress, updated over Server-Sent Events
- A drop-centric inventory with rewards, progress, status filters and claim
  buttons
- Channel list with online/ACL/drops badges, search and one-click watch
- Campaign browser with per-drop rewards, progress and claim buttons
- Settings editor: priority list, excluded games, priority mode, miner
  language, proxy, connection quality, reload interval, badges/emotes,
  available-drops check and the sign-in endpoint
- Account and session panel, wanted-game pills and a live console log
- Start or cancel a Twitch browser sign-in, open the sign-in page it serves,
  and request a session renewal without leaving the dashboard
- Quick actions with a single pause/resume toggle, refresh inventory, stop
  watching, claim ready drops and exit

The UI is available in English, Simplified Chinese and Traditional Chinese and
has light/dark themes. All requests are same-origin.

## Development

There is nothing to compile. The files are plain HTML, CSS and JavaScript:

```
index.html
assets/app.js
assets/styles.css
```

`API.md` documents the HTTP contract the dashboard uses.

The CLI repository vendors a copy under `webui/` so PyInstaller can ship it
inside the single-file binary. After changing this repository, sync it:

```bash
# in a checkout of TwitchDropsMiner-CLI
scripts/sync-webui.sh /path/to/TwitchDropMiner_WebAddon
```

Then rebuild the CLI (`./build_cli.sh` on Linux, `build.bat` on Windows).

## Security

- The token is stored in `localStorage` (or `sessionStorage` if "Remember on
  this browser" is unchecked). "Disconnect" forgets it.
- The API refuses every `/api/*` request without a valid `X-API-Token` header
  (or `Authorization: Bearer <token>`).
- Static files are public, which is what lets the page ask for the token. No
  miner data is included in them.
- Sign-in URLs that carry one-off tokens are never exposed through the API.

## License

MIT — see [LICENSE](LICENSE).
