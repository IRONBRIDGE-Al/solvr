---
name: ironbridge-ibcp
description: IronBridge Compliance Protocol v2 — on-chain verification that an agent action passed all constitutional checks before execution. Anchored to Base L2.
category: governance
tier: free
solvr_api: https://wwwironbridge.org
auth: none
source: https://wwwironbridge.org
---

# ironbridge-ibcp

IronBridge Compliance Protocol (IBCP v2) — on-chain verification that an agent action was constitutionally executed. Every execution that passes IBCP gets a Base L2 anchor.

## What IBCP gives you

- On-chain proof of constitutional execution (Base L2, permanent)
- Compliance check against 345+ laws before any action commits
- Tamper-evident: IBCP record is anchored to the same LAW-25 row as the action
- ibCheck primary — IBCP is the on-chain source of truth for agent compliance

## Check compliance endpoint

```
GET https://wwwironbridge.org/api/ibcp/check?action_hash=<sha256>
```

## Response shape

```json
{
  "compliant": true,
  "action_hash": "sha256-of-action",
  "law_count": 1200,
  "chainlink_row": "sha256-linking-to-law25-ledger",
  "base_tx": "0x...",
  "ts_ms": 1747123456789
}
```

- `compliant` — true if all applicable laws passed
- `law_count` — number of laws active at time of execution
- `chainlink_row` — links to the LAW-25 sealed row
- `base_tx` — Base L2 transaction hash (permanent on-chain anchor)

## Verify the Base anchor

```javascript
const result = await fetch(
  'https://wwwironbridge.org/api/ibcp/check?action_hash=' + yourActionHash
).then(r => r.json());

if (!result.compliant) throw new Error('Action not constitutionally compliant');
console.log('Anchored on Base:', result.base_tx);
console.log('Sealed in LAW-25:', result.chainlink_row);
```

## What makes IBCP different from a log

1. **Checks before committing** — laws run against every action before execution
2. **Anchors on Base** — compliance record is a real on-chain transaction
3. **Hash-links to LAW-25** — Base tx and LAW-25 row share a chainlink hash; neither can be altered without breaking both
4. **No backdating** — Base block timestamps are immutable

## Compliance chain

```
Action proposed -> IBCP checks laws -> Pass -> Execute + LAW-25 row
-> LAW-25 row hash -> Base L2 anchor tx
-> ibcp/check returns { compliant: true, base_tx, chainlink_row }
```

## See also

- `ironbridge-law25-verifier` — verify the LAW-25 row that IBCP anchors to
- `ironbridge-x402-envelope` — x402 payment pipeline including IBCP compliance
- `ironbridge-passport-mint` — Passport required for full IBCP access
