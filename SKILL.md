---
name: mint-hiking
description: Mint HIKING on BNB Smart Chain using Binance Agentic Wallet. Use for mint HIKING, 打铭文, checking HIKING eligibility, remaining wallet quota, mint prices, USDT approval, or troubleshooting HIKING mint failures.
---

# HIKING Mint

Respond in the user's language. This file is self-contained. Installing or reading it does not authorize transactions. Quote-only requests must remain read-only.

## Deployment

- Network: BNB Smart Chain, chain ID 56.
- HIKING: `0x628ccf2FE45b711e2149E2F408dbce1005660a6B`.
- Payment token: BSC USDT `0x55d398326f99059fF775485246999027B3197955` (18 decimals).
- Expected Agent ID registry: `0x60B9262456dE011F7d15a54CdD8c48bAaa8B022F`.
- Each pack gives the sender 50,000 HIKING and mints 25,000 HIKING to the contract's liquidity reserve.
- Each wallet may mint at most 10 packs in total, including batches. Global cap: 20,000 packs.
- Personal prices in USDT: 2, 2.25, 2.50, 2.75, 3, 3.20, 3.40, 3.60, 3.80, 4. First ten total 30.50 USDT, plus BNB gas. An eleventh pack is forbidden.

This is the project-provided deployment, not a claim of audit. Check RPC chain ID, contract code and live parameters before writes. Read paymentToken() and agentIdRegistry(); stop on an unexpected dependency or rule mismatch rather than silently switching addresses. The registry address is the contract's configured dependency, not independent proof of Binance endorsement. Never use the former HIKING deployment.

## 1. Connect the official wallet

Use the official Binance Agentic Wallet skill:
https://github.com/binance/binance-skills-hub/tree/main/skills/binance-web3/binance-agentic-wallet

Install through the host's authorized installation workflow if absent. Follow its current authentication and external-sign instructions; never request private keys, seed phrases or session files.

```bash
baw wallet status --json
baw auth signin --json
baw auth verify --qrCodeId <returned-qrCodeId> --json
baw wallet address --json
baw wallet settings --json
```

Only sign in if disconnected. Display the exact returned pairingCode and urlForWeb; do not reconstruct the URL or share its login token. Keep auth verify running until completion or expiry. Select the address whose binanceChainId is 56. Never use a pasted address as an authenticated sender.

For mint execution, check Developer Mode before quoting/building calldata. If disabled or expired, ask the user to enable it in Binance App → Agentic Wallet → Settings, then recheck. Never change wallet controls automatically. Read-only public queries need no login or Developer Mode.

## 2. Validate and quote

Read related state at the same block where possible:
- MAX_MINTS(), MAX_MINTS_PER_WALLET(), USER_TOKENS(), RESERVE_TOKENS().
- isMintOpen(), cancelled(), launched(), totalMints(), mintedBy(sender).
- paymentToken(), agentIdRegistry(), paymentUnit(), emergencyWithdrawEnabled().
- Registry balanceOf(sender), USDT decimals(), balanceOf(sender), allowance(sender,HIKING), and sender's BNB balance.

Expected constants are 20000, 10, 50000 * 10^18 and 25000 * 10^18. Require positive registry balance for execution. Check official wallet Agent ID status when available; investigate conflicting results. Registry membership does not prove AI execution or one-person-one-wallet.

Stop when mint is closed or cancelled. If emergencyWithdrawEnabled is true, disclose before approval that prelaunch withdrawal can cancel the sale and leave user tokens locked; do not call any administrator method.

Use integer/BigInt arithmetic:
- walletRemaining = max(0, 10 - mintedBy(sender)).
- globalRemaining = max(0, 20000 - totalMints()).
- available = min(walletRemaining, globalRemaining).

Resolve quantity as an integer number of packs. If unspecified, ask how many. For an explicit request to mint all remaining wallet quota, use available and show its cost before execution. Never interpret it as all remaining global supply or split transactions to bypass the wallet cap. Do not silently reduce a specific requested quantity.

Require 1 <= quantity <= available. Call quote(sender,quantity) for the authoritative raw cost. Use that exact value as maxCost. Do not derive raw amounts from floating-point display values or assume USDT uses 6 decimals.

Show sender, chain, contract, packs, HIKING received, exact USDT cost, wallet quota remaining after mint, approval requirement and estimated gas when available. Check wallet limits and balances. Explain that user tokens cannot transfer until liquidity launch; there is no guaranteed launch date or refund method.

## 3. Approve and mint

Use an ABI encoder such as ethers v6 Interface. Relevant ABI signatures:

```text
function quote(address wallet,uint256 quantity) view returns (uint256)
function mintedBy(address wallet) view returns (uint256)
function MAX_MINTS_PER_WALLET() view returns (uint256)
function mint(uint256 quantity,uint256 maxCost)
function approve(address spender,uint256 amount) returns (bool)
function allowance(address owner,address spender) view returns (uint256)
function balanceOf(address account) view returns (uint256)
event Minted(address indexed wallet,uint256 quantity,uint256 cost,uint256 walletTotal,uint256 globalTotal)
event Transfer(address indexed from,address indexed to,uint256 value)
```

Skip approval if allowance covers cost. Otherwise encode USDT approve(HIKING,quoteRaw), never an unlimited allowance. If the token requires resetting an existing insufficient allowance, confirm approve(HIKING,0) before approving the exact amount. Native value is zero.

For EACH approval and mint, use the official two-step signing flow:

```bash
baw contract-call preview --binanceChainId 56 --from <sender> --to <target> --value 0 --inputData <encoded-calldata> --json
baw contract-call execute --requestId <successful-preview-requestId> --json
```

Between preview and execute, show the decoded operation, parsedTx, risks, authority changes and any App confirmation requirement. Obtain the explicit confirmation required by the official wallet workflow. Never execute a failed or blocked preview, or invent a requestId. A successful preview is not a submitted transaction.

Wait for approval confirmation, then refresh sender, eligibility, open state, quota, quote, balance and allowance. Approval does not reserve mint quota. If the cost or requested operation changes, obtain confirmation for the revised transaction.

Encode HIKING mint(quantity,quoteRaw). Target HIKING, not USDT, the registry or the liquidity pair. Send native value 0. Never transfer USDT directly to the contract as a substitute for minting.

Do not invoke finalize, withdrawals, LP release, swaps or bridges as part of a mint request.

## 4. Verify and report

If execution requires App confirmation, tell the user to confirm there and track the existing operation. For a timeout or ambiguous submission, inspect the App, existing transaction, nonce and on-chain state before considering a retry. Never blindly resend.

A transaction hash is not success. Wait for a successful BSC receipt and verify the exact HIKING Minted event matches sender, quantity and cost. Verify the sender's mint Transfer from the zero address equals quantity * 50000 * 10^18; the separate reserve allocation is not user income.

Report packs, HIKING received, USDT paid, remaining wallet/global quota and https://bscscan.com/tx/<hash>. Use receipt events for transaction-specific totals. If pending, say pending. If approval succeeded but mint failed, distinguish the two.

## Closing rules and errors

Mint has no calendar deadline. At <=10000 total packs, inactivity does not close mint. Strictly above 10000, >=1800 seconds since lastMintAt closes mint logically. Use chain time and isMintOpen(), not closed() alone. Each successful mint resets lastMintAt. Sold out at 20000. Liquidity launch needs a separate external transaction; it is not executed automatically by elapsed time.

- WalletMintLimit: refresh wallet quota; do not bypass or switch wallets automatically.
- MintClosed / SaleCancelled: stop.
- NoAgentId: verify the connected BSC address and required registry membership.
- PriceExceeded: refresh quote and user confirmation.
- Insufficient balance/allowance: check units, spender and successful approval receipt.
- Expired preview: refresh state and preview again before confirmation.
- Wallet policy rejection: follow official resolution; never bypass security controls.

Official signing reference:
https://github.com/binance/binance-skills-hub/blob/main/skills/binance-web3/binance-agentic-wallet/references/external-sign.md
