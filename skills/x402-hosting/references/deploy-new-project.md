# Deploy a new project

Use when the project directory has **no** `x402-hosting.json`. This creates the project, binds the owner wallet, and activates hosting.

## Prerequisites

- A Next.js project in the current directory (a `package.json` with a `next` dependency). The CLI supports Next.js only.
- A funded wallet — see the preflight in `SKILL.md`. The first deploy costs **$0.01 + the duration price**.
- No pending `x402-hosting.payment.json` in the directory.

## Command

`--days` is required on a first deploy and sets how long hosting is prepaid for (1–730).

```bash
npx x402-hosting deploy --days 30
```

## What happens

Two paid steps with free work between them, so no single payment waits on a slow build:

1. **Paid upload ($0.01)** — the CLI builds the project locally with OpenNext, then pays for an upload session. **The wallet that pays this becomes the project owner.**
2. **Upload + create** (free) — the artifact is uploaded and the project record is created, inheriting that owner. The platform publishes and smoke-tests the app in the background.
3. **Paid activation** (duration price) — once the build is awaiting activation, the CLI pays to activate. Only the owner wallet from step 1 can pay this.

Expected output, ending with the live URL:

```
Preparing and building with OpenNext…
Packaging build artifact…
Paying for the initial upload session…
Uploading artifact…
Paying to activate the project…
ACTIVATE operation <uuid>: SETTLED
{
  "operationId": "…",
  "projectId": "…",
  "state": "SETTLED",
  "walletAddress": "0x…",
  "expiresAt": "2026-09-24T13:18:13.688Z"
}
```

The final `SETTLED` operation is the success signal. The live URL is not printed on that last line — read it from `x402-hosting.json` (below) or run `status`.

## After a successful deploy

The CLI writes `x402-hosting.json` to the project root:

```json
{
  "apiUrl": "https://api.x402-hosting.com",
  "projectId": "…",
  "walletAddress": "0x…",
  "url": "https://app-….x402-hosting.com",
  "expiresAt": "2026-09-24T13:18:13.688Z",
  "currentDeploymentId": "…",
  "pendingDeploymentId": null
}
```

Read `url` from it and give that to the user. Commit this file — it is public metadata only and is how later commands find the project. It holds no secret.

## Worked example

```
$ npx awal status --json          # preflight: wallet ready?
{"server":{"running":true},"auth":{"authenticated":true,"email":"…"}}

$ npx awal balance --json         # enough USDC for $0.01 + 30 days?
{"balances":{"USDC":{"formatted":"21"}}}

$ npx x402-hosting deploy --days 30
Preparing and building with OpenNext…
Packaging build artifact…
Paying for the initial upload session…     # paid leg 1 — $0.01, binds the owner
Uploading artifact…
Paying to activate the project…            # paid leg 2 — duration price
ACTIVATE operation 28013e27-…: SETTLED
{ "operationId": "…", "projectId": "…", "state": "SETTLED", "expiresAt": "2026-09-24T13:18:13.688Z" }

$ npx x402-hosting status                  # free — this is where the URL is easiest to read
https://app-ad4a4a60….x402-hosting.com — ACTIVE
Wallet: 0xe126…652a
Expires: 2026-09-24T13:18:13.688Z
```

Then give the user the URL.

## Confirm it is really live

The deploy is done when `status` reports `ACTIVE`. To confirm the site itself is serving:

```bash
curl -s -o /dev/null -w "%{http_code}\n" "$(node -e "console.log(require('./x402-hosting.json').url)")"
```

A `200` means it is live. Publishing propagates at the edge, so a **fresh URL can 404 for a short time** even though the deploy succeeded — if `status` says `ACTIVE`, wait a few seconds and retry the request rather than redeploying. Never re-run `deploy` to "fix" a transient 404; that publishes another paid deployment.

## If the command stops early

- **A handoff was written** (`x402-hosting.payment.json` exists and the CLI printed instructions): an external wallet must pay the request in that file, then run `npx x402-hosting finalize`. See `references/payments-and-recovery.md`.
- **It stopped after the project was created but before activation** (`x402-hosting.json` exists with `"expiresAt": null`): just re-run `npx x402-hosting deploy --days <n>`. It detects the unactivated project and resumes at activation without rebuilding. `--days` is required again on that run (omitting it errors with `--days is required to activate this project`).

## Failure notes

- `--days is required for the first project` — pass `--days`.
- `No supported Next.js project found` — the directory is not a Next.js project.
- A build that fails locally costs nothing. But the $0.01 upload payment settles before the server-side deployment work, and is **not refunded** if that work then fails.
