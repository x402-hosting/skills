# Payments and recovery

## The rule that prevents double charges

If a payment tool reports an error, **the payment may still have settled** — tools can time out or mangle a response after the money moved. Never retry a payment on the strength of a client-side error.

Always run this first, from the project directory:

```bash
npx x402-hosting finalize
```

`finalize` ignores the payment tool's output and asks the server what actually happened, reconciling by an operation id the CLI generated **before** paying. It then either completes the work you already paid for, or reports that nothing settled and clears the pending handoff so you can safely retry.

Do **not** delete `x402-hosting.payment.json` to get unstuck — that discards the only pointer to a possibly-settled payment.

## Payer modes

`--payer` (default `auto`) applies to every paid command:

| Mode | Behavior |
| --- | --- |
| `auto` | Env key if `EVM_PRIVATE_KEY` is set, else the agentic wallet if installed and signed in, else the external handoff |
| `env` | Sign inline with `EVM_PRIVATE_KEY`. The key is read only after a 402 arrives and is never persisted |
| `awal` | Pay inline through the Coinbase agentic wallet CLI |
| `external` | Always write `x402-hosting.payment.json` for another wallet to pay |

`--payer-cmd '<template>'` delegates to any wallet CLI. The template is split on whitespace and each placeholder becomes one argument — no shell is involved. `{url}` and `{headers}` are required (`{headers}` carries the idempotency key that reconciliation depends on); `{method}`, `{body}`, `{maxAmount}`, and `{operationId}` are optional.

```bash
npx x402-hosting renew --days 30 --payer-cmd 'mywallet pay {url} -X {method} -H {headers} --max {maxAmount}'
```

`--payer` and `--payer-cmd` are mutually exclusive.

## The external handoff flow

When the payer is `external` (or `auto` finds no wallet), the CLI writes `x402-hosting.payment.json` and exits successfully. That file contains the exact HTTP request to pay, the decoded payment requirements, and the operation id. Complete the request with any x402-capable wallet, then:

```bash
npx x402-hosting finalize
```

Every paid request carries its parameters in the **query string with an empty body**, so wallet tools that drop request bodies can pay them.

## Interpreting finalize

- **Recorded / completed** — the payment settled and the work finished. Nothing more to do.
- **Nothing settled** — no payment was recorded, nothing was charged, and the handoff is cleared. Safe to retry the original command.
- **Ended `FAILED` / `SETTLEMENT_FAILED`** — the handoff is cleared, but a settlement failure can still mean the charge went through. Check the wallet's receipts before paying again.
- **Still pending** — the operation has not reached a terminal state. Wait, then run `finalize` again. Do not pay again.

## Other notes

- Reads (`status`, `deployments`) are free and never involve a wallet.
- A build that fails locally costs nothing; a paid $0.01 upload is not refunded if the later deployment fails.
- The paid legs only create records — presigning, state changes — so they stay fast and do not risk timing out behind a slow build.
