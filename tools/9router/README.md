# 9router — Independent Security Assessment

**Assessed by:** OM Open AI  
**Package:** [`9router`](https://www.npmjs.com/package/9router) on npm  
**Version reviewed:** `0.5.81`  
**Assessment date:** 2026-09-18  
**Method:** Tarball inspection — package downloaded and extracted, **never installed or executed**  
**Verdict:** ⚠️ Usable with care — legitimate tool, elevated trust requirements

---

## What is 9router?

9router is a **local AI API proxy and router** — a CLI tool that runs a Next.js web server on your machine and sits between your application code and AI provider APIs (OpenAI, Anthropic, Gemini, etc.). It provides:

- Unified API endpoint for multiple AI providers
- Key pooling and rotation across provider accounts
- Request/response logging (MITM proxy mode)
- Web dashboard for monitoring and configuration
- Optional public tunnel via Cloudflare or Tailscale

It is comparable in concept to [LiteLLM](https://github.com/BerriAI/litellm) or [LM Studio](https://lmstudio.ai), but marketed as a CLI-first tool with routing and proxy-pool features.

---

## Assessment Methodology

```bash
# Download tarball — NO install
curl -sL https://registry.npmjs.org/9router/-/9router-0.5.81.tgz -o 9router.tgz

# List contents
tar tzf 9router.tgz

# Extract key files for inspection
tar xzf 9router.tgz package/package.json \
                    package/hooks/postinstall.js \
                    package/cli.js \
                    package/src \
                    package/app/cli/.build-home/.9router/jwt-secret \
                    package/app/cli/.build-home/.9router/machine-id

# Grep for network calls, telemetry, credential patterns
grep -rE "(fetch|https?://|telemetr|analytics|exfil|upload)" package/
grep -rE "machineId|machine_id" package/src/

# Cleanup
rm -rf package 9router.tgz
```

---

## Findings

### Package metadata

| Property | Value | Notes |
|----------|-------|-------|
| Publisher | `decolua <decoluadt@gmail.com>` | Single maintainer, no org |
| Versions | 229 as of assessment | ~1 per day since Jan 2026 |
| First published | 2026-01-03 | ~9 months old |
| Unpacked size | 51 MB | Large for a CLI — contains pre-built Next.js app |
| License | MIT | ✅ |
| Dependencies | `enquirer`, `node-forge`, `node-machine-id`, `react`, `react-dom` | |

---

### Finding 1 — MITM proxy is the core feature

**Severity: Informational (by design)**

The dashboard includes a dedicated MITM proxy view (`/dashboard/mitm`). This is not hidden — it is a documented, navigable feature. However, its implication is significant: **every AI API request you route through 9router is intercepted and logged by the tool**, including prompt content and provider API keys.

This is architecturally identical to running a man-in-the-middle proxy on your own API traffic. If the server component were ever to exfiltrate this data, you would lose both API keys and conversation history. No exfiltration was found in the reviewed code, but the data does pass through the proxy by design.

**What to do:** Use a restricted API key with a spending cap. Never use your primary/production key.

---

### Finding 2 — `postinstall` hook runs at install time without prompt

**Severity: Medium**

`hooks/postinstall.js` executes automatically when you run `npm install -g 9router`. It downloads and installs additional native packages (`better-sqlite3` or `sql.js`, and `systray2` on macOS/Linux) into `~/.9router/runtime/node_modules/` from npm.

```js
// hooks/postinstall.js
const { ensureSqliteRuntime } = require("./sqliteRuntime");
const { ensureTrayRuntime } = require("./trayRuntime");
ensureSqliteRuntime({ silent: false });
ensureTrayRuntime({ silent: false });
```

You are consenting to these secondary downloads when you run `npm install`. The failure is non-fatal (the tool warns and retries at runtime), but you should know this happens.

**What to do:** Inspect `hooks/sqliteRuntime.js` and `hooks/trayRuntime.js` in the tarball before installing if you want to audit every package fetched.

---

### Finding 3 — Bundled default credentials in tarball

**Severity: Low (local-only)**

The tarball ships with two pre-seeded files at `app/cli/.build-home/.9router/`:

- `jwt-secret` — a 64-character hex string
- `machine-id` — a SHA-256 hash

These are installed into `~/.9router/` and used to generate a local CLI authentication token (CLI ↔ local server). They are **not used to authenticate against any remote service** based on the code reviewed. The server regenerates its own secret on first boot; the bundled values appear to be build-time placeholders.

However: because all copies of `0.5.81` ship the same bundled `jwt-secret`, anyone with local access to your `~/.9router/` directory and knowledge of the bundled secret could construct a valid CLI token before the server regenerates it.

**What to do:** No action needed in typical single-user environments. On shared systems, verify `~/.9router/auth/` permissions after install.

---

### Finding 4 — `node-machine-id` reads hardware fingerprint

**Severity: Informational**

`node-machine-id` reads a stable hardware identifier (from `/etc/machine-id` on Linux, the IOPlatformUUID on macOS, or the MachineGuid on Windows). In 9router, this value is used as a component of the local CLI authentication token — it is never observed being sent to an external server.

```js
// src/cli/api/client.js — all requests go to localhost:20128
const DEFAULT_CONFIG = { host: "localhost", port: 20128, protocol: "http:" };
```

The `api/client.js` file hardcodes `localhost` as its only target. No external endpoints were found in the client-side source.

---

### Finding 5 — Default binding to `0.0.0.0`

**Severity: Medium (configuration)**

When started without `--host`, the server binds to `0.0.0.0`, making the dashboard and proxy reachable from any device on your local network. The CLI warns about this:

```
⚠ Network-exposed: reachable at http://192.168.x.x:20128 (bound 0.0.0.0). Use --host 127.0.0.1 for local-only.
```

Anyone on your network who discovers port `20128` can access your AI proxy dashboard and potentially use your configured API keys.

**What to do:** Always start with `--host 127.0.0.1` unless you intentionally want network access.

---

### Finding 6 — Tunnel feature exposes proxy to the internet

**Severity: High (if enabled)**

9router integrates `cloudflared` and `tailscale` to create public tunnels to your local server. If you enable tunneling, your AI proxy — including all configured provider API keys — is reachable from the public internet via a `*.trycloudflare.com` or Tailscale hostname.

**This feature was not enabled by default** in the reviewed version, but it is accessible from the dashboard and CLI flags.

**What to do:** Do not enable tunneling unless you have a specific, understood use case and have secured the dashboard with authentication.

---

### What was NOT found

| Concern | Result |
|---------|--------|
| External telemetry / analytics | ✅ Not found |
| Phone-home on startup | ✅ Not found |
| API key exfiltration to remote servers | ✅ Not found |
| Crypto mining code | ✅ Not found |
| Credential harvesting patterns | ✅ Not found |
| Obfuscated / minified malicious code | ✅ Not found |
| Remote command execution from external server | ✅ Not found |

---

## Risk Summary

| Risk | Level | Condition |
|------|-------|-----------|
| API key exposure | 🔴 High | If tunnel is enabled |
| API key exposure | 🟡 Medium | Default (0.0.0.0 bind on LAN) |
| API key exposure | 🟢 Low | With `--host 127.0.0.1` + no tunnel |
| Prompt/conversation exposure | 🟡 Medium | Logged by MITM by design |
| Secondary package install | 🟡 Medium | postinstall hook downloads sqlite/systray |
| Machine fingerprinting | 🟢 Low | Local use only, no exfil found |
| Shared-system credential risk | 🟢 Low | Only on multi-user machines |

---

## Safe Usage Guide

If you decide to install and use 9router, follow these steps:

### 1. Create a restricted API key

Before installing, create a **new, restricted API key** with your AI provider:

- OpenAI: Platform → API Keys → Create new secret key → set a monthly spend limit (e.g. $5)
- Anthropic: Console → API Keys → New Key
- Never use your primary key or an org-level key

### 2. Install locally, not globally

```bash
# Prefer local install to contain the blast radius
mkdir ~/9router-sandbox && cd ~/9router-sandbox
npm install 9router

# Inspect what was installed before running
ls node_modules/9router/
ls ~/.9router/     # created by postinstall
```

### 3. Run bound to localhost only

```bash
./node_modules/.bin/9router --host 127.0.0.1
# Dashboard → http://127.0.0.1:20128/dashboard
```

### 4. Never enable tunneling

```bash
# DO NOT run:
9router --tunnel        # exposes your proxy to the internet
```

### 5. Use a firewall rule to block the port externally

```bash
# macOS — block inbound to port 20128
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add $(which node)
```

### 6. Rotate your key after use

If you used 9router for testing, revoke and regenerate the API key you configured in it.

---

## Alternatives

If the MITM proxy and trust model of 9router concern you, consider these audited alternatives:

| Tool | Type | Code | Notes |
|------|------|------|-------|
| [LiteLLM](https://github.com/BerriAI/litellm) | Python proxy | Open source, large team | Industry standard, widely audited |
| [LM Studio](https://lmstudio.ai) | Desktop app | Closed source | For local models only |
| [OpenRouter](https://openrouter.ai) | Hosted service | Hosted | No local install required |
| [omopen.ai free proxy](https://omopen.ai/free) | Hosted service | — | Free tier, OM-managed infrastructure |

---

## Disclaimer

This assessment reflects the state of `9router@0.5.81` as reviewed on 2026-09-18. It is provided for informational purposes only. The absence of malicious patterns in the reviewed code does not guarantee the tool is free of risks — code changes between versions, and not all server-side bundle files were exhaustively reviewed. Always exercise independent judgement.

This assessment is not affiliated with, endorsed by, or sponsored by the authors of 9router. The 9router name and any associated trademarks belong to their respective owners.

---

*Assessment by [OM Open AI](https://omopen.ai) · Licensed [CC BY 4.0](../../LICENSE)*
