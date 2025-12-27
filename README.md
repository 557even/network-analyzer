# network-analyzer
Interactive Network Analyzer + Webmaster Utils
Single-file, retro terminal network + webmaster utilities page.

This is a **client-side** tool intended for **authorized** diagnostics on systems/domains you own or have permission to test.

## Features
- **Get Public IP** (ipify)
- **Ping (Browser)** (approximate timing via `fetch()`; can be affected by CORS, HTTPS, firewall, DNS)
- **Connection Info** (Navigator + Network Information API when available)
- **WHOIS / RDAP** (RDAP over HTTPS; resolves hostname → A record first)
- **ARIN Lookup** (ARIN RDAP for IP networks)
- **Traceroute** (runs from a **remote probe** via Globalping, not your device)
- **Theme toggle** (dark/light)
- **Copy output**

## Files
- `index.html` — everything (UI + logic)

## Run locally
### Option A: open the file
Just open `index.html` in a browser.

### Option B: serve it (recommended)
```bash
# Python
python -m http.server 8000
# then open http://localhost:8000
