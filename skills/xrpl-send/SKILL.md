# XRPL Send

Send XRP on the XRP Ledger. Check balance, send payments, view history.

## Commands

```bash
# Check wallet balance
node skills/xrpl-send/scripts/xrpl-send.js balance

# Send XRP
node skills/xrpl-send/scripts/xrpl-send.js send --to <address> --amount <XRP>

# Send with destination tag (for exchanges)
node skills/xrpl-send/scripts/xrpl-send.js send --to <address> --amount <XRP> --tag <number>

# Send with memo
node skills/xrpl-send/scripts/xrpl-send.js send --to <address> --amount <XRP> --memo "payment for X"

# View transaction history
node skills/xrpl-send/scripts/xrpl-send.js history --limit 10
```

## Natural Language

The agent can parse requests like:
- "send 5 XRP to rJ2Tzg..."
- "check XRP balance"
- "show XRPL history"

## Safety

- 1 XRP reserve always maintained (XRPL base reserve)
- Validates destination address format
- Checks destination account exists (warns if sending <1 XRP to unactivated account)
- Confirms balance before sending

## Wallet

- Address: `YOUR_XRPL_ADDRESS`
- Keyfile: `~/.openclaw/workspace/.secrets/xrpl-wallet.json`
