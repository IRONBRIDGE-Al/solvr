---
name: ironbridge-passport-mint
description: Mint an IronBridge Passport NFT on Base L2 using EIP-6963 multi-wallet detection. Unlocks operator access and LAW-25 sealed execution.
category: identity
tier: free
solvr_api: https://wwwironbridge.org
auth: none
source: https://wwwironbridge.org
---

# ironbridge-passport-mint

Mint an IronBridge Passport NFT on Base L2. The Passport is the on-chain identity layer for every IronBridge agent — gating full access to the platform, the x402 payment pipeline, and on-chain operator status.

## What the Passport gives you

- On-chain identity tied to a Base wallet address
- Unlocks passport-gated API routes and LAW-25 sealed execution
- Required for Operator Partner tier and IBCP compliance checks
- ERC-721 on Base mainnet — transferable, verifiable, chain-permanent

## Live mint endpoint

```
POST https://wwwironbridge.org/api/mint-passport
```

No API key required. Caller provides a connected Base wallet via EIP-6963.

## EIP-6963 wallet detection

IronBridge uses EIP-6963 for wallet detection — eliminates MetaMask window injection conflicts. Any EIP-6963 compliant wallet works (MetaMask, Coinbase Wallet, Rabby, etc.).

```javascript
const providers = [];
window.addEventListener('eip6963:announceProvider', (event) => {
  providers.push(event.detail);
});
window.dispatchEvent(new Event('eip6963:requestProvider'));

const provider = providers[0].provider;
const accounts = await provider.request({ method: 'eth_requestAccounts' });
const address = accounts[0];
```

## Calling the mint

```javascript
const res = await fetch('https://wwwironbridge.org/api/mint-passport', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ address })
});
const { tx } = await res.json();

const txHash = await provider.request({
  method: 'eth_sendTransaction',
  params: [tx]
});
```

## Verify passport status

```javascript
const check = await fetch(
  'https://wwwironbridge.org/api/passport-status?address=' + address
);
const { hasPassport, tokenId, mintedAt } = await check.json();
```

## Tier access after mint

| State | Access |
|-------|--------|
| No passport | Free tier queries only |
| Passport held | Full query access, IBCP compliance checks |
| Passport + 500M $CLERK | Standard x402 payment tier |
| Passport + 1B $CLERK | Full x402 priority routing |

## LAW-25 sealed mint

Every successful mint produces a sealed LAW-25 receipt — cryptographic proof of when and to whom each Passport was issued.

## See also

- `ironbridge-law25-verifier` — verify the mint seal
- `ironbridge-x402-envelope` — payment pipeline unlocked by Passport
- `ironbridge-ibcp` — on-chain compliance requiring Passport
