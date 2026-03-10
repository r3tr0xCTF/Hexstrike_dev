# HexStrike AI — gh057x/dev Fork

> Personal development branch of [HexStrike AI](https://github.com/0x4m4/hexstrike-ai) — a modular AI-powered penetration testing framework built on the MCP (Model Context Protocol) stack.
>
> This fork extends the original with a full **XSS C2 attack chain**: JS-Tap C2 integration, automatic tunnel/redirector support, dalfox+kxss discovery pipeline, JS secret mining, and loot intelligence analysis.

---

## What's New in This Fork

| Module | What it does |
|---|---|
| **JS-Tap C2 Integration** | Embeds JS-Tap XSS C2 server; generates & delivers `telemlib.js` implant payloads |
| **Tunnel Manager** | Exposes the C2 via ngrok / cloudflared / serveo / localhost.run automatically |
| **Redirector Support** | Generates Apache/Nginx reverse-proxy configs to front the C2 |
| **Beacon Listener** | Polls JS-Tap for live sessions and pulls all captured loot |
| **XSS Auto-Inject** | Automatically injects JS-Tap payloads into discovered parameters |
| **Payload Obfuscation** | 6 techniques: base64_eval, fromcharcode, hex_url, unicode_url, string_split, fetch_dynamic |
| **dalfox + kxss Pipeline** | katana crawl → kxss reflection filter → dalfox `--blind=telemlib_url` for reflected + blind/stored XSS in one pass |
| **JS Secret Miner** | Crawls all JS files on a target; detects AWS keys, JWTs, API keys, hardcoded passwords, private keys, and more |
| **Loot Intelligence Engine** | Analyzes captured JS-Tap sessions for credentials, auth tokens, secrets across cookies, storage, XHR calls, and form posts |
| **Engagement Reporter** | Generates structured engagement reports from all findings |

---

## Full XSS Attack Chain

```
target
  │
  ├─ js_secret_mine ──────────────────────── secrets in JS files
  │     katana crawl → 18+ regex patterns
  │     + trufflehog + nuclei exposures
  │
  └─ dalfox_xss_chain ────────────────────── XSS discovery + C2
        │
        ├─ tunnel_start (ngrok/cloudflared)
        ├─ jstap_start_server
        │
        ├─ katana (JS-aware crawl)
        ├─ waybackurls (historical URLs)
        ├─ kxss (reflection pre-filter)
        ├─ dalfox --blind=<telemlib_url>
        │     ↑ implant delivered automatically
        │     for reflected AND blind/stored XSS
        │
        ├─ beacon_list_clients
        └─ loot_analyze (per client_id)
              cookies · localStorage · XHR headers
              form posts · user inputs → findings
```

---

## MCP Tools Added

### JS-Tap C2
| Tool | Description |
|---|---|
| `jstap_start_server` | Start the JS-Tap C2 server |
| `jstap_stop_server` | Stop JS-Tap |
| `jstap_server_status` | Get server status and active session count |
| `jstap_generate_payload` | Generate a JS-Tap implant payload for a target |

### Tunnels & Infrastructure
| Tool | Description |
|---|---|
| `tunnel_start` | Start a tunnel (ngrok/cloudflared/serveo/localhost.run) |
| `tunnel_stop` | Stop the active tunnel |
| `tunnel_status` | Get tunnel status and public URL |
| `jstap_launch_with_tunnel` | Start JS-Tap + tunnel in one step |
| `redirector_generate` | Generate Apache/Nginx redirector config |

### Beacon & Session Management
| Tool | Description |
|---|---|
| `beacon_start` | Start polling JS-Tap for new sessions |
| `beacon_stop` | Stop beacon polling |
| `beacon_list_clients` | List all active JS-Tap clients |
| `beacon_get_loot` | Fetch raw loot for a client ID |

### XSS Discovery & Delivery
| Tool | Description |
|---|---|
| `xss_auto_inject` | Auto-inject JS-Tap payload into discovered parameters |
| `xss_full_chain` | Full XSS chain: inject + tunnel + C2 + beacon |
| `kxss_check` | Fast reflection check on a list of URLs |
| `dalfox_xss_chain` | Full pipeline: crawl → kxss → dalfox → JS-Tap → loot |
| `payload_obfuscate` | Obfuscate a JS payload with chosen technique |

### Intelligence & Reporting
| Tool | Description |
|---|---|
| `js_secret_mine` | Crawl all JS files on target and scan for hardcoded secrets |
| `loot_analyze` | Analyze a JS-Tap session for credentials and sensitive data |
| `engagement_report` | Generate a structured engagement report |
| `nuclei_to_xss_chain` | Legacy nuclei → XSS pipeline |

---

## JS Secret Miner — Detected Patterns

| Pattern | Severity |
|---|---|
| AWS Access Key (`AKIA...`) | CRITICAL |
| GitHub Token / PAT | CRITICAL |
| Stripe Live Secret Key | CRITICAL |
| SendGrid API Key | CRITICAL |
| Private Key Header | CRITICAL |
| JWT Token | HIGH |
| Google API Key | HIGH |
| Slack Token | HIGH |
| Twilio Auth Token | HIGH |
| Firebase Key | HIGH |
| Hardcoded Password / API Key | HIGH |
| Basic Auth in URL | HIGH |
| Mailchimp Key | HIGH |
| Internal Endpoint Leak | MEDIUM |
| Generic Bearer / Token | MEDIUM |
| Stripe Public Key | MEDIUM |

Optional deeper scanning via **trufflehog** and **nuclei exposures/** templates if installed.

---

## Dependencies

### Python packages
```bash
pip install -r requirements.txt
```

### External tools used by new modules
```bash
# XSS pipeline
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/Emoe/kxss@latest
go install github.com/hahwul/dalfox/v2@latest
go install github.com/tomnomnom/waybackurls@latest

# Secret mining (optional)
brew install trufflehog          # or: curl -sSfL https://trufflehog.io/install.sh | sh
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Tunnels (install at least one)
# ngrok:       https://ngrok.com/download
# cloudflared: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/
```

### JS-Tap (git submodule)
```bash
git submodule update --init --recursive
cd tools/jstap
pip install -r requirements.txt
```

---

## Setup

```bash
# Clone with submodules
git clone --recurse-submodules <your-repo-url>
cd hexstrike-ai

# Python environment
python -m venv env
source env/bin/activate        # Windows: env\Scripts\activate
pip install -r requirements.txt

# Start the server
python hexstrike_server.py

# In a second terminal — start the MCP client
python hexstrike_mcp.py
```

---

## Branch History

```
e007800  feat: JS secret miner and loot intelligence engine
6d7d9e2  feat: replace nuclei XSS pipeline with dalfox+kxss
9130cad  feat: payload obfuscation, engagement reporting, nuclei→XSS pipeline
8466a85  feat: redirector support, beacon listener, XSS auto-inject
2408f21  feat: add TunnelManager for internet-accessible C2 delivery
7613407  chore: convert tools/jstap to proper git submodule
9703455  feat: integrate JS-Tap XSS C2 into HexStrike AI
         ── upstream HexStrike AI commits below ──
```

---

## Upstream Project

This fork is based on **[HexStrike AI v6.0](https://github.com/0x4m4/hexstrike-ai)** by [@0x4m4](https://github.com/0x4m4).
Original license: MIT — see [LICENSE](LICENSE).

**JS-Tap** is included as a submodule: [github.com/hoodoer/JS-Tap](https://github.com/hoodoer/JS-Tap).

---

## Disclaimer

This tool is intended for **authorized security testing and research only**.
Always obtain written permission before testing any system you do not own.
