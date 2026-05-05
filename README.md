# ZerionGuard — Autonomous DCA Agent on Solana

> **Solana Frontier Hackathon submission**

An autonomous Dollar-Cost Averaging (DCA) agent that executes **real onchain swaps** via Jupiter, controlled through Telegram, and protected by a layered policy engine.

---

## 🎥 Demo Video

> https://www.loom.com/share/f91a44a5b3ef4f219a86979de958d20f

---

## ✨ What it does

ZerionGuard lets you set up a DCA strategy through a Telegram bot. Once created, the agent autonomously buys your chosen token on a schedule — with every execution gated through a 5-layer policy engine that keeps the agent safe and bounded.

```
You: /new
Bot: Which token? → SOL
     Amount/buy?  → $5 USDC
     How often?   → every 30 min
     Spend cap?   → $50

[30min later] ⚡ DCA Executed! Bought 0.029 SOL | Tx: 4csxYx...
[60min later] ⚡ DCA Executed! Bought 0.029 SOL | Tx: 9abcDe...
```

---

## 🏗 Architecture

```
Telegram Bot (src/bot.mjs)
      │
      ▼
  Scheduler (setInterval per strategy)
      │
      ▼
  Executor
      │
      ├── Policy Engine (5 layers)
      │       ├── Spend limit (per strategy)
      │       ├── Token allowlist
      │       ├── Daily cap (global)
      │       ├── Expiration check
      │       └── Slippage guard
      │
      └── Jupiter API v6  ←  real onchain swaps on Solana
```

---

## 🔒 Policy Engine

Five policies run before **every** trade. If any policy denies, the trade is blocked.

| Policy | Scope | What it enforces |
|--------|-------|-----------------|
| `spend-limit` | Per-strategy | Blocks once cumulative spend hits the cap |
| `token-allowlist` | Global | Only SOL, USDC, USDT, JUP, BONK, WIF, PYTH |
| `daily-cap` | Global | 24h ceiling across all active strategies |
| `expiration` | Per-strategy | Auto-cancels once cap is reached |
| `slippage-guard` | Per-trade | Max 0.5% slippage via Jupiter |

---

## 🤖 Telegram Commands

| Command | Description |
|---------|-------------|
| `/start` | Main menu with inline buttons |
| `/new` | Launch strategy creation wizard |
| `/list` | View all strategies + pause/resume/cancel |
| `/portfolio` | Agent wallet balance (SOL + USDC) |
| `/history` | Last 5 executions with tx hashes |
| `/pause <id>` | Pause strategy |
| `/resume <id>` | Resume strategy |
| `/cancel <id>` | Cancel strategy |
| `/help` | Full command reference |

---

## 🚀 Quick Start

### Prerequisites

- Node.js ≥ 20
- Telegram bot token from [@BotFather](https://t.me/BotFather)
- Solana wallet with USDC (mainnet) or SOL (devnet)

### 1 — Clone & install

```bash
git clone https://github.com/setunda3-tech/zerionguard
cd zerionguard
npm install
```

### 2 — Configure

```bash
cp .env.example .env
```

Edit `.env`:

```env
TELEGRAM_BOT_TOKEN=your_token_here
SOLANA_PRIVATE_KEY=your_base58_private_key
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
```

### 3 — Run

```bash
node src/bot.mjs
```

Open Telegram → find your bot → `/start`

### Docker

```bash
docker-compose up -d
```

---

## ⚙️ Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `TELEGRAM_BOT_TOKEN` | ✅ | — | From @BotFather |
| `SOLANA_PRIVATE_KEY` | ✅ | — | Base58 encoded private key |
| `SOLANA_RPC_URL` | — | mainnet-beta | Solana RPC endpoint |
| `GLOBAL_DAILY_SPEND_CAP_USD` | — | `1000` | 24h global cap |
| `MAX_STRATEGIES_PER_USER` | — | `5` | Per-user strategy limit |
| `MIN_INTERVAL_MINUTES` | — | `30` | Minimum DCA interval |

---

## 🔌 How it connects to Solana

1. All swaps use **Jupiter Aggregator v6** for best price routing
2. Transactions are signed locally with the agent keypair
3. Confirmed on-chain via `connection.confirmTransaction`
4. Every tx hash is sent to Telegram with a Solscan explorer link

---

## 🧪 Testing on Devnet

```bash
# Set in .env:
SOLANA_RPC_URL=https://api.devnet.solana.com

# Run swap test:
node scripts/test-swap-devnet.mjs

# Run bot:
node src/bot.mjs
```

On devnet, swaps are simulated with real Jupiter price quotes but no real funds are used.

---

## 📁 Project Structure

```
zerionguard/
├── src/
│   └── bot.mjs              # Telegram bot + scheduler + executor
├── scripts/
│   ├── test-swap-devnet.mjs # Devnet swap test
│   └── generate-solana-wallet.mjs
├── wallet.js                # Solana wallet helper
├── jupiter-executor.js      # Jupiter swap executor
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## 📝 License

MIT
