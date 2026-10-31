# HIKING capped deployment reference

Project-provided BSC deployment: `0x628ccf2FE45b711e2149E2F408dbce1005660a6B`. Verify live state before execution. SKILL.md is self-contained for GitHub distribution.

## Rules

| Item | Value |
|---|---|
| Chain | BSC, ID 56 |
| ERC20 | Hiking / HIKING, 18 decimals |
| Personal cap | 10 cumulative packs per wallet |
| Global cap | 20,000 packs, not transaction count |
| Per pack | 50,000 HIKING to sender, 25,000 to LP reserve |
| Fees | 80% LP; 20% fixed treasury after launch |
| Close | 20,000 packs, or strictly >10,000 and >=1800 seconds since last successful mint |
| Clock | Chain timestamp; successful mint refreshes lastMintAt |
| <=10,000 | No inactivity expiry, deadline or refund method |
| Transfers | Locked until successful liquidity launch; unrestricted by Agent ID afterwards |
| Launch | Needs external finalize transaction; time passing alone sends no transaction |

Personal tier prices, USDT: **2, 2.25, 2.50, 2.75, 3, 3.20, 3.40, 3.60, 3.80, 4**. First ten total 30.50; no later pack is mintable after the wallet cap. Already minted 4 → next 2 cost 6.20. Already minted 10 → no quota remains. Always use on-chain quote.

Supply at closure = actual packs ×75,000; user allocation is ×50,000. Full user supply 1B, LP reserve 500M. No promised resale price. Registry balance proves only membership in that registry; provenance and transfer/upgrade rules require verification.

## Encoding

Load `Hiking.abi.json` from this directory into ethers v6 `Interface`. Encode mint with `[BigInt(quantity), quoteRaw]`; do not reconstruct raw quote from a displayed decimal number. Use the same ABI for custom errors, Minted and Transfer events.

Payment human-readable ABI:

```javascript
[
  'function decimals() view returns (uint8)',
  'function balanceOf(address) view returns (uint256)',
  'function allowance(address,address) view returns (uint256)',
  'function approve(address spender,uint256 amount) returns (bool)'
]
```

Registry ABI: `function balanceOf(address) view returns (uint256)`.

Quote checks the wallet cap; identity and remaining global capacity must also be checked. PriceAt uses a positive one-based ordinal; every ordinal >=10 returns 4 payment units. MAX_MINTS_PER_WALLET is 10 and exceeding it raises WalletMintLimit. PRICE_TIERS is a separate pricing parameter. Mint quantity is packs, maxCost is payment smallest units, and native value is zero. No private-key execution helpers are bundled.

## Failures

| Error | Action |
|---|---|
| InvalidQuantity | Require integer packs within remaining capacity |
| WalletMintLimit | Stop or ask for a quantity within remaining wallet quota |
| SupplyLimit | Refresh global quota; ask before changing explicit quantity |
| MintClosed | Stop; do not substitute a market purchase |
| NoAgentId | Check official status and authenticated registry for exact sender |
| PriceExceeded | Refresh quote and authorization; do not blindly raise maxCost |
| Insufficient allowance/balance | Check token, spender, exact units and receipt |
| UnsupportedPaymentToken | Stop and investigate payment/deployment behavior |
| TransfersLocked | Explain prelaunch transfer lock |
| Wallet risk/limit/mode failure | Follow official resolution, never bypass |
| Unknown/pending tx | Inspect existing transaction before resubmitting |

An off-chain preview failure spends no transaction gas; a mined revert normally does. Base the report on observed receipts.

## Test emergency option

Read emergencyWithdrawEnabled, emergencyOperator and cancelled. Option defaults false at deployment and is immutable. If enabled, the deploying address can withdraw payment tokens before launch using emergencyWithdrawPayment(amount). First withdrawal permanently cancels mint and liquidity launch; user tokens remain transfer-locked. It is not a buyer refund. The method cannot access LP or bypass its lock and is unavailable after launch. Disclose this authority before preparing a test-mode mint. Never invoke it under a mint request.
