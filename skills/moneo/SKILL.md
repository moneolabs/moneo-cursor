---
name: moneo
description: >-
  Give an AI agent its own guarded wallet on Robinhood Chain. Use when the
  agent needs to check a balance, send USDG, price or place a trade in
  tokenized equities, review positions, or explain why a spend was refused.
  Works via the bundled Moneo MCP server.
---

# Moneo: a guarded wallet for agents

Moneo hands an agent a wallet whose key it never sees and a policy that runs
before anything is signed. The agent can hold and send USDG and trade tokenized
equities on Robinhood Chain, and every attempt — allowed or refused — lands on
a ledger with a reason attached.

The mental model: **quote → confirm → execute**, with the guard in between. A
refusal comes back as a result the model can read, not an exception it should
retry.

## Setup

The plugin bundles the MCP server. On first run it generates a key and keeps it
at `~/.moneo/key.json`; the wallet address is printed to the server's stderr.
**Fund that address with USDG** to pay or trade, and keep a little ETH on
Robinhood Chain for gas. Until it is funded the wallet reads zero and spending
tools have nothing to spend.

Ask the agent for the wallet status and it will tell you the address.

## Tools

| Tool | Use it to |
|---|---|
| `get_balance` | Balance and what is not already committed. Free. |
| `get_spending_limits` | The limits in force and what is left of each budget. Free. |
| `preflight_payment` | Ask whether a payment would pass, without making it. Free. |
| `pay` | Send USDG to an address. Policy runs first. Irreversible. |
| `list_payments` | Payments made this session, with verdicts. Free. |
| `get_quote` | Price a trade across the pools without committing. Free. |
| `execute_order` | Quote and execute: market, limit, TWAP, or bracket. Irreversible. |
| `list_positions` | Open positions with cost basis and unrealized P&L. Free. |
| `close_position` | Exit with the same guarantees as entry. Irreversible. |

## What can be traded

Tokenized equities settle against USDG on Robinhood Chain: **AAPL, TSLA, NVDA,
SPY**. Orders fill against Uniswap V3 pools, and the router picks the fee tier
with the best output — liquidity is not spread evenly across tiers, so this
matters.

Quotes report price, the pool fee, and measured price impact. Show all three
before asking the user to confirm.

## Order types

- **market** — fill now at the quoted price, inside the slippage bound
- **limit** — rest at a price and give up after `expiresIn`
- **twap** — spread across a `window` in slices, to reduce impact
- **bracket** — the exit ships with the entry: `takeProfit` and `stopLoss`
  attached at submission, so an unattended position still has a way out

Prefer `bracket` for anything the user will not be watching.

## The safety model

Limits are enforced **below** the model, before any signature:

- a per-transaction cap
- a rolling 24-hour cap
- a velocity cap
- an allowlist for counterparties, so funds cannot go to an address the owner
  never approved
- an escalation threshold above which a spend needs approval

A prompt-injected or confused model cannot exceed them, because the check does
not run inside the model's turn. Read them with `get_spending_limits` and
report the numbers when the user asks what the agent is allowed to do.

To run your own limits, point `MONEO_POLICY` at a policy JSON file in the MCP
server's `env`.

## Reading a refusal

When something is blocked you get the rule, the limit, and the amount that hit
it — for example that a payment exceeds the per-transaction cap, that nothing
was signed, and that no fee was paid. Relay that to the user as-is. Do not
retry, split the amount, or reword the request; ask the owner to raise the
limit if the spend is genuinely wanted.

## Try it

- *"What's my wallet address and balance?"* — free, and tells you where to send funds
- *"What am I allowed to spend?"* — the caps at a glance
- *"Price $300 of AAPL"* — a real quote with fee and impact, nothing committed
- *"Buy $300 of AAPL and protect the position"* — a bracket order, exits attached
- *"What happened while I was away?"* — payments and positions with verdicts
