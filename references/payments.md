# Payments

Commands for sending and receiving lightning payments, checking balances, and querying transaction history.

## Commands

```bash
# Lightning + on-chain balances
npx -y @getalby/hub-cli@0.5.0 get-balances

# Get an on-chain deposit address
npx -y @getalby/hub-cli@0.5.0 get-onchain-address

# Send an on-chain payment from the hub's on-chain wallet (confirm with human if channels are open and amount may cause anchor reserves may be drained)
npx -y @getalby/hub-cli@0.5.0 pay-onchain bc1... --amount 100000

# Sweep the entire on-chain balance to an address (confirm with human if any channels are open - anchor reserves will be drained)
npx -y @getalby/hub-cli@0.5.0 pay-onchain bc1... --all

# Pay a BOLT11 invoice
npx -y @getalby/hub-cli@0.5.0 pay-invoice lnbc...

# Pay a zero-amount invoice, specifying the amount in millisatoshis
npx -y @getalby/hub-cli@0.5.0 pay-invoice lnbc... --amount 1000

# Create a BOLT11 invoice
npx -y @getalby/hub-cli@0.5.0 make-invoice --amount 1000 --description "test"

# List recent payments
npx -y @getalby/hub-cli@0.5.0 list-transactions

# List with pagination
npx -y @getalby/hub-cli@0.5.0 list-transactions --limit 50 --offset 0

# Look up a specific payment by payment hash
npx -y @getalby/hub-cli@0.5.0 lookup-transaction <paymentHash>
```

## On-Chain Reserve Warning

After running `get-balances`, check the on-chain balance. If `lightning.totalSpendable > 0` and `onchain.spendable < 20000`, warn the user in a friendly, non-scary way:

> Your on-chain balance is low. While your funds are safe in your channel, having at least ~20,000 sats on-chain provides a safety reserve in case a channel ever needs to close on-chain unexpectedly. You can top up via `get-onchain-address`.

## Receiving Without a Channel

On **LDK**, if the hub has no channel yet, a `make-invoice` for an amount above the current receive capacity can trigger a **JIT channel** — an LSP opens a channel automatically when the payment arrives, with the fee deducted from that payment. See [JIT Channels](./jit-channels.md) for the LDK-only constraint, limits, and how to set expectations about the fee.

## Notes

- `--amount` for `pay-invoice` is in millisatoshis (msat). Use for zero-amount invoices only.
- `--amount` for `make-invoice` is in millisatoshis.
- `get-balances` returns both lightning (channel) and on-chain balances. This is the **hub wallet** balance — it is **not** the balance of any individual app.
- `pay-onchain` `--amount` is in **sats**, and spends from the **on-chain** wallet (not lightning). Use `--all` to sweep the whole on-chain balance instead of `--amount`. The address is **not validated for you** — read the full address back to the user and get explicit confirmation before sending; an on-chain payment is irreversible.
