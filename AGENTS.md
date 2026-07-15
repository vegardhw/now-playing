# AGENTS.md

## Project overview

A simple, single-page "Now Playing" display for music, showing album art with
a blurred background effect. No build step, no dependencies — plain
self-contained HTML/CSS/JS files. Two backends are supported:

- **Home Assistant** (`now-playing-ha.html`) — polls a HA media player entity
  via the REST API (through an NGINX reverse proxy to work around CORS/auth).
- **Music Assistant** (`now-playing-ma.html`) — connects directly via
  WebSocket for real-time updates.
- **`discover-players-ma.html`** — standalone helper page to list Music
  Assistant player IDs (used to fill in `PLAYER_ID` in the MA display).

## Files

- `now-playing-ha.html` — HA-based now playing display
- `now-playing-ma.html` — MA-based now playing display
- `discover-players-ma.html` — MA player ID discovery tool
- `README.md` — user-facing setup/deployment docs

## Conventions

- Each HTML file is fully self-contained (inline `<script>`/`<style>`), no
  external build tools or package managers involved.
- Configuration lives in a clearly marked `CONFIGURATION` block of constants
  at the top of the `<script>` in each file (e.g. `DEV_MODE`, `MEDIA_PLAYER`,
  `MA_ACCESS_TOKEN`, `PLAYER_ID`, `MA_WS_URL`, `MA_BASE_URL`).
- `DEV_MODE = true` renders static test data so the UI can be checked without
  a live HA/MA connection — keep this working when editing display logic.
- Keep the two now-playing pages visually/behaviorally consistent (same
  blurred-background layout and animation) even though their data sources
  differ.

## When making changes

- Update `README.md` if you add/rename configuration variables or files.
- Test changes by opening the HTML file directly in a browser with
  `DEV_MODE = true` before wiring up a real HA/MA backend.
- Preserve backward compatibility of config variable names since users edit
  these files directly when deploying.
