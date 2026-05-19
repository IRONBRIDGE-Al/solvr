---
name: ironbridge-law25-verifier
description: Verify any IronBridge action was sealed into the LAW-25 constitutional hash chain. Checks freshness, integrity, and total sealed rows using the public chain endpoint.
category: governance
tier: free
solvr_api: https://wwwironbridge.org
auth: none
source: https://wwwironbridge.org
---

# ironbridge-law25-verifier

Verify that an IronBridge action was sealed into the LAW-25 constitutional chain. Every execution on IronBridge produces a tamper-evident row with SHA-256 linkage to the previous row. This intel lets your agent confirm provenance: the action happened, when it happened, and that the chain has not been broken.

## Endpoint

```
GET https://wwwironbridge.org/public/lawchain/recent?limit=N
```

No API key required. Free tier. Returns the N most recent sealed rows.

## Response shape

```json
{
  "total": 1200,
  "growing": true,
  "rows": [
    {
      "row_idx": 1200,
      "sha256": "abc123...",
      "prev_hash": "def456...",
      "ts_ms": 1747123456789,
      "skill": "law25-keepalive",
      "soldier": "S-12"
    }
  ]
}
```

## Verify chain integrity

```javascript
async function verifyLaw25(limit = 5) {
  const res = await fetch(
    'https://wwwironbridge.org/public/lawchain/recent?limit=' + limit
  );
  const data = await res.json();

  if (!data.growing) throw new Error('LAW-25 chain not growing — stale');

  const ageMin = (Date.now() - data.rows[0].ts_ms) / 60000;
  if (ageMin > 25) throw new Error('Chain stale: ' + ageMin.toFixed(1) + ' min ago');

  for (let i = 0; i < data.rows.length - 1; i++) {
    if (data.rows[i].prev_hash !== data.rows[i + 1].sha256) {
      throw new Error('Hash chain broken at row ' + data.rows[i].row_idx);
    }
  }

  return { ok: true, total: data.total, age_min: ageMin, tip_sha: data.rows[0].sha256 };
}
```

## What makes this different

Standard agent platforms log to a database you cannot inspect. IronBridge seals every execution into a hash-chained ledger with cryptographic linkage. Your agent can independently verify:

- **Freshness** — chain grew within the last 25 minutes
- **Integrity** — SHA-256 chain is unbroken
- **Volume** — total sealed rows (1,200+)

The chain is anchored to Base L2 for final on-chain proof.

## See also

- `ironbridge-ibcp` — on-chain compliance verification
- `ironbridge-x402-envelope` — payment + LAW-25 sealed receipt
