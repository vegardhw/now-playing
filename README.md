# Now Playing

A "Now Playing" display featuring album artwork with a blurred background effect. Supports both **Home Assistant** and **Music Assistant** as sources.

## Files

| File | Description |
|------|-------------|
| `now-playing-ha.html` | Now Playing display via Home Assistant API |
| `now-playing-ma.html` | Now Playing display via Music Assistant WebSocket |
| `discover-players-ma.html` | Helper tool to list available Music Assistant player IDs |
| `npm-ha.conf` | Example NGINX Proxy Manager custom location config for `now-playing-ha.html` |
| `npm-ma.conf` | Example NGINX Proxy Manager custom location config for `now-playing-ma.html` |

---

## now-playing-ha.html

Polls a Home Assistant media player entity every 2 seconds and displays the current track.

**Configuration** — edit the block at the top of the file:

| Variable | Description |
|----------|-------------|
| `DEV_MODE` | `true` shows static test data; set to `false` for production |
| `MEDIA_PLAYER` | HA entity ID, e.g. `media_player.sonos_roam` |
| `API_PATH` | NGINX proxy path forwarding to the HA API, e.g. `/api/ha` |

### NGINX Proxy Manager Setup

Because the page runs in the browser it cannot call the HA API directly (CORS). Use a reverse proxy to forward requests and inject the auth token server-side. See `npm-ha.conf` for a full example location block.

1. In NGINX Proxy Manager, open your proxy host → **Custom locations**
2. Add a location:
   - **Location**: `/api/ha` (match `API_PATH`)
   - **Forward**: `http://<HA_IP>:8123`
3. Paste the contents of `npm-ha.conf` as the custom NGINX config, replacing `<token-here>` and the backend IP/port.

To get a token: HA profile → **Long-Lived Access Tokens** → **Create Token**.

---

## now-playing-ma.html

Connects to Music Assistant over WebSocket and displays the current track in real time.

**Configuration** — edit the block at the top of the file:

| Variable | Description |
|----------|-------------|
| `DEV_MODE` | `true` shows static test data; set to `false` for production |
| `PLAYER_ID` | MA player ID to monitor (use `discover-players-ma.html` to find it) |
| `API_PATH` | NGINX proxy path forwarding to the MA server, e.g. `/api/ma` |

The WebSocket (`{API_PATH}/ws`) and image proxy (`{API_PATH}/imageproxy`) requests are both built as same-origin relative URLs from `API_PATH`, so the browser automatically uses `wss:`/`https:` when the page itself is served over TLS.

Unlike Home Assistant, there's no `MA_ACCESS_TOKEN` field to fill in — the PAT is never stored in this file or exposed in the page source. Instead, NGINX Proxy Manager injects it server-side into every proxied request (see below), exactly like the `Authorization` header trick used for HA.

### NGINX Proxy Manager Setup

As with Home Assistant, the browser talks only to a same-origin path; NGINX Proxy Manager forwards it (including the WebSocket upgrade) to the real MA server and appends the access token itself. See `npm-ma.conf` for a full example location block, written in the same style as `npm-ha.conf` (used for `now-playing-ha.html`).

Key differences from the HA config:

- MA authenticates via an `access_token` **query-string parameter**, not an `Authorization` header, so the token is injected via `rewrite` instead of `proxy_set_header`.
- MA's real endpoints (`/ws`, `/imageproxy`) live at the server root with no prefix, so the rewrite strips `/api/ma` entirely (unlike HA, which keeps its `/api` prefix).
- The WebSocket is a real, long-lived connection here (not unused boilerplate like in the HA config), so `proxy_read_timeout`/`proxy_send_timeout` are increased well beyond the default 60s to avoid nginx dropping an idle-but-open connection (e.g. during mobile/kiosk tab throttling).

1. In NGINX Proxy Manager, open your proxy host → **Custom locations**
2. Add a location:
   - **Location**: `/api/ma` (match `API_PATH`)
   - **Forward**: `http://<MA_IP>:8095`
3. Paste the contents of `npm-ma.conf` as the custom NGINX config, replacing `<MA_TOKEN_HERE>` and the backend IP/port.

Create the PAT under Music Assistant's user settings, same as before — it's just configured in NPM now instead of in the HTML file.

---

## discover-players-ma.html

Connects to Music Assistant and lists all available players with their IDs. Use this to look up the value for `PLAYER_ID` in `now-playing-ma.html`.

This is a one-time discovery tool, so it connects **directly** to your MA server via its LAN `ws://` address rather than going through a reverse proxy — no NGINX setup required. Run it locally (e.g. open the file directly in a browser, or serve it over plain HTTP) rather than over HTTPS, since browsers block a plain `ws://` connection from a page loaded over `https://` (mixed content).

**Configuration** — edit the two variables at the top of the script block:

| Variable | Description |
|----------|-------------|
| `MA_ACCESS_TOKEN` | Music Assistant PAT (used only here — `now-playing-ma.html` no longer stores one, see above) |
| `MA_WS_URL` | Direct WebSocket URL to your MA server, e.g. `ws://192.168.1.20:8095/ws` |

---

## Deployment

1. Copy the desired HTML file(s) to your web server
2. Set `DEV_MODE` to `false` and fill in all configuration variables
3. For `now-playing-ha.html` and `now-playing-ma.html`, configure the corresponding NGINX Proxy Manager custom location as described above
4. Access the page through your web server

## Usage Ideas

- **Kiosk display**: Run on a Raspberry Pi connected to a monitor
- **Dashboard panel**: Embed as a webpage card in Home Assistant or MA
- **Smart mirror**: Integrate into a magic mirror display

## License

MIT
