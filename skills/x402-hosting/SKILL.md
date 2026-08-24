---
name: x402-hosting
description: "Deploy and manage Next.js hosting via the x402-hosting CLI — create a project, publish new deployments, roll back, renew hosting, transfer ownership, and delete. Hosting is prepaid with USDC over the x402 protocol and the paying wallet owns the project. Use whenever the user mentions deploying a Next.js app, shipping or publishing a site, getting a live URL, redeploying or pushing a new version, rolling back a deployment, renewing or extending hosting, hosting expiry, transferring or deleting a project, or x402-hosting.com."
---

# x402-hosting

Deploy a Next.js project to `x402-hosting.com` through the `x402-hosting` CLI. Hosting is prepaid with USDC over x402: **the wallet that pays owns the project**, and every later management action must be paid by that same wallet. This skill is a router: read the relevant reference file in `references/` for the task at hand.

## Preflight: Confirm the payment wallet

Every write costs money, so before any command that pays, confirm a wallet is available. `--payer auto` (the default) resolves **after** a 402 arrives, in this order:

1. `EVM_PRIVATE_KEY` in the environment → pays inline with that key.
2. The Coinbase agentic wallet, if installed and signed in → pays inline through it.
3. Otherwise → writes `x402-hosting.payment.json` for an external wallet, then you run `finalize`.

Before executing a paid command, tell the user the exact action and expected maximum USDC charge, then obtain explicit confirmation in the current conversation. Never print, log, or expose `EVM_PRIVATE_KEY`.

To confirm the agentic wallet is ready:

```bash
npx awal status --json
```

Ready means `server.running` is `true` **and** `auth.authenticated` is `true`. If it is not signed in, use the `agentic-wallet` skill to complete sign-in, or set `EVM_PRIVATE_KEY`. Reads (`status`, `deployments`) are free and need no wallet.

### Funding

The wallet needs enough **USDC** to cover the action (see Costs below) — a first deploy needs roughly `$0.01 + days × $0.20`. Check with:

```bash
npx awal balance --json
```

If the balance is short, use the `agentic-wallet` skill to fund the wallet (`npx awal address` for the deposit address, `npx awal show` to open the wallet window for onramp). Fund on the network the API charges on: the 402 challenge names it, and the handoff file records it under `paymentRequired.accepts[0].network`. Do not guess the network — read it from the challenge.

## Routing

Pick the reference that matches the task and `Read` it before acting:

| Task | Reference |
| --- | --- |
| First deploy of a project, "deploy this app", get a live URL, initial setup | `references/deploy-new-project.md` |
| Publish a new version, redeploy, push changes, ship an update to an existing project | `references/deploy-new-version.md` |
| Renew hosting, extend, "expires soon", expired project, add days | `references/renew-hosting.md` |
| Roll back to a previous deployment, undo a bad deploy | `references/manage-project.md` |
| Transfer ownership to another wallet, delete a project | `references/manage-project.md` |
| A payment tool errored, an interrupted command, a pending `x402-hosting.payment.json` | `references/payments-and-recovery.md` |

If the project directory has no `x402-hosting.json`, the task is a **first deploy**. If it has one, the task is a **new version** (or management).

## Shared rules

- **Never pay twice on an error.** If a payment tool reports failure, the payment may still have settled. Run `npx x402-hosting finalize` **before** retrying anything. See `references/payments-and-recovery.md`.
- **A pending `x402-hosting.payment.json` blocks new payments.** `deploy` refuses to start while one exists. Resolve it with `finalize`, never by deleting it blindly.
- **The owner wallet is fixed at creation.** It is the wallet that paid for the first upload. Pay every later action from that same wallet, or the server rejects it before settlement with `wallet_not_owner`.
- **`deploy` is never a no-op.** On an activated project every run builds and publishes a **new paid deployment ($0.01)**. Do not re-run it to "check" or "be safe" — use `status` (free) to inspect state.
- **Project name** defaults to the directory name. Pass `--project-name <name>` on the *first* deploy to set a different display name; it has no effect on later deployments.
- **Never invent `--api-url`.** Omit it; the CLI defaults to production. A project's `x402-hosting.json` pins its own `apiUrl`.
- **Input validation**: `--to` must match `^0x[0-9a-fA-F]{40}$`; `--days` must be an integer 1–730; deployment ids are UUIDs. Reject values containing spaces, semicolons, pipes, or backticks before placing them in a command.
- **JSON output**: `status` and `deployments` support `--json` for machine-readable output.
- **Activate within 30 minutes.** Between the two payments of a first deploy the project waits in `AWAITING_ACTIVATION`; past that deadline it is torn down and the $0.01 upload payment is lost. Do not leave a first deploy half-finished.
- **`deploy` modifies the project** on first run (adds pinned `@opennextjs/cloudflare`/`wrangler` devDependencies, an `x402:build` script, `open-next.config.ts`, `wrangler.jsonc`, and `.gitignore` entries). Mention this before deploying a repo you do not own.
- **Next.js version**: only `>=15.5.21 <16` or `>=16.2.11` is supported; anything else fails before any payment.
- **Report the URL.** After a successful deploy, read `url` from `x402-hosting.json` and give it to the user.

## Costs

| Action | Cost |
| --- | --- |
| Read status / list deployments | Free |
| First deploy | $0.01 upload + the duration price ($0.20 × days) |
| New deployment of an existing project | $0.01 |
| Renew | Duration price only ($0.20 × days) |
| Rollback / transfer / delete | $0.01 each |

A build that fails locally costs nothing (it runs before any payment), but **once the $0.01 upload is paid it is not refunded** — even if the deployment later fails server-side.

## Quick command index

| Command | Purpose |
| --- | --- |
| `npx x402-hosting deploy --days <n>` | First deploy: create the project and activate hosting |
| `npx x402-hosting deploy` | Publish a new deployment of the existing project |
| `npx x402-hosting status` | Read project state, wallet, and expiry (free) |
| `npx x402-hosting status --json` | Same, machine-readable (free) |
| `npx x402-hosting deployments --all` | List every deployment (free) |
| `npx x402-hosting renew --days <n>` | Extend hosting |
| `npx x402-hosting rollback <deployment-id>` | Promote a previous deployment |
| `npx x402-hosting transfer --to 0x…` | Hand ownership to another wallet |
| `npx x402-hosting delete` | Queue project deletion |
| `npx x402-hosting finalize` | Reconcile a pending payment without paying again |
