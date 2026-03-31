# Now Playing

A "Now Playing" display featuring album artwork with a blurred background effect. Supports both **Home Assistant** and **Music Assistant** as sources.

## Files

| File | Description |
|------|-------------|
| `now-playing-ha.html` | Now Playing display via Home Assistant API |
| `now-playing-ma.html` | Now Playing display via Music Assistant WebSocket |
| `discover-players-ma.html` | Helper tool to list available Music Assistant player IDs |

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

Because the page runs in the browser it cannot call the HA API directly (CORS). Use a reverse proxy to forward requests and inject the auth token server-side.

1. In NGINX Proxy Manager, open your proxy host → **Custom locations**
2. Add a location:
   - **Location**: `/api/ha` (match `API_PATH`)
   - **Forward**: `http://<HA_IP>:8123`
3. Add custom NGINX config:

```nginx
rewrite ^/api/ha/(.*) /$1 break;
proxy_set_header Authorization "Bearer YOUR_LONG_LIVED_ACCESS_TOKEN";
proxy_set_header Host $host;
```

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
| `MA_WS_URL` | WebSocket URL, e.g. `ws://192.168.1.20:8095/ws` |
| `MA_BASE_URL` | HTTP base URL for image proxying, e.g. `http://192.168.1.20:8095` |

---

## discover-players-ma.html

Connects to Music Assistant and lists all available players with their IDs. Use this to look up the value for `PLAYER_ID` in `now-playing-ma.html`.

**Configuration** — edit the two variables at the top of the script block:

| Variable | Description |
|----------|-------------|
| `MA_ACCESS_TOKEN` | Same PAT used in `now-playing-ma.html` |
| `MA_WS_URL` | Same WebSocket URL used in `now-playing-ma.html` |

---

## Deployment

1. Copy the desired HTML file(s) to your web server
2. Set `DEV_MODE` to `false` and fill in all configuration variables
3. For `now-playing-ha.html`, configure the NGINX proxy as described above
4. Access the page through your web server

## Usage Ideas

- **Kiosk display**: Run on a Raspberry Pi connected to a monitor
- **Dashboard panel**: Embed as a webpage card in Home Assistant or MA
- **Smart mirror**: Integrate into a magic mirror display

## License

MIT
