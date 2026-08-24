# Manage a project: rollback, transfer, delete

All three cost **$0.01** and must be paid by the owner wallet (`walletAddress` in `x402-hosting.json`). All require an existing `x402-hosting.json` in the directory.

## Roll back to a previous deployment

List deployments first (free) and take the id of the one to promote:

```bash
npx x402-hosting deployments --all
```

```
d2 <deployment-id> READY current
d1 <deployment-id> READY
```

Then:

```bash
npx x402-hosting rollback <deployment-id>
```

The chosen deployment becomes current and is republished. Rollback does **not** change the expiry.

## Transfer ownership

Hands the project to another wallet. After settlement the **current wallet loses all access immediately** — it can no longer deploy, renew, or delete.

```bash
npx x402-hosting transfer --to 0x0000000000000000000000000000000000000000
```

- `--to` must match `^0x[0-9a-fA-F]{40}$`. Validate before running, and confirm the address with the user — this is irreversible from your side; only the new owner can transfer it back.
- Interactive confirmation is required unless `--yes` is passed. Do not pass `--yes` on the user's behalf without explicit instruction.

## Delete a project

```bash
npx x402-hosting delete
```

Queues deletion: the Worker and stored artifact are torn down and the URL stops serving. This is **irreversible** — there is no undelete, and prepaid time is not refunded. Requires interactive confirmation unless `--yes` is passed; do not add `--yes` unless the user explicitly asked to skip the prompt.

## Failure notes

- `wallet_not_owner` — pay from the owner wallet.
- `project_operation_in_progress` — another operation is still settling; wait and check `npx x402-hosting status`.
- `A pending x402-hosting.payment.json exists` — run `npx x402-hosting finalize` first.
