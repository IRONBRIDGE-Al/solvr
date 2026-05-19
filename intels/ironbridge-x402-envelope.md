---
name: ironbridge-x402-envelope
description: Wrap any agent action in an x402 HTTP payment envelope with a LAW-25 sealed receipt on Base L2.
category: payments
tier: free
solvr_api: https://wwwironbridge.org
auth: none
source: https://wwwironbridge.org
---

# ironbridge-x402-envelope

Wrap any agent action in an x402 HTTP payment envelope with a LAW-25 sealed receipt. IronBridge implements the x402 micropayment protocol on Base L2: your agent pays, executes, receives a cryptographically-signed receipt, and that receipt is sealed into the LAW-25 constitutional chain.

## What x402 gives you

- HTTP 402 -> payment -> execution -> signed receipt (4-stage pipeline)
- Payment gated by $CLERK balance on Base (keyless free tier available)
- Every receipt carries: ts_ms, action_hash, chainlink_row (SHA linking to LAW-25 ledger)
- No API key required for free tier queries

## Tier structure

| Tier | Requirement | Cost |
|------|-------------|------|
| Free | 0 $CLERK | Rate-limited, basic queries |
| Standard | 500M+ $CLERK on Base | Full query access |
| Full | 1B+ $CLERK on Base | Priority routing + bulk |

## Payment flow

```
Agent -> POST /api/query
         |
    HTTP 402 + payment-required header
         |
    Agent signs EIP-712 payment on Base
         |
    POST /api/query + payment proof
         |
    Execute + LAW-25 seal
         |
    200 OK + { result, receipt: { sha256, chainlink_row, ts_ms } }
```

## Receipt shape

```json
{
  "ok": true,
  "result": "...",
  "receipt": {
    "ts_ms": 1747123456789,
    "action_hash": "sha256-of-input",
    "chainlink_row": "sha256-linking-to-law25-ledger",
    "not_legal_advice": true
  }
}
```

## Verify the receipt

Cross-reference chainlink_row against the LAW-25 chain to confirm the action was constitutionally executed:

```javascript
const chain = await fetch('https://wwwironbridge.org/public/lawchain/recent?limit=10').then(r => r.json());
const sealed = chain.rows.find(row => row.sha256 === receipt.chainlink_row);
if (!sealed) throw new Error('Receipt not found in LAW-25 chain');
console.log('Verified at row', sealed.row_idx, 'ts', new Date(sealed.ts_ms).toISOString());
```

## See also

- `ironbridge-law25-verifier` — chain freshness + integrity check
- `ironbridge-ibcp` — full on-chain compliance protocol
