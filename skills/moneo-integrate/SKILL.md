---
name: moneo-integrate
description: >-
  Add a guarded wallet to the user's own agent project with the Moneo
  TypeScript SDK. Use when they want their agent to hold funds, pay, or trade
  in their own code rather than through the bundled MCP server.
---

# Add a guarded wallet to your own agent

The MCP server is one way to hold the SDK. When the user wants this inside
their own project, reach for the packages directly.

```bash
npm install @moneolabs/wallet @moneolabs/guard @moneolabs/trading
```

## The shape

A policy, a wallet that runs it before every signature, and a trading layer
that shares it.

```ts
import { createGuard } from "@moneolabs/guard";
import { createWallet, evmSigner, robinhoodRail, ROBINHOOD_TOKENS } from "@moneolabs/wallet";
import { createTrading, robinhoodVenue, robinhoodPrices } from "@moneolabs/trading";
import { cachedPrices } from "@moneolabs/core";

const signer = evmSigner();            // key persists at ~/.moneo/key.json
const prices = cachedPrices(robinhoodPrices({ tokens: ROBINHOOD_TOKENS }));

const guard = createGuard({
  perTransaction: { max: "$250" },
  rolling24h: { max: "$1,000" },
  velocity: { max: 10, per: "1h" },
  counterparties: "allowlist-only",
  allow: ["venue:*"],
  escalate: { above: "$100" },
}, { agent: "overnight-01", prices });

const wallet = await createWallet({
  agent: "overnight-01",
  signer,
  rail: robinhoodRail({ signer }),
  asset: "USDG",
  guard,
});

const trading = createTrading({
  agent: "overnight-01",
  guard,
  quoteAsset: "USDG",
  venues: [robinhoodVenue({ signer, tokens: ROBINHOOD_TOKENS })],
});
```

## The loop

Every `execute` passes the guard first. A refusal is a value, so the loop keeps
running instead of throwing.

```ts
const quote = await trading.quote({ sell: "USDG", buy: "AAPL", notional: "300 USDG" });

await trading.execute(quote, {
  type: "bracket",        // the exit ships with the entry
  takeProfit: 1.04,
  stopLoss: 0.98,
  maxSlippage: 0.003,     // reject rather than fill worse
});
```

## The morning after

```ts
trading.positions();

for (const entry of await guard.history({ since: "8h" })) {
  console.log(entry.verdict, entry.action, entry.reason);
}
```

## Points worth making to the user

- **The signer never returns key material.** Nothing on its public shape
  exposes the key, so there is no field a model can read or paste elsewhere.
- **The guard runs below the model.** Limits are checked before a signature, so
  a prompt-injected model cannot exceed them by being persuasive.
- **Slippage bounds reject, they do not degrade.** An order that cannot fill
  inside the bound comes back rejected rather than filled worse.
- **Refusals carry reasons.** Surface them; they are the audit trail that makes
  an unattended run reviewable.
- **Swap the rail and the venue, keep everything else.** They are interfaces —
  point them at a different chain or venue and the code above is unchanged.
