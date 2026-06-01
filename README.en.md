<div align="center">

[![Telegram](https://img.shields.io/badge/Telegram_Channel-@avaco__cloud-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/avaco_cloud)

<h1>🦕 Avaco Deno</h1>

<p>A lightweight free relay for the XHTTP protocol on Deno Deploy</p>

[![Deno](https://img.shields.io/badge/Deno-≥2.0-000000?style=flat-square&logo=deno&logoColor=white)](https://deno.com)
[![Deno Deploy](https://img.shields.io/badge/Deploy-Deno_Deploy-70FFAF?style=flat-square&logo=deno&logoColor=black)](https://deno.com/deploy)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue?style=flat-square)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-brightgreen?style=flat-square)](#)
[![Free Tier](https://img.shields.io/badge/Free_Tier-10K_req%2Fmonth-orange?style=flat-square)](#free-tier-limits)

---

**🌐 زبان / Language**

**[🇮🇷 فارسی](README.md)** &nbsp;|&nbsp; **[🇬🇧 English](README.en.md)**

**[ [Introduction](#-introduction) • [Architecture](#-architecture) • [Deployment](#-deployment-steps) • [Multi-App](#-multi-app-deployment) • [Variables](#-environment-variables) • [Troubleshooting](#-troubleshooting) • [Management](#-management) ]**

</div>

---

## 📖 Introduction

A lightweight reverse proxy built on pure TypeScript/Deno (zero dependencies) that runs on **Deno Deploy** (free tier) and forwards XHTTP traffic from a public URL to your private backend server.

### ✨ Features

| Feature | Description |
|---------|-------------|
| 🦕 **Pure TypeScript** | No external dependencies — standard Deno only |
| 🔐 **Authentication** | Supports secret key via `x-relay-key` header |
| 🚦 **Traffic Control** | Concurrent request limit with `MAX_INFLIGHT` |
| ⏱️ **Timeout Management** | Auto-abort slow connections via `UPSTREAM_TIMEOUT_MS` |
| 🔍 **Built-in Debug** | `/__debug` endpoint for live status inspection |
| 🌍 **Multi-region** | Choose from multiple geographic locations |
| 📦 **Multi-app** | Deploy multiple apps for different backend servers |
| 💸 **Free** | 10,000 requests/month on the free tier |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Your Client                          │
│                  (v2rayN / Hiddify / ...)                   │
└──────────────────────────┬──────────────────────────────────┘
                           │  HTTPS
                           │  HOST: APP.ORG.deno.net
                           │  Path: /PUBLIC_RELAY_PATH
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                🦕 Deno Deploy (Edge)                        │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │           Avaco Deno Relay (TypeScript)             │   │
│   │                                                     │   │
│   │  ✓ Path validation  (PUBLIC_RELAY_PATH)             │   │
│   │  ✓ Authentication   (x-relay-key)                   │   │
│   │  ✓ Traffic control  (MAX_INFLIGHT)                  │   │
│   │  ✓ Header forward   (no IP/Host leak)               │   │
│   │  ✓ Bidirectional stream                             │   │
│   └──────────────────────────┬──────────────────────────┘   │
└──────────────────────────────┼──────────────────────────────┘
                               │  HTTP
                               │  Path: /RELAY_PATH
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              🖥️ Your Private Server (Xray-core)             │
│              TARGET_DOMAIN:PORT                             │
│              Protocol: VLESS + XHTTP                        │
└─────────────────────────────────────────────────────────────┘
```

### Request Flow

```
Client  ──►  Deno Deploy (PUBLIC_RELAY_PATH)  ──►  Server (RELAY_PATH)  ──►  Response
        ◄──                                   ◄──                        ◄──
```

---

## 🚀 Deployment Steps

### Prerequisite: Install Deno

```bash
curl -fsSL https://deno.land/install.sh | sh
export PATH="$HOME/.deno/bin:$PATH"

# Verify installation (must be 2.x)
deno --version
```

---

### Step 1 — Get an Access Token

1. Go to **[console.deno.com](https://console.deno.com)**
2. Create an **Org** (e.g. `myorg`)
3. Go to **Settings → Access Tokens → New Token**
4. Copy the token — it starts with **`ddo_`**

> ⚠️ Do NOT use `deployctl` — only `deno deploy` works on the new console.deno.com platform

---

### Step 2 — Clone and Enter Directory

```bash
git clone https://github.com/avacocloud/avaco-deno-relay.git
cd avaco-deno-relay/deno
```

---

### Step 3 — Create App and Deploy

```bash
deno deploy create \
  --token=ddo_YOUR_TOKEN \
  --org=YOUR_ORG \
  --app=YOUR_APP_NAME \
  --source=local \
  --entrypoint=main.ts \
  --region=eu \
  --no-wait \
  .
```

> ⚠️ **`--no-wait` is required** — without it you'll get a routing error

---

### Step 4 — Set Environment Variables

```bash
TOKEN=ddo_YOUR_TOKEN
ORG=YOUR_ORG
APP=YOUR_APP_NAME

deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN       "https://YOUR-SERVER:PORT"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP PUBLIC_RELAY_PATH   "/api"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH          "/api"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY           "your-secret-key-min-16-chars"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT        "512"
```

---

### Step 5 — Test and Verify

Your app URL:
```
https://YOUR_APP_NAME.YOUR_ORG.deno.net
```

```bash
curl https://YOUR_APP_NAME.YOUR_ORG.deno.net/__debug
```

Successful output:
```json
{
  "TARGET_BASE": "https://your-server:port",
  "PUBLIC_RELAY_PATH": "/api",
  "RELAY_PATH": "/api",
  "RELAY_KEY_SET": true,
  "UPSTREAM_TIMEOUT_MS": 0,
  "MAX_INFLIGHT": 512,
  "inFlight": 0
}
```

---

## 📦 Multi-App Deployment

One of Deno Deploy's advantages is running multiple apps in parallel, each pointing to a different backend server:

```bash
TOKEN=ddo_YOUR_TOKEN
ORG=YOUR_ORG

APPS=(relay1 relay2 relay3 relay4)
DOMAINS=(
  "https://server1.example.com:443"
  "https://server2.example.com:443"
  "https://server3.example.com:443"
  "https://server4.example.com:443"
)

# Create all apps
for i in 0 1 2 3; do
  deno deploy create \
    --token=$TOKEN --org=$ORG \
    --app=${APPS[$i]} \
    --source=local --entrypoint=main.ts \
    --region=eu --no-wait .
done

# Set env for all apps
for i in 0 1 2 3; do
  APP=${APPS[$i]}
  DOMAIN=${DOMAINS[$i]}
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN       "$DOMAIN"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP PUBLIC_RELAY_PATH   "/api"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH          "/api"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
  deno deploy env add --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT        "512"
done
```

> 💡 When you hit the free tier limit, create a new org with another account and redeploy.

---

## ⚙️ Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TARGET_DOMAIN` | ✅ | — | Full URL of your server. Example: `https://1.2.3.4:443` |
| `PUBLIC_RELAY_PATH` | ✅ | `/api` | Path the client connects to. Example: `/xhttp` |
| `RELAY_PATH` | ✅ | `/api` | Path configured in your Xray server |
| `RELAY_KEY` | ❌ | — | Secret key (min 16 chars) — recommended |
| `UPSTREAM_TIMEOUT_MS` | ❌ | `0` | Upstream timeout in milliseconds. `0` = no timeout |
| `MAX_INFLIGHT` | ❌ | `512` | Maximum concurrent requests |

### Important Notes

- **`TARGET_DOMAIN`** must not have a trailing `/` — `https://1.2.3.4:443` ✅ not `https://1.2.3.4:443/` ❌
- **`PUBLIC_RELAY_PATH`** and **`RELAY_PATH`** cannot be `/`
- If `RELAY_KEY` is set, the client must send the `x-relay-key` header
- **`RELAY_PATH`** can be anything but **must exactly match the path configured in your Xray server**

---

## 🌍 Region Selection

To see all available regions:

```bash
deno deploy create --help
```

Set the region in the `create` command:
```bash
--region=YOUR_REGION
```

Example:
```bash
deno deploy create \
  --token=ddo_YOUR_TOKEN \
  --org=YOUR_ORG \
  --app=YOUR_APP_NAME \
  --source=local \
  --entrypoint=main.ts \
  --region=eu \
  --no-wait \
  .
```

---

## 🔍 Troubleshooting

### Primary Debug Tool

```bash
curl https://YOUR_APP.YOUR_ORG.deno.net/__debug
```

This endpoint shows the full relay status — always check here first.

---

### Common Errors and Fixes

<details>
<summary>❌ <b>500 — Misconfigured: TARGET_DOMAIN is not set</b></summary>

**Cause:** `TARGET_DOMAIN` variable is not set.

```bash
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN "https://YOUR-SERVER:PORT"
```

</details>

<details>
<summary>❌ <b>500 — Misconfigured: RELAY_PATH cannot be '/'</b></summary>

**Cause:** Path is set to `/` which is not allowed.

```bash
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_PATH "/api"
```

</details>

<details>
<summary>❌ <b>500 — Misconfigured: RELAY_KEY is too short</b></summary>

**Cause:** Secret key is less than 16 characters.

```bash
# Generate a random key:
openssl rand -hex 16

# Then set it:
deno deploy env add --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY "your-generated-key"
```

</details>

<details>
<summary>❌ <b>403 — Forbidden</b></summary>

**Cause:** `x-relay-key` header is missing or incorrect.

Add this header in your client config:
```
x-relay-key: your-secret-key
```

</details>

<details>
<summary>❌ <b>404 — Not Found</b></summary>

**Cause:** Request path does not match `PUBLIC_RELAY_PATH`.

```bash
# Check what's currently set:
deno deploy env list --token=$TOKEN --org=$ORG --app=$APP
```

</details>

<details>
<summary>❌ <b>503 — Server Busy: Too Many Inflight Requests</b></summary>

**Cause:** Concurrent requests exceed `MAX_INFLIGHT`.

```bash
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP MAX_INFLIGHT "1024"
```

</details>

<details>
<summary>❌ <b>502 — Bad Gateway</b></summary>

**Cause:** Relay cannot reach `TARGET_DOMAIN`.

**Checklist:**
- [ ] Is `TARGET_DOMAIN` correct? (IP and port)
- [ ] Has the server firewall opened the port?
- [ ] Is Xray running on the server?
- [ ] Test direct access: `curl http://YOUR-SERVER:PORT`

</details>

<details>
<summary>❌ <b>504 — Gateway Timeout</b></summary>

**Cause:** Server did not respond within the configured timeout.

```bash
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP UPSTREAM_TIMEOUT_MS "0"
```

</details>

<details>
<summary>❌ <b>Deploy gives routing error</b></summary>

**Cause:** Missing `--no-wait` flag.

```bash
# --no-wait is mandatory:
deno deploy create ... --no-wait .
```

</details>

<details>
<summary>❌ <b>Hit the free tier limit</b></summary>

```bash
# Create a new org at console.deno.com
# Get a new token
# Redeploy with the new org
deno deploy create --token=NEW_TOKEN --org=NEW_ORG --app=APP_NAME ...
```

</details>

---

### Health Check Commands

```bash
# Full debug:
curl -v https://YOUR_APP.YOUR_ORG.deno.net/__debug

# Test with RELAY_KEY:
curl -H "x-relay-key: your-secret-key" https://YOUR_APP.YOUR_ORG.deno.net/api

# Simple ping:
curl -o /dev/null -s -w "%{http_code}\n" https://YOUR_APP.YOUR_ORG.deno.net/__debug
# Should return 200
```

---

## 🛠️ Management

```bash
# List all variables
deno deploy env list --token=$TOKEN --org=$ORG --app=$APP

# Update a variable
deno deploy env update-value --token=$TOKEN --org=$ORG --app=$APP TARGET_DOMAIN "https://NEW-SERVER:PORT"

# Delete a variable
deno deploy env delete --token=$TOKEN --org=$ORG --app=$APP RELAY_KEY

# Redeploy (after code changes)
deno deploy --token=$TOKEN --org=$ORG --app=$APP --prod --no-wait .

# Delete a single app
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  "https://api.deno.com/v2/apps/APP_NAME"

# Delete multiple apps
for app in relay1 relay2 relay3 relay4; do
  curl -s -X DELETE -H "Authorization: Bearer $TOKEN" \
    "https://api.deno.com/v2/apps/$app"
done
```

---

## 💰 Free Tier Limits

| Metric | Value |
|--------|-------|
| 📡 HTTP Requests | **10,000 / month** |
| ⚡ CPU Time | 0.2 hours / month |
| 🧠 Memory | 768 MB |
| 🌐 Outbound Traffic | 1 GiB / month |
| 😴 Sleep | None |

> 💡 When you hit the limit, create a new org with another account and redeploy.

---

## 📁 File Structure

```
deno/
├── main.ts        ← Main relay code (pure TypeScript)
├── deno.json      ← Deno configuration
└── README.md      ← Persian README
└── README.en.md   ← This file
```

---

## 💖 Support

If this project helped you and you'd like to support it, you can donate with cryptocurrency:

[![Donate with crypto](https://nowpayments.io/images/embeds/donation-button-white.svg)](https://nowpayments.io/donation?api_key=53edc3b4-8a65-451a-9ca9-67c30519c7a5)

---

## 🔗 Related Links

- 🦕 [Deno Deploy Console](https://console.deno.com)
- 🔑 [Deno Access Tokens](https://console.deno.com/account/access-tokens)
- 📦 [Deno Deploy Docs](https://docs.deno.com/deploy/manual)
- 🛠️ [XHTTP-Installer (auto server setup)](https://github.com/avacocloud/XHTTP-Installer)

---

## 📜 License

This project is licensed under **GNU GPL-3.0**.

Any redistribution, fork, or modification **must preserve**:

- ✅ Original copyright: `Copyright (C) 2025 avaco_cloud`
- ✅ Link to original repository: [github.com/avacocloud/avaco-deno-relay](https://github.com/avacocloud/avaco-deno-relay)
- ✅ Credit to original author: [@avaco_cloud](https://t.me/avaco_cloud)
- ✅ This LICENSE file unchanged

> ⚠️ Removing or replacing author attribution violates this license and constitutes copyright infringement, resulting in a **DMCA Takedown**.

For licensing questions or commercial use: [t.me/avaco_cloud](https://t.me/avaco_cloud)

---

<div align="center">

Made with ❤️ by [@avaco_cloud](https://github.com/avacocloud)

</div>
