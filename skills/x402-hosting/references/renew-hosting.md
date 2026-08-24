# Renew hosting

Use to extend prepaid hosting before (or shortly after) it expires. Renewal costs the **duration price only** — $0.20 × days, with no $0.01 management surcharge.

## Check the expiry first

Free:

```bash
npx x402-hosting status
```

```
https://app-<slug>.x402-hosting.com — ACTIVE
Wallet: 0x…
Expires: 2026-09-24T13:18:13.688Z
```

`expiresAt` is also in `x402-hosting.json`. The states that matter:

- `ACTIVE` — renewing extends **from the current expiry**, so no time is lost by renewing early.
- `EXPIRED` — the site is down but still renewable during the grace window; renewing republishes it and extends **from now**.
- Anything else (`DELETED`, `DELETION_PENDING`, `FAILED`) — not renewable.

## Command

```bash
npx x402-hosting renew --days 30
```

`--days` is required (1–730). The owner wallet must pay; it is `walletAddress` in `x402-hosting.json`.

By default the renewal targets the project's current (or pending) deployment. To pin a specific one, pass an id from `npx x402-hosting deployments`:

```bash
npx x402-hosting renew --days 90 --deployment <deployment-id>
```

## After renewing

The CLI prints the settled operation and updates `expiresAt` in `x402-hosting.json`. Confirm with:

```bash
npx x402-hosting status
```

## Grace window and expiry

After `expiresAt` passes, the site stops serving and the project enters a grace period during which renewal still works. Once the grace window closes the stored artifact is deleted and the project can no longer be renewed — starting over requires **deleting `x402-hosting.json` first**, then `deploy --days <n>`, which creates a new project with a **new URL**. Running `deploy` with the stale file still present charges $0.01 for an upload that can never be published. Renew before then to keep the URL.

## Failure notes

- `No current or pending deployment is recorded for renewal` — pass `--deployment <id>` explicitly, taking an id from `npx x402-hosting deployments`.
- `project_not_renewable` — the project is outside its renewal window or already deleted.
- `wallet_not_owner` — pay from the owner wallet in `x402-hosting.json`.
- If a payment tool errors, run `npx x402-hosting finalize` **before** retrying — see `references/payments-and-recovery.md`.
