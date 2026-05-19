---
name: ironbridge-skill-runtime
description: SRP-1.0 skill execution protocol — HMAC-sealed skill loading with drift detection and a LAW-25 sealed receipt per run.
category: runtime
tier: free
solvr_api: https://wwwironbridge.org
auth: none
source: https://wwwironbridge.org
---

# ironbridge-skill-runtime

IronBridge Skill Runtime Protocol (SRP-1.0) — the execution contract for every skill running on the platform. SRP defines how skills are loaded, verified, and sealed. Any agent exposing a skill via IronBridge gets HMAC integrity checking, LAW-25 sealed execution, and drift detection automatically.

## What SRP-1.0 gives you

- HMAC-SHA256 integrity check on every skill before execution
- LAW-25 sealed receipt per skill run — cryptographic proof of execution
- Drift detection: if a skill changes unexpectedly, SRP flags it before the next run
- Cron-scheduled execution with the same receipt guarantee as manual calls

## Skill structure

Every IronBridge-compatible skill is three files:

```
skills/<skill-name>/
  index.js       — the skill logic
  manifest.json  — metadata + sha256 of index.js at registration
  cron.txt       — schedule (optional, e.g. "*/15 * * * *")
```

## manifest.json shape

```json
{
  "name": "skill-name",
  "version": "1.0.0",
  "description": "What this skill does",
  "sha256_index_js": "sha256-of-index-js-at-registration-time",
  "law25_required": true,
  "tier": "free"
}
```

## Execution flow

```
Skill invoked (cron or manual)
-> SRP reads manifest.json sha256_index_js
-> SRP computes live sha256 of index.js
-> Match: execute / Mismatch: halt + drift alert
-> index.js runs
-> LAW-25 row written (if law25_required: true)
-> Receipt: { result, sha256_index_js, chainlink_row, ts_ms }
```

## Reading a skill receipt

```javascript
const receipt = await fetch('https://wwwironbridge.org/api/skill/run', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ skill: 'your-skill-name', params: {} })
}).then(r => r.json());

// receipt.chainlink_row links to the LAW-25 sealed row
```

## Drift detection

If index.js is modified after registration without updating manifest.json, SRP detects the mismatch before execution:

```json
{
  "ok": false,
  "error": "skill-drift",
  "expected_sha256": "abc123...",
  "actual_sha256": "def456...",
  "skill": "your-skill-name"
}
```

No compromised or modified skill can run silently on IronBridge.

## See also

- `ironbridge-law25-verifier` — verify the LAW-25 row produced per skill run
- `ironbridge-x402-envelope` — x402 payment envelope wrapping skill execution
- `ironbridge-ibcp` — IBCP compliance check applied during skill execution
