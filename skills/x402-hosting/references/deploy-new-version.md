# Deploy a new version of an existing project

Use when the project directory **already has** `x402-hosting.json` and the user wants to publish updated code ("redeploy", "push my changes", "ship an update").

## Prerequisites

- `x402-hosting.json` present, with `expiresAt` set (the project is activated). If `expiresAt` is `null`, the first deploy never finished — see `references/deploy-new-project.md`.
- The **owner wallet** available to pay. It is `walletAddress` in `x402-hosting.json`; any other wallet is rejected before settlement.
- No pending `x402-hosting.payment.json`.

## Command

No `--days` — that is only for hosting duration, and a new deployment does not change the expiry.

```bash
npx x402-hosting deploy
```

## What happens

1. The CLI rebuilds the project locally with OpenNext.
2. It pays **$0.01** for a deployment upload session, and retains the exact built artifact alongside the payment so recovery never depends on a byte-identical rebuild.
3. It uploads the artifact and completes the deployment (free), which the platform publishes in the background.

A successful redeploy ends with an operation line reading `PAYMENT_SKIPPED` — despite the name that means **success** (the deployment step itself carries no charge; the $0.01 was the upload). Judge success by the exit code and `status`, never by that word, or you will re-run `deploy` and pay again.

The new deployment becomes the project's current deployment. The URL does not change, and **the expiry does not change** — a new deployment does not extend hosting. To extend, see `references/renew-hosting.md`.

## Verify

Both reads are free:

```bash
npx x402-hosting status
npx x402-hosting deployments --all
```

`deployments` lists each deployment with its number, id, and state, marking which is `current` and which is `pending`. Use a listed id with `rollback` if the new version is bad — see `references/manage-project.md`.

## Failure notes

- `wallet_not_owner` — the paying wallet is not `walletAddress` from `x402-hosting.json`. Pay from the owner wallet.
- `A pending x402-hosting.payment.json exists` — resolve it with `npx x402-hosting finalize` first; do not delete it blindly.
- `Project is <state>; cannot publish a new deployment.` — the project is expired, deleting, or failed. If expired, renew first.
- A build that fails locally costs nothing; the $0.01 upload is not refunded once paid. If the **paid session** succeeded but publishing was interrupted, run `npx x402-hosting finalize` — it reuses the artifact you already paid for instead of charging again.
