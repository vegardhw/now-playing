# Now Playing

A beautiful "Now Playing" display for Home Assistant media players, featuring album artwork with a blurred background effect.

![Now Playing Display](https://via.placeholder.com/800x500/1a1a1a/ffffff?text=Now+Playing+Display)

## Overview

This is a single HTML file that displays the currently playing track from a Home Assistant media player (e.g., Sonos, Chromecast, etc.). It features:

- Large album artwork display
- Track title, artist, and album name
- Blurred album art background with subtle animation
- Auto-updates every 2 seconds
- Dark theme optimized for displays/kiosks
- Responsive design for various screen sizes

## Requirements

- **Home Assistant** with a media player entity
- **NGINX Proxy Manager** (or similar reverse proxy) to handle API authentication
- A web server to host the HTML file (can be the same NGINX)

## Configuration

Edit the configuration section at the top of `now-playing.html`:

```javascript
// ============================================
// CONFIGURATION - Edit these values as needed
// ============================================

// Set to true to use test data without Home Assistant connection
const DEV_MODE = true;

// Home Assistant media player entity ID
const MEDIA_PLAYER = 'media_player.sonos_roam';

// NGINX proxy path for Home Assistant API
const API_PATH = '/ha-api';

// ============================================
```

| Setting | Description |
|---------|-------------|
| `DEV_MODE` | Set to `true` to display test data without connecting to Home Assistant. Useful for development and testing the visual layout. |
| `MEDIA_PLAYER` | The entity ID of your Home Assistant media player (e.g., `media_player.sonos_roam`, `media_player.living_room_speaker`). |
| `API_PATH` | The proxy path configured in NGINX that forwards requests to Home Assistant. |

## NGINX Proxy Manager Setup

Since the HTML runs in the browser, it cannot directly access Home Assistant's API due to CORS restrictions. You need to set up a reverse proxy that:

1. Forwards API requests to Home Assistant
2. Injects the authentication token server-side

### Setup Steps

1. In NGINX Proxy Manager, edit your proxy host
2. Go to **Custom locations** tab
3. Add a new location:
   - **Location**: `/ha-api`
   - **Scheme**: `http`
   - **Forward Hostname/IP**: Your Home Assistant IP (e.g., `10.0.5.50`)
   - **Forward Port**: `8123`
4. Click the ⚙️ gear icon and add this custom configuration:

```nginx
rewrite ^/ha-api/(.*) /$1 break;
proxy_set_header Authorization "Bearer YOUR_LONG_LIVED_ACCESS_TOKEN";
proxy_set_header Host $host;
```

5. Replace `YOUR_LONG_LIVED_ACCESS_TOKEN` with your actual Home Assistant long-lived access token

### Getting a Home Assistant Token

1. Go to your Home Assistant profile (click your username in the sidebar)
2. Scroll down to **Long-Lived Access Tokens**
3. Click **Create Token**
4. Give it a name (e.g., "Now Playing Display")
5. Copy the token and use it in the NGINX configuration

## Development Mode

When `DEV_MODE` is set to `true`, the page displays static test data with a random album art image. This is useful for:

- Testing the visual design without Home Assistant
- Developing on a machine that can't reach your Home Assistant instance
- Demonstrating the display without a live media player

Set `DEV_MODE` to `false` when deploying for production use.

## Deployment

1. Copy `now-playing.html` to your web server
2. Configure NGINX Proxy Manager as described above
3. Set `DEV_MODE` to `false`
4. Update `MEDIA_PLAYER` with your entity ID
5. Access the page through your web server

## Usage Ideas

- **Kiosk display**: Run on a Raspberry Pi connected to a monitor
- **Dashboard panel**: Embed in Home Assistant as a webpage card
- **Smart mirror**: Integrate into a magic mirror display

## License

MIT
