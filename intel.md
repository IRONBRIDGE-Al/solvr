---
name: Solvr Intelligence API
handle: @solvrbot
description: Permissionless intelligence for AI agents. 105 intel modules covering crypto, onchain, news, dev, social, and markets. One API. Free tier is keyless. Hold $SOLVR on Base to unlock standard and full tiers.
website: https://solvrbot.com
x: https://x.com/solvrbot
category: intelligence
version: 1.1
base_url: https://api.solvrbot.com
auth: Bearer <api_key>
---

# Solvr Intelligence API

Solvr is the **permissionless intelligence layer** for AI agents. Every autonomous agent that trades, deploys tokens, or makes financial decisions needs market data, security signals, news, and world context. Solvr provides all of it through a single API — no approval, no KYC, no human gatekeeper at any tier.

- **Free tier**: keyless. Just call. No wallet, no signup, no nothing.
- **Standard / Full tier**: permissionless but staked. Hold $SOLVR on-chain to unlock — code is the only gatekeeper.

## Quick Start (3 steps)

### Step 1 — Get a Base wallet

You need a wallet on the Base blockchain (chain ID 8453). Any of these work:
- **ClawBank** (https://clawbank.co) — purpose-built AI agent banking
- **Bankr** (https://bankr.bot) — AI-powered DeFi with built-in wallet
- Any Base-compatible EOA (MetaMask, Coinbase Wallet, etc.)

Your wallet address is used to sign registration and to verify your $SOLVR balance for tier access.

### Step 2 — Register your agent

Sign a message with your wallet and POST your `intel.md` to register:

```python
import time, requests
from eth_account import Account
from eth_account.messages import encode_defunct

wallet = "0xYOUR_WALLET_ADDRESS"
private_key = "0xYOUR_PRIVATE_KEY"
timestamp = int(time.time())

message = f"Register Solvr agent\nWallet: {wallet}\nTimestamp: {timestamp}"
sig = Account.sign_message(encode_defunct(text=message), private_key=private_key)

intel_md = """---
name: My Agent
handle: myagent
description: What my agent does
website: https://myagent.xyz
category: trading
---

My agent analyzes token launches and posts security alerts.
"""

resp = requests.post("https://api.solvrbot.com/api/v1/agent/register", json={
    "wallet": wallet,
    "sig": sig.signature.hex(),
    "timestamp": timestamp,
    "intel_md": intel_md,
})

data = resp.json()
api_key = data["api_key"]   # store this — shown once
handle  = data["handle"]    # your @handle on Solvr
```

Registration is free. The `api_key` is returned once — save it. Re-registering the same wallet issues a new key.

### Step 3 — Unlock higher tiers by holding $SOLVR

Solvr checks your registered wallet's $SOLVR balance on Base to determine your tier:

| Tier | $SOLVR held | Req/day | Req/hour | Endpoints |
|---|---|---|---|---|
| **Free** | 0 | 100 | 10 | News, world data, DEX search/trending |
| **Standard** | 20,000,000+ | 10,000 | 1,000 | + Token intel, security scan, TA, DEX analytics |
| **Full** | 1,000,000,000+ | Unlimited | 5,000 | + Full TA, future streaming/webhooks |

**How to get $SOLVR on Base:**
- Via Bankr: message `@bankrbot buy 20M SOLVR` (or use their app)
- Via ClawBank: fund wallet then swap on any Base DEX
- Via Uniswap/Aerodrome on Base directly
- Contract: `0x` — see https://solvrbot.com for current address

Once you hold the required amount in your registered wallet, your tier upgrades automatically on the next API call (cached for 10 minutes).

---

## Authentication

Include your API key on every request:

```
Authorization: Bearer solvr_xxxxxxxxxxxxxxxx
```

Requests without an API key are accepted at free tier, rate-limited by IP.

---

## Endpoints

### Free Tier

#### GET /api/v1/news
Real-time world news from BBC, Guardian, Al Jazeera, NPR, TechCrunch, Ars Technica.

```
GET /api/v1/news?topic=AI+regulation&category=tech&limit=5
```

Parameters:
- `topic` — keyword search (optional). If omitted, returns top headlines.
- `category` — `general` | `world` | `tech` | `business` | `science` | `health` | `politics`
- `limit` — 1–10 (default: 5)

Response:
```json
{
  "success": true,
  "category": "tech",
  "topic": "AI regulation",
  "count": 5,
  "articles": [
    {
      "title": "EU AI Act enforcement begins",
      "source": "BBC Technology",
      "summary": "The European Union's landmark AI Act...",
      "url": "https://bbc.co.uk/...",
      "published_at": "Mon, 05 May 2026 08:30:00 GMT"
    }
  ],
  "fetched_at": "2026-05-05T10:00:00Z"
}
```

---

#### GET /api/v1/worlddata
Economic and demographic data for 180+ countries (World Bank) + US macro indicators (FRED).

```
GET /api/v1/worlddata?query=population&country=Philippines
GET /api/v1/worlddata?query=fed+rate
GET /api/v1/worlddata?query=gdp&country=China
```

Parameters:
- `query` — what you want: `population`, `GDP`, `inflation rate`, `unemployment rate`, `life expectancy`, `co2 emissions`, `fed rate`, `10y treasury yield`, `vix`, `s&p 500`, `yield curve`, `jobless claims`, `literacy rate`, `poverty rate`, `renewable energy`, and more
- `country` — country name or ISO-2 code (e.g. `Philippines`, `US`, `China`, `EU`). Omit for global/world aggregate.

Response:
```json
{
  "success": true,
  "indicator": "Total Population",
  "value": 113880000,
  "formatted": "113.9M",
  "unit": "people",
  "country": "Philippines",
  "year": "2023",
  "change_pct": 1.4,
  "source": "World Bank Open Data",
  "fetched_at": "2026-05-05T10:00:00Z"
}
```

US macro indicators (requires FRED): `fed rate`, `10y treasury yield`, `2y treasury yield`, `yield curve`, `vix`, `s&p 500`, `m2`, `jobless claims`, `housing starts`, `dollar index`, `consumer sentiment`, `retail sales`, `industrial production`, `us gdp`, `breakeven inflation`.

---

#### GET /api/v1/dex/search
Search tokens by name, symbol, or contract address.

```
GET /api/v1/dex/search?q=SOLVR
```

---

#### GET /api/v1/dex/trending
Currently trending tokens across all chains.

```
GET /api/v1/dex/trending
```

---

### Standard Tier (20M+ $SOLVR)

#### GET /api/v1/intel/{ca}
Unified token intelligence — single call returns everything about a token.

```
GET /api/v1/intel/0x1234...abcd
```

Response includes: `price_usd`, `market_cap`, `fdv`, `liquidity_usd`, `volume_24h`, `price_change_24h`, `holder_count`, `security` (risk_score/verdict/flags), `social` (website/telegram/discord/x), `factory`, `launched_on`, `deployer_x`, `pair_url`, `indexed`.

---

#### POST /api/v1/security/scan
Multi-source token security scan with 0–100 risk score.

```python
requests.post("/api/v1/security/scan", json={"address": "0x...", "chain": "base"})
```

Response: `risk_score`, `verdict` (safe/caution/danger), `flags`, `goplus`, `honeypot`, `liquidity`, `deployer_history`.

---

#### POST /api/v1/security/bundles
Detect coordinated insider buying patterns.

```python
requests.post("/api/v1/security/bundles", json={"address": "0x...", "max_early_buyers": 20})
```

---

#### GET /api/v1/dex/token/{address}
Detailed token info by contract address.

---

#### GET /api/v1/dex/analytics
Multi-timeframe price, volume, and liquidity analytics.

```
GET /api/v1/dex/analytics?q=SOLVR
```

---

#### POST /api/v1/ta/quick
Quick technical analysis snapshot — RSI, MACD, Bollinger Bands.

```python
requests.post("/api/v1/ta/quick", json={"symbol": "BTC"})
```

---

### Full Tier (1B+ $SOLVR)

#### POST /api/v1/ta/analysis
Full technical analysis — RSI, MACD, Bollinger Bands, Stochastic, ADX, ATR, EMA (multi-timeframe).

```python
requests.post("/api/v1/ta/analysis", json={"symbol": "ETH", "interval": "4h"})
```

---

#### GET /api/v1/catalog
Returns all endpoints, tier definitions, and your current tier. Free for all.

```
GET /api/v1/catalog
Authorization: Bearer solvr_xxx
```

---

## Error Codes

| Code | Meaning |
|---|---|
| `403` | Endpoint requires higher tier — stake more $SOLVR |
| `429` | Rate limit exceeded for your tier |
| `402` | (Legacy) Payment required — upgrade to new tier model |
| `503` | Upstream data source temporarily unavailable |

---

## Integration Example (Python)

```python
import requests

BASE = "https://api.solvrbot.com"
HEADERS = {"Authorization": "Bearer solvr_your_key_here"}

def get_news(topic=None, category="general"):
    params = {"category": category, "limit": 5}
    if topic:
        params["topic"] = topic
    return requests.get(f"{BASE}/api/v1/news", params=params, headers=HEADERS).json()

def get_world_data(query, country=None):
    params = {"query": query}
    if country:
        params["country"] = country
    return requests.get(f"{BASE}/api/v1/worlddata", params=params, headers=HEADERS).json()

def scan_token(address):
    return requests.post(f"{BASE}/api/v1/security/scan",
        json={"address": address, "chain": "base"}, headers=HEADERS).json()

def token_intel(address):
    return requests.get(f"{BASE}/api/v1/intel/{address}", headers=HEADERS).json()

# Examples
news = get_news("Federal Reserve")
pop  = get_world_data("population", "Philippines")
fed  = get_world_data("fed rate")
scan = scan_token("0x1234...abcd")
intel = token_intel("0x1234...abcd")
```

---

## Notes for Agent Developers

- **Tier is checked on every request** against your wallet's live $SOLVR balance on Base (cached 10 min). No separate staking transaction is required — just hold $SOLVR.
- **Re-registration** with the same wallet issues a new API key and updates your profile. Old key is revoked.
- **Handle uniqueness**: If your requested handle is taken, a numeric suffix is appended automatically.
- **Timestamp window**: Registration signatures are valid for 10 minutes. Use Unix timestamp (seconds).
- **Rate limits** are per API key (or per IP for unauthenticated requests). Day window resets at UTC midnight.
- **Data freshness**: News is cached 5 min. World Bank/FRED data cached 1 hour. Token data near-real-time.

---

*Solvr — The Permissionless Intelligence Layer for the Agent Economy*
*solvrbot.com | @solvrbot | hello@solvrlabs.ai*
