---
name: fassets
description: FAssets mint and redeem on Flare. Mint FXRP by sending XRP to an agent, redeem FXRP back to XRP on XRPL. Full programmatic pipeline — no dApp needed. Triggers on "/fassets", "mint fxrp", "redeem fxrp", "fassets redeem", "fassets mint".
---

# FAssets Skill

Full mint and redeem pipeline for FAssets (FXRP) on Flare Network. FAssets are trustless, over-collateralized wrapped tokens backed by real XRP on the XRPL.

## Full Pipeline

```
MINT:  XRP (XRPL) ──→ Reserve collateral (Flare) ──→ Send XRP to agent (XRPL) ──→ FDC verifies ──→ FXRP minted (Flare)
REDEEM: FXRP (Flare) ──→ Burn FXRP (Flare) ──→ Agent sends XRP (XRPL) ──→ XRP received
```

## Prerequisites

```bash
npm install ethers xrpl
```

You need:
- **Flare wallet** (encrypted keystore) with FLR for gas + collateral reservation fee
- **XRPL wallet** (created with `create-xrpl-wallet.js`) with XRP for minting

## Commands

```bash
# Show parameters (lot size, fees)
node skills/fassets/scripts/fassets.js info

# List available minting agents
node skills/fassets/scripts/fassets.js agents

# Mint FXRP (full pipeline: reserve → send XRP → wait for mint)
node skills/fassets/scripts/fassets.js mint --lots 1

# Redeem FXRP → XRP
node skills/fassets/scripts/fassets.js redeem --lots 1 --xrpl-address rXXX...

# Check XRPL balance and recent transactions
node skills/fassets/scripts/fassets.js status --xrpl-address rXXX...

# Create XRPL wallet (prerequisite for minting)
node skills/wallet/scripts/create-xrpl-wallet.js --save ~/.secrets/xrpl-wallet.json
```

## Contracts

| Contract | Address | Purpose |
|----------|---------|---------|
| FXRP Token | `0xAd552A648C74D49E10027AB8a618A3ad4901c5bE` | ERC-20 FAsset token (6 decimals) |
| AssetManager | `0x2a3Fe068cD92178554cabcf7c95ADf49B4B0B6A8` | Mint/redeem controller |
| AssetManagerController | `0x097B93eEBe9b76f2611e1E7D9665a9d7Ff5280B3` | System controller |

## Key Parameters

| Parameter | Value |
|-----------|-------|
| Lot Size | 10 FXRP (= 10 XRP) |
| Decimals | 6 |
| Reservation Fee | ~0.15 FLR per lot |
| Minting Fee | ~0.1% (paid in XRP, added to lot) |
| Redemption Fee | ~0.2% (deducted from XRP received) |
| Minting Time | 5-20 minutes (FDC verification) |
| Redemption Time | Usually < 5 minutes |

## Minting Flow (Detailed)

1. **Reserve Collateral** — Call `reserveCollateral()` on AssetManager with FLR fee
   - Returns: agent's XRPL address, amount to send, payment reference, deadline
2. **Send XRP** — Send exact amount to agent's XRPL address with payment reference in memo
   - XRPL account needs 10 XRP reserve + minting amount + agent fee
3. **FDC Verification** — Flare Data Connector verifies the XRPL payment (automatic, 5-20 min)
4. **Minting Execution** — Agent/executor calls `executeMinting()` with FDC proof
   - FXRP is minted to the original minter's Flare address

## Redemption Flow

1. **Approve** FXRP spending to AssetManager (one-time)
2. **Call** `redeem(lots, xrplAddress, executor)` — burns FXRP
3. **Wait** — Agent sends XRP to your XRPL address (usually < 5 min)
4. **Receive** XRP minus redemption fee

## Important Notes

- All operations are in **lots** (1 lot = 10 FXRP = 10 XRP)
- XRPL addresses don't need to be pre-activated — incoming ≥10 XRP activates them
- The payment reference (memo) is **critical** — without it the agent can't match the payment
- The `executor` parameter can be `address(0)` — the agent's own executor typically handles it
- First reservation that goes unpaid will time out and release collateral (CRF is lost)

## Example: Full Round Trip

```bash
# 1. Create XRPL wallet
node skills/wallet/scripts/create-xrpl-wallet.js --save ~/.secrets/xrpl-wallet.json

# 2. Fund XRPL wallet with XRP (need ≥20 XRP: 10 reserve + 10 to mint)

# 3. Mint 1 lot (10 FXRP)
node skills/fassets/scripts/fassets.js mint --lots 1
# → Reserves collateral, sends XRP, waits for FXRP to arrive

# 4. Use FXRP in DeFi (swap, LP, etc.)

# 5. Redeem back to XRP
node skills/fassets/scripts/fassets.js redeem --lots 1 --xrpl-address rXXX...
# → Burns FXRP, agent sends XRP within minutes
```

## Dependencies

```bash
npm install ethers xrpl
```
