# x402-hosting skills

Official agent skills for deploying and managing applications on [x402-hosting.com](https://x402-hosting.com).

## Available skills

### x402-hosting

Deploy and manage Next.js hosting through the `x402-hosting` CLI. The skill covers first deployments, publishing new versions, renewals, rollbacks, ownership transfers, deletion, and payment recovery.

Hosting is prepaid with USDC over the x402 protocol. The wallet that pays for the first deployment owns the project and must authorize later management operations.

## Install

Install interactively:

```bash
npx skills add x402-hosting/skills --skill x402-hosting
```

Install globally for a specific agent:

```bash
npx skills add x402-hosting/skills --skill x402-hosting -g -a codex -y
```

The repository follows the open [Agent Skills specification](https://agentskills.io/specification) and is compatible with the [Vercel Skills CLI](https://github.com/vercel-labs/skills).

## Structure

```text
skills/
└── x402-hosting/
    ├── SKILL.md
    └── references/
```

## Safety

Paid hosting operations spend USDC. The skill requires the agent to state the expected maximum charge and obtain explicit confirmation before executing a paid command. Read-only status and deployment-listing commands are free.
