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
| `MA_ACCESS_TOKEN` | Music Assistant Personal Access Token (PAT) |
| `PLAYER_ID` | MA player ID to monitor (use `discover-players-ma.html` to find it) |
| `API_PATH` | NGINX proxy path forwarding to the MA server, e.g. `/api/ma` |

The WebSocket (`{API_PATH}/ws`) and image proxy (`{API_PATH}/imageproxy`) requests are both built as same-origin relative URLs from `API_PATH`, so the browser automatically uses `wss:`/`https:` when the page itself is served over TLS.

Unlike Home Assistant, `MA_ACCESS_TOKEN` **must** be set in this file (it can't be injected server-side). Music Assistant's WebSocket API requires the client to send an in-band `"auth"` JSON command after connecting — a reverse proxy only sees raw WebSocket frames, not the application protocol running inside them, so it has no way to construct/send that command on the client's behalf (unlike HA's REST API, which authenticates via a plain `Authorization` header NGINX can inject on every request). The token will be visible in this file / the page source, same trade-off as `discover-players-ma.html`.

### NGINX Proxy Manager Setup

Even though NGINX can't handle authentication for MA, a reverse proxy is still worth using here to avoid mixed-content errors: the browser talks to a same-origin path, and NGINX forwards it (including the WebSocket upgrade) to the real MA server. See `npm-ma.conf` for a full example location block, written in the same style as `npm-ha.conf` (used for `now-playing-ha.html`).

Key differences from the HA config:

- No token injection here — this location purely handles path rewriting and the WebSocket upgrade.
- MA's real endpoints (`/ws`, `/imageproxy`) live at the server root with no prefix, so the rewrite strips `/api/ma` entirely (unlike HA, which keeps its `/api` prefix).
- The WebSocket is a real, long-lived connection here (not unused boilerplate like in the HA config), so `proxy_read_timeout`/`proxy_send_timeout` are increased well beyond the default 60s to avoid nginx dropping an idle-but-open connection (e.g. during mobile/kiosk tab throttling).

1. In NGINX Proxy Manager, open your proxy host → **Custom locations**
2. Add a location:
   - **Location**: `/api/ma` (match `API_PATH`)
   - **Forward**: `http://<MA_IP>:8095`
3. Paste the contents of `npm-ma.conf` as the custom NGINX config, replacing the backend IP/port.

Create the PAT under Music Assistant's user settings and put it in `MA_ACCESS_TOKEN` in `now-playing-ma.html`.

---

## discover-players-ma.html

Connects to Music Assistant and lists all available players with their IDs. Use this to look up the value for `PLAYER_ID` in `now-playing-ma.html`.

It uses the same same-origin `API_PATH` approach as `now-playing-ma.html`, routed through the same NGINX Proxy Manager `/api/ma` custom location (see `npm-ma.conf`). This avoids the mixed-content `"The operation is insecure"` error that a raw `ws://` URL would trigger when the page is served over HTTPS.

If you'd rather run this one-off tool without going through NPM at all (e.g. quickly open the file locally against a LAN-only MA server), you can instead set `MA_WS_URL` back to a direct `ws://<MA_IP>:8095/ws` URL — just make sure you open the page itself over plain `http://` (or `file://`) in that case, not `https://`, or the browser will block the insecure WebSocket the same way.

**Configuration** — edit the variables at the top of the script block:

| Variable | Description |
|----------|-------------|
| `MA_ACCESS_TOKEN` | Music Assistant PAT (same one used in `now-playing-ma.html`) |
| `API_PATH` | Same NGINX proxy path used in `now-playing-ma.html`, e.g. `/api/ma` |

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
