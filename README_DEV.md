# HexStrike AI — gh057x/dev Fork

> Personal development branch of [HexStrike AI](https://github.com/0x4m4/hexstrike-ai) — a modular AI-powered penetration testing framework built on the MCP (Model Context Protocol) stack.
>
> This fork extends the original with a comprehensive offensive toolkit spanning the full pentest lifecycle: XSS C2 delivery, secret mining, authentication attacks, infrastructure analysis, and protocol-level exploitation.

---

## What's New in This Fork

### XSS C2 Chain
| Module | What it does |
|---|---|
| **JS-Tap C2 Integration** | Embeds JS-Tap XSS C2 server; generates & delivers `telemlib.js` implant payloads |
| **Tunnel Manager** | Exposes the C2 via ngrok / cloudflared / serveo / localhost.run automatically |
| **Redirector Support** | Generates Apache/Nginx reverse-proxy configs to front the C2 |
| **Beacon Listener** | Polls JS-Tap for live sessions and pulls all captured loot |
| **XSS Auto-Inject** | Automatically injects JS-Tap payloads into discovered parameters |
| **Payload Obfuscation** | 6 techniques: base64_eval, fromcharcode, hex_url, unicode_url, string_split, fetch_dynamic |
| **dalfox + kxss Pipeline** | katana crawl → kxss reflection filter → dalfox `--blind=telemlib_url` for reflected + blind/stored XSS in one pass |

### Intelligence & Recon
| Module | What it does |
|---|---|
| **JS Secret Miner** | Crawls all JS files on a target; detects AWS keys, JWTs, API keys, hardcoded passwords, private keys, and more |
| **Loot Intelligence Engine** | Analyzes captured JS-Tap sessions for credentials, auth tokens, secrets across cookies, storage, XHR calls, and form posts |
| **Session Hijack Validator** | Replays captured cookies against 22 auth endpoints; confirms admin access and discovers usernames |
| **Auto-Recon Pipeline** | One-call domain-to-exploitation: subfinder → httpx → parallel CSP+secrets → dalfox XSS → loot analysis |
| **Subdomain Enumeration** | Multi-tool pipeline: subfinder + amass + dnsx + shuffledns + alteration; wildcard filtering, live host probing, screenshot capture |

### Web Vulnerability Scanners
| Module | What it does |
|---|---|
| **CSP Analyzer** | Parses Content-Security-Policy headers; 10 weakness checks including unsafe-inline, trusted CDN bypass, static nonce |
| **Prototype Pollution Scanner** | ppfuzz + ppmap + manual `__proto__` reflection probes; 6 XSS gadget chain escalations |
| **CORS Tester** | 14 origin variants per endpoint; CRITICAL/HIGH/MEDIUM classification; generates `fetch()` PoC snippets |
| **SSTI Scanner** | 8 detection probes across all major template engines; tplmap integration; RCE confirmation |
| **SSRF Scanner** | 30+ param names; direct internal probes (AWS/GCP/Azure metadata, Redis, Docker, k8s); blind OOB token injection; 8 IP bypass variants |
| **GraphQL Recon** | 7-stage pipeline: endpoint discovery (18 paths), engine fingerprinting (12 engines), introspection, clairvoyance, 6 vuln checks, schema analysis, InQL |
| **Cache Poisoning** | 7 techniques: unkeyed header injection (20 headers), fat GET, web cache deception, parameter cloaking, Vary analysis, HOP-by-HOP stripping, path confusion |
| **XXE Injection** | Blind + OOB (interactsh/collaborator), DTD-based file exfil, SVG/DOCX/XLSX vectors, error-based, parameter entity, SSRF via XXE, protocol handlers |

### Authentication & Token Attacks
| Module | What it does |
|---|---|
| **JWT Attack Suite** | alg:none (15 variants), HMAC brute-force (34 secrets), RS256→HS256 confusion, kid injection (SQL/traversal/cmd/SSRF), jku/x5u URL injection, embedded JWK, exp/nbf manipulation, endpoint replay |
| **OAuth/OIDC Tester** | OIDC discovery (8 paths), redirect_uri bypass (14 templates), state CSRF, PKCE downgrade, implicit flow leakage, scope creep (18 scopes), token endpoint misconfig, code reuse |
| **API Key Bruteforcer** | 24 API key headers enumerated; 34 built-in common secrets; rate-limit profiling + UA/IP rotation bypass; query param spray (14 names) |

### Protocol & Logic Testing
| Module | What it does |
|---|---|
| **HTTP Request Smuggling** | CL.TE, TE.CL, TE.TE (12 obfuscation variants), H2 downgrade, timing-based blind — all via raw sockets |
| **Business Logic Fuzzer** | Price/amount tampering, negative quantities, coupon abuse, workflow skipping, race conditions (N concurrent), parameter pollution, hidden params, mass assignment |
| **WebSocket Security** | Endpoint discovery (17 paths), auth bypass (6 variants), origin validation/CSWSH, PoC generation, message injection (XSS/SQLi/cmd/SSTI/proto pollution), subprotocol abuse |

### Reporting
| Module | What it does |
|---|---|
| **Engagement Reporter** | Generates structured engagement reports from all findings |

---

## Full Attack Chains

### XSS C2 Chain
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

### Auto-Recon Pipeline
```
target
  │
  ├─ Stage 1: subdomain_enum ─────────────── subfinder + amass + dnsx
  │     + shuffledns + alteration bruteforce
  │     → wildcard filter → httpx live probe
  │
  ├─ Stage 2 (parallel): ─────────────────── per live host
  │     ├─ csp_analyze
  │     └─ js_secret_mine
  │
  ├─ Stage 3: prioritize ─────────────────── rank by attack surface
  │
  ├─ Stage 4: dalfox_xss_chain ───────────── XSS campaign on top hosts
  │     + jstap_launch_with_tunnel
  │
  └─ Stage 5: post-exploitation
        ├─ loot_analyze (per beacon session)
        └─ session_hijack (per client)
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

### Intelligence & Recon
| Tool | Description |
|---|---|
| `js_secret_mine` | Crawl all JS files on target and scan for hardcoded secrets |
| `loot_analyze` | Analyze a JS-Tap session for credentials and sensitive data |
| `session_hijack` | Replay captured cookies against auth endpoints |
| `auto_recon` | Full domain-to-exploitation pipeline in one call |
| `subdomain_enum` | Multi-tool subdomain enumeration with live probing and screenshots |
| `engagement_report` | Generate a structured engagement report |
| `nuclei_to_xss_chain` | Legacy nuclei → XSS pipeline |

### Web Vulnerability Scanners
| Tool | Description |
|---|---|
| `csp_analyze` | Parse and audit Content-Security-Policy headers |
| `proto_pollution_scan` | Detect prototype pollution and escalate to XSS gadget chains |
| `cors_test` | Test CORS misconfigurations with 14 origin bypass variants |
| `ssti_scan` | Detect and confirm Server-Side Template Injection with RCE |
| `ssrf_scan` | SSRF detection: direct internal probes + blind OOB token injection |
| `graphql_recon` | Full GraphQL recon: endpoint discovery, introspection, vuln checks |
| `cache_poison` | Web cache poisoning: 7 techniques including unkeyed headers and deception |
| `xxe_scan` | XXE injection: blind/OOB, DTD exfil, SVG/DOCX vectors, protocol handlers |

### Authentication & Token Attacks
| Tool | Description |
|---|---|
| `jwt_attack` | Full JWT attack suite: alg:none, brute-force, confusion, kid/jku injection |
| `oauth_test` | OAuth/OIDC misconfiguration: redirect bypass, PKCE, implicit flow, scope creep |
| `api_key_bruteforce` | Enumerate API key headers and spray common secrets with RL bypass |

### Protocol & Logic Testing
| Tool | Description |
|---|---|
| `http_smuggle` | HTTP request smuggling: CL.TE, TE.CL, TE.TE, H2 downgrade, timing blind |
| `biz_logic_fuzz` | Business logic: price tampering, race conditions, workflow skip, mass assignment |
| `websocket_test` | WebSocket security: auth bypass, CSWSH, message injection, subprotocol abuse |

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

### External tools — core pipeline
```bash
# XSS pipeline
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/Emoe/kxss@latest
go install github.com/hahwul/dalfox/v2@latest
go install github.com/tomnomnom/waybackurls@latest

# Subdomain enumeration
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install github.com/projectdiscovery/shuffledns/cmd/shuffledns@latest
go install github.com/owasp-amass/amass/v4/...@master

# GraphQL
pip install graphw00f
pip install clairvoyance
pip install inql

# Prototype pollution
go install github.com/dwisiswant0/ppfuzz@latest
go install github.com/kleiton0x00/ppmap@latest

# Subdomain takeover
go install github.com/PentestPad/subzy@latest

# SSRF / OOB callbacks
# interactsh-client: https://github.com/projectdiscovery/interactsh

# Secret mining (optional)
curl -sSfL https://trufflehog.io/install.sh | sh
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Screenshots (optional)
go install github.com/sensepost/gowitness@latest
```

### Tunnels (install at least one)
```bash
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

# Start the server (port 8888)
python hexstrike_server.py

# In a second terminal — start the MCP client
python hexstrike_mcp.py
```

---

## Branch History

```
f4b9fc1  fix: use tempfile.gettempdir() instead of hardcoded /tmp (Windows compat)
39799c5  feat: HTTP smuggling, business logic fuzzer, WebSocket security tester
36d7565  feat: JWT attack suite and OAuth/OIDC misconfiguration tester
e02f2d8  feat: cache poisoning scanner and API key bruteforcer
a7c41a5  feat: GraphQL recon — endpoint discovery, engine fingerprinting, vuln checks
e007800  feat: JS secret miner and loot intelligence engine
         (+ session hijack, CSP analyzer, auto-recon, prototype pollution,
            CORS, SSTI, subdomain takeover, SSRF — added in same session)
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
