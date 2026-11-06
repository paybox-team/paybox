<div align="center">
  <img src="./apps/web/public/assets/paybox-logo.svg" alt="PayBox logo" width="160" />
  <h1>PayBox</h1>
  <p><strong>Pay-per-query internet access for AI agents on Stellar via x402.</strong></p>
  <p>
    <a href="https://github.com/paybox-team/paybox/actions"><img src="https://img.shields.io/github/actions/workflow/status/paybox-team/paybox/ci.yml?branch=main&label=CI" alt="CI" /></a>
    <img src="https://img.shields.io/badge/license-Apache%202.0-blue" alt="License" />
    <img src="https://img.shields.io/badge/node-%3E%3D20-brightgreen" alt="Node" />
    <img src="https://img.shields.io/badge/network-Stellar%20testnet-blueviolet" alt="Stellar" />
  </p>
</div>

---

PayBox is a pay-per-query API gateway for AI agents. Powered by Stellar and the x402 payment protocol, it routes agent requests through a provider marketplace where every search, news lookup, and web scrape is individually priced and settled on-chain. No subscriptions. Just transparent, auditable micropayments per request.

## How it works

```
Agent → PayBox API → x402 challenge → Agent pays → Route unlocks → Result returned
                          ↓
                   Stellar testnet settlement
                          ↓
                   Usage + spend logged to SQLite
```

1. Agent picks a provider and sends a query
2. API responds with HTTP 402 + payment details
3. Agent settles via x402 on Stellar testnet
4. Route executes, result is returned with payment proof
5. Every query is logged with trace ID, price, and latency

## Features

- **Provider marketplace** — search, news, scrape with price/latency/quality comparison
- **x402 payment middleware** — HTTP 402 challenge/response on every paid route
- **Stellar testnet settlement** — real on-chain micropayments via `@x402/stellar`
- **Sponsorship flow** — challenge/grant system for sponsored agent sessions
- **Spend analytics** — per-category usage, payment history, settlement digest
- **Demo mode** — deterministic fallback for zero-credential demos
- **Agent CLI** — `search`, `news`, `scrape` commands with structured receipt output
- **React dashboard** — provider cards, paid query workspace, analytics panel

## Stack

| Layer | Tech |
|-------|------|
| API | Express 5, TypeScript, x402 |
| Payments | Stellar testnet, x402, OpenZeppelin facilitator |
| Storage | SQLite (better-sqlite3), memory adapter for tests |
| Frontend | React 18, Vite, Tailwind CSS |
| Agent client | Node CLI, tsx |
| Monorepo | npm workspaces |

## Quickstart

```bash
# 1. Clone and install
git clone https://github.com/paybox-team/paybox.git
cd paybox
npm install

# 2. Copy env and enable demo mode
cp .env.example .env

# 3. Start API (terminal 1)
npm run dev:api

# 4. Start web dashboard (terminal 2)
npm run dev:web
# → http://localhost:5173
```

No credentials needed for demo mode. The API runs deterministically with `DEMO_MODE=true`.

## API routes

| Method | Route | Auth | Description |
|--------|-------|------|-------------|
| GET | `/health` | — | Service health + config snapshot |
| GET | `/api/providers` | — | Provider catalog |
| GET | `/api/catalog` | — | Full catalog with pricing |
| GET | `/api/usage` | — | Paginated usage events |
| GET | `/api/analytics` | — | Spend summary + breakdown |
| GET | `/api/audit/digest` | — | Settlement digest |
| GET | `/x402/search` | x402 | Paid search query |
| GET | `/x402/news` | x402 | Paid news query |
| GET | `/x402/scrape` | x402 | Paid page scrape |
| POST | `/api/paid/run` | grant | Sponsored paid run |
| POST | `/api/sponsorship/challenge` | — | Issue challenge |
| POST | `/api/sponsorship/grants` | — | Exchange grant |

## Agent CLI

```bash
# Search
npm run cli -- search "latest soroban updates" --provider search.basic

# News
npm run cli -- news "stablecoin micropayments" --provider news.fast

# Scrape
npm run cli -- scrape "https://developers.stellar.org" --provider scrape.page

# Machine-readable receipt
npm run cli -- search "stellar x402" --provider search.pro --receipt

# Full demo showcase
npm run demo:agent
```

## Real payment validation

```bash
# 1. Set DEMO_MODE=false in .env and fill real funded keys
# 2. Start the API
npm run dev:api

# 3. Run the validator
npm run validate:real
```

Output includes `payment-response` header, trace ID, provider, and price — full on-chain proof.

## Environment variables

```bash
# Required for real payments
STELLAR_NETWORK=stellar:testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
X402_FACILITATOR_URL=https://channels.openzeppelin.com/x402/testnet
X402_FACILITATOR_API_KEY=
X402_PAY_TO_ADDRESS=G...
DEMO_CLIENT_SECRET_KEY=S...
DEMO_CLIENT_PUBLIC_KEY=G...

# Server
PORT_API=3001
API_BASE_URL=http://localhost:3001
VITE_API_BASE_URL=http://localhost:3001

# Demo mode (no credentials needed)
DEMO_MODE=true

# Optional AI providers
GROQ_API_KEY=
BRAVE_API_KEY=
NEWS_API_KEY=

# Sponsorship (disabled by default)
SPONSORSHIP_ENABLED=false
SPONSORSHIP_SIGNING_SECRET=
```

## Monorepo structure

```
paybox/
├── apps/
│   ├── api/              # Express API + x402 middleware
│   ├── agent-client/     # Node CLI agent
│   └── web/              # React dashboard
├── packages/
│   └── shared/           # Zod schemas + shared types
├── scripts/              # Dev utilities
└── .env.example
```

## Development

```bash
npm run build         # Build all workspaces
npm run typecheck     # TypeScript check across monorepo
npm test              # Full test suite
npm run test:coverage # Coverage with thresholds
npm run lint          # ESLint
npm run format        # Prettier
```

## Funding testnet wallets

1. Generate a Stellar keypair at [laboratory.stellar.org](https://laboratory.stellar.org)
2. Fund with [Friendbot](https://friendbot.stellar.org)
3. Add USDC testnet trustline
4. Fund USDC from [Circle testnet faucet](https://faucet.circle.com)

## License

Apache 2.0 — see [LICENSE](./LICENSE).
