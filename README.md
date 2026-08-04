# Moneo Agent Wallet & Trading for Cursor

Give Cursor's agent its own wallet. Hold and send USDG, trade tokenized
equities on Robinhood Chain, and have every spend checked against a policy
before anything is signed — without leaving the editor.

Built on [`@moneolabs/mcp`](https://www.npmjs.com/package/@moneolabs/mcp)
([GitHub](https://github.com/moneolabs/moneo) · [site](https://moneowallet.com)).

## What you get

| Component | What it does |
|---|---|
| **MCP server** (`mcp.json`) | Nine guarded tools: `get_balance`, `preflight_payment`, `pay`, `get_spending_limits`, `list_payments`, `get_quote`, `execute_order`, `list_positions`, `close_position`. |
| **`moneo` skill** | Teaches the agent the quote → confirm → execute flow, order types, and how to read a refusal. |
| **`moneo-status` skill** | Address, balance, limits, positions, and recent activity in one pass. |
| **`moneo-integrate` skill** | Teaches the agent to add a guarded wallet to *your* projects with the TypeScript SDK. |
| **Spend-safety rule** | Always on: spending tools need explicit approval, free reads preferred, refusals surfaced and never routed around. |

## First run

The server generates a key on first start and keeps it at `~/.moneo/key.json`.
The wallet address is printed to stderr — **fund it with USDG** to pay or
trade, and keep a little ETH on Robinhood Chain for gas. Ask the agent for its
wallet status and it will tell you the address.

The key never leaves that file. No tool returns it, so there is nothing for a
model to read, log, or paste anywhere.

## Limits that hold

Spending is capped *below* the model — checked before any signature, not inside
the model's turn:

- per-transaction cap
- rolling 24-hour cap
- velocity cap
- counterparty allowlist, so funds cannot go to an address you never approved
- escalation threshold above which a spend needs your approval

A misbehaving or prompt-injected model cannot exceed them. A blocked attempt
comes back as a result with the rule and the number it hit, and it lands on the
ledger with everything else. The bundled rule additionally requires the agent
to confirm every spend with you first.

Bring your own limits by pointing `MONEO_POLICY` at a policy JSON file in the
server's `env`.

## What can be traded

Tokenized equities settling against USDG on Robinhood Chain: **AAPL, TSLA,
NVDA, SPY**. Orders fill against Uniswap V3 pools; the router takes the fee
tier with the best output. Market, limit, TWAP, and bracket orders, with a
slippage bound that rejects rather than filling worse.

## Try it

- *"What's my wallet address and balance?"* — free, and tells you where to send funds
- *"What am I allowed to spend?"* — the caps and what's left of each
- *"Price $300 of AAPL"* — a real quote with fee and impact, nothing committed
- *"Buy $300 of AAPL and protect the position"* — a bracket order with exits attached
- *"Try to send $5,000 somewhere"* — watch the guard refuse it, and say why

## Configuration

| Variable | Default | What |
|---|---|---|
| `MONEO_AGENT` | `moneo-agent` | Identity recorded on every decision |
| `MONEO_ASSET` | `USDG` | Default asset for amounts that don't name one |
| `MONEO_POLICY` | cautious built-in | Path to a policy JSON file |
| `MONEO_KEY_PATH` | `~/.moneo/key.json` | Where the key lives |

## License

MIT © Moneo Labs
