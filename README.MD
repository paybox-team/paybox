<div align="center">
  <img src="./apps/web/public/assets/paybox-logo.svg" alt="PayBox" width="120" />

  <h1>PayBox</h1>
  <p>internet actions · Stellar · x402</p>

  <p>
    <img src="https://img.shields.io/github/actions/workflow/status/paybox-team/paybox/ci.yml?branch=main&label=CI&style=flat-square" />
    <img src="https://img.shields.io/badge/license-Apache%202.0-blue?style=flat-square" />
    <img src="https://img.shields.io/badge/node-%3E%3D20-brightgreen?style=flat-square" />
    <img src="https://img.shields.io/badge/network-Stellar%20testnet-blueviolet?style=flat-square" />
  </p>
</div>

<br />

```
  Agent picks provider
        │
        ▼
  POST /x402/search ──► HTTP 402 + payment details
        │                        │
        │               Agent settles on Stellar
        │                        │
        ◄────────────────────────┘
  Result + payment proof returned
        │
        ▼
  Usage logged · Spend tracked · Analytics updated
```

> PayBox turns internet actions into priced, payable intents.

---

## 📦 Monorepo

```
paybox/
│
├── apps/
│   ├── api/                    # Express API + x402 middleware
│   │   └── src/
│   │       ├── lib/
│   │       │   ├── x402.ts         ← payment middleware
│   │       │   ├── pricing.ts      ← provider catalog + SLA badges
│   │       │   ├── persistence.ts  ← storage layer
│   │       │   └── sponsorship/    ← challenge/grant flow
│   │       ├── routes/
│   │       │   ├── public.ts       ← /health, /api/providers, /api/analytics
│   │       │   ├── protected.ts    ← /x402/search, /x402/news, /x402/scrape
│   │       │   ├── demo.ts         ← /api/paid/run (sponsored)
│   │       │   └── sponsorship.ts  ← /api/sponsorship/*
│   │       └── services/
│   │           └── query-service.ts ← provider execution + fallback
│   │
│   ├── agent-client/           # Node CLI agent
│   │   └── src/
│   │       ├── cli.ts              ← search / news / scrape commands
│   │       ├── client.ts           ← x402 fetch + payment flow
│   │       ├── transcript.ts       ← demo transcript generator
│   │       └── validate-real.ts    ← real payment proof runner
│   │
│   └── web/                    # React dashboard
│       └── src/
│           ├── pages/
│           │   ├── LandingPage.tsx ← hero + live stats
│           │   └── ControlDeckPage.tsx ← provider cards + query workspace
│           └── lib/
│               ├── receipt.ts      ← export payment receipts
│               └── x402.ts         ← browser x402 client
│
└── packages/
    └── shared/                 # Shared Zod schemas + TypeScript types
        └── src/
            ├── schemas.ts          ← receipt, provider, grant schemas
            └── types.ts            ← QueryResult, UsageEvent, PaymentAttempt
```

---

## 🚀 Quickstart

```bash
git clone https://github.com/paybox-team/paybox.git
cd paybox
npm install
cp .env.example .env      # DEMO_MODE=true by default

# Terminal 1
npm run dev:api            # → http://localhost:3001

# Terminal 2
npm run dev:web            # → http://localhost:5173
```

No credentials needed in demo mode.

---

## 🌐 API Routes

```
paybox-api (port 3001)
│
├── PUBLIC
│   ├── GET  /health                    health + config snapshot
│   ├── GET  /api/providers             provider catalog
│   ├── GET  /api/catalog               catalog with pricing
│   ├── GET  /api/matrix                provider capability matrix
│   ├── GET  /api/usage                 paginated usage events
│   ├── GET  /api/analytics             spend summary + breakdown
│   ├── GET  /api/audit/digest          settlement digest
│   └── GET  /api/scenarios             demo scenario manifest
│
├── PAID  (x402 protected)
│   ├── GET  /x402/search               paid search query
│   ├── GET  /x402/news                 paid news query
│   └── GET  /x402/scrape               paid page scrape
│
├── SPONSORED
│   └── POST /api/paid/run              sponsored run with signed grant
│
└── SPONSORSHIP
    ├── POST /api/sponsorship/challenge  issue wallet challenge
    ├── POST /api/sponsorship/grants     exchange for signed grant
    └── POST /api/sponsorship/preview    preview sponsorship eligibility
```

---

## 🤖 Agent CLI

```bash
# Search
npm run cli -- search "latest soroban updates" --provider search.basic

# News
npm run cli -- news "stablecoin micropayments" --provider news.fast

# Scrape
npm run cli -- scrape "https://developers.stellar.org" --provider scrape.page

# Structured receipt output
npm run cli -- search "stellar x402" --provider search.pro --receipt

# Full demo flow
npm run demo:agent

# Real payment proof (requires funded testnet keys)
npm run validate:real
```

---

## 🏪 Provider Catalog

```
category   id                price     latency   quality
─────────────────────────────────────────────────────────
search     search.live       $0.05     1500ms    99
search     search.basic      $0.01      700ms    75
search     search.pro        $0.02     1100ms    90
news       news.fast         $0.015     800ms    72
news       news.deep         $0.03     1400ms    93
scrape     scrape.page       $0.02     1000ms    70
scrape     scrape.extract    $0.04     1700ms    95
```

All providers fall back to deterministic output when live keys are unavailable.

---

## ⚙️ Environment Variables

```bash
# Stellar + x402
STELLAR_NETWORK=stellar:testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
X402_FACILITATOR_URL=https://channels.openzeppelin.com/x402/testnet
X402_FACILITATOR_API_KEY=
X402_PAY_TO_ADDRESS=G...

# Agent wallets
DEMO_CLIENT_SECRET_KEY=S...
DEMO_CLIENT_PUBLIC_KEY=G...

# Server
PORT_API=3001
API_BASE_URL=http://localhost:3001
VITE_API_BASE_URL=http://localhost:3001
DEMO_MODE=true              # set false for real payments

# Optional AI providers
GROQ_API_KEY=
BRAVE_API_KEY=
NEWS_API_KEY=

# Sponsorship (off by default)
SPONSORSHIP_ENABLED=false
SPONSORSHIP_SIGNING_SECRET=
```

---

## 🧪 Testing

```bash
npm test                   # full suite
npm run test:coverage      # coverage + thresholds
npm run typecheck          # TypeScript across all workspaces
npm run lint               # ESLint
npm run format:check       # Prettier
```

---

## 💳 Testnet Wallet Setup

```
1. Generate keypair   →  https://laboratory.stellar.org
2. Fund with XLM      →  https://friendbot.stellar.org
3. Add USDC trustline →  via Stellar Laboratory
4. Fund USDC          →  https://faucet.circle.com
```

---

## 🏗 Stack

```
Language      TypeScript 5
API           Express 5
Payments      x402, @x402/stellar, @x402/express
Network       Stellar testnet (Soroban RPC)
Storage       SQLite (better-sqlite3), memory adapter
Frontend      React 18, Vite, Tailwind CSS
Schemas       Zod
Monorepo      npm workspaces
```

---

## License

Apache 2.0 — see [LICENSE](./LICENSE).
