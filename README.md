# 🌿 BREEZO Network

<div align="center">

## Hyperlocal Air Quality DePIN on Solana

**BREEZO** is a decentralized physical infrastructure network (DePIN) for real-world air quality monitoring powered by Solana.

Physical ESP32 sensor nodes collect environmental data such as PM2.5, PM10, AQI, temperature, and humidity. Operators earn **BREEZO tokens** for contributing verifiable environmental data, while developers and users access live hyperlocal insights through dashboards, APIs, and chat interfaces.

---

**Built for cleaner cities, open environmental data, and decentralized infrastructure.**

</div>

---

# 📋 Table of Contents

* [Overview](#-overview)
* [Problem Statement](#-problem-statement)
* [Key Features](#-key-features)
* [Architecture](#-architecture)
* [System Flow](#-system-flow)
* [Backend — NodeService](#-backend--nodeservice)
* [Smart Contract](#-smart-contract)
* [Telegram & Premium Payments](#-telegram--premium-payments)
* [Reward Engine](#-reward-engine)
* [Security](#-security)
* [Project Structure](#-project-structure)
* [Getting Started](#-getting-started)
* [Environment Variables](#-environment-variables)
* [API Overview](#-api-overview)
* [Tech Stack](#-tech-stack)
* [Future Vision](#-future-vision)
* [License](#-license)

---

# 🌍 Overview

South Asian cities face severe air pollution, yet most existing AQI systems only provide city-wide averages rather than street-level environmental visibility.

BREEZO solves this by combining:

* 🌿 Real ESP32 air quality sensors
* ⚡ Node.js + MongoDB backend
* ⛓️ Solana smart contracts with Anchor
* 💰 Tokenized reward incentives
* 🤖 Telegram & WhatsApp premium AI access
* 📊 Real-time dashboards and APIs

The result is a fully decentralized environmental data economy where physical infrastructure earns on-chain rewards.

---

# 🚨 Problem Statement

Existing environmental monitoring systems are often:

* centralized and opaque
* expensive to deploy
* lacking real-time neighborhood visibility
* difficult to integrate into open developer ecosystems
* missing incentives for physical sensor deployment

Citizens need hyperlocal environmental data.

Sensor operators need incentives.

Developers need open programmable infrastructure.

BREEZO provides all three.

---

# ✨ Key Features

## 🌿 Hyperlocal Air Quality Monitoring

ESP32 devices publish:

* PM2.5
* PM10
* AQI
* Temperature
* Humidity

with real-time ingestion and visualization.

---

## ⛓️ Solana-Powered DePIN Rewards

Sensor operators earn **BREEZO tokens** for contributing useful environmental data.

Rewards are:

* calculated automatically
* synced on-chain
* claimable directly to user wallets

---

## 🔐 Secure Device Verification

Every device submission is:

* signed using NaCl cryptographic signatures
* replay-protected using timestamps
* validated before storage or rewards

Only authentic hardware can submit valid data.

---

## 📡 Real-Time Dashboard

The React frontend provides:

* live AQI updates
* network map visualization
* node dashboards
* reward balances
* claim interfaces
* historical metrics

---

## 🤖 Telegram & WhatsApp Premium AI

Users can:

* query nearby AQI
* ask AI-powered environmental questions
* unlock premium requests via Solana Pay

Payments are verified automatically using transaction memos and Helius indexing.

---

# 🏗 Architecture

```text
┌──────────────────────────────────────────────────────────────────────┐
│                           BREEZO Network                            │
├──────────────┬────────────────────────┬──────────────────────────────┤
│   Hardware   │        Backend         │          Blockchain          │
│              │                        │                              │
│  ESP32       │  Node.js + Express     │  Solana + Anchor Program    │
│  Sensors     │  MongoDB + Socket.IO   │  SPL Token                  │
│              │                        │                              │
│ PM2.5  ──────┼─► ingestData()         │                              │
│ PM10         │  verifySignature() ────┼─► Node PDA                  │
│ Temp         │  calculateReward()     │   rewardBalance             │
│ Humidity     │  syncToSolana() ───────┼─► addReward()               │
│ AQI          │                        │   claimReward()             │
│              │                        │                              │
│              │ REST API               │ Treasury PDA                │
│              │ Dashboard Updates      │ SPL Transfers               │
│              │ Telegram Bot           │                              │
└──────────────┴────────────────────────┴──────────────────────────────┘
```

---

# ⚙️ System Flow

## 1️⃣ Node Registration

Admin registers an ESP32 sensor:

```text
createNode()
```

This creates:

* MongoDB node record
* empty latest-state document
* optional on-chain PDA

---

## 2️⃣ Sensor Ingestion

ESP32 devices send signed payloads:

```json
{
  "nodeId": "abc123",
  "pm25": 12.4,
  "pm10": 18.7,
  "temp": 28.5,
  "humidity": 64,
  "aqi": 90,
  "timestamp": 1714298320,
  "signature": "base64encoded..."
}
```

Backend processing:

```text
ingestData()
 ├─ Verify timestamp
 ├─ Verify NaCl signature
 ├─ Store latest reading
 ├─ Calculate reward
 └─ Sync to Solana if threshold reached
```

---

## 3️⃣ Reward Synchronization

Once reward accumulation reaches threshold:

```text
syncToSolana()
  └─ addReward(amount)
```

Rewards are credited to the node PDA on Solana.

---

## 4️⃣ User Claim Flow

Users claim rewards from the frontend:

```text
claimReward()
 ├─ Verify wallet ownership
 ├─ Check reward balance
 └─ SPL transfer: Treasury → User ATA
```

---

## 5️⃣ Premium Chat Access

Telegram / WhatsApp flow:

```text
User exceeds free quota
        │
        ▼
Generate Solana Pay QR
        │
        ▼
User pays BREEZO tokens
        │
        ▼
Poller verifies payment via Helius
        │
        ▼
Premium requests unlocked
```

---

# 🧠 Backend — NodeService

The `NodeService` is the core orchestration layer of BREEZO.

It handles:

* sensor ingestion
* signature verification
* reward calculation
* Solana synchronization
* claim handling
* dashboard aggregation

---

## `ingestData()`

Receives sensor payloads from ESP32 devices.

### Responsibilities

* validates node ownership
* prevents replay attacks
* verifies NaCl signatures
* updates live sensor state
* calculates rewards
* triggers async Solana sync

---

## `createNodeOnChain()`

Creates the Solana PDA for a device.

### PDA Seeds

```text
["node", ownerPubkey, devicePubkey]
```

Stores:

* owner wallet
* device public key
* reward balance

---

## `syncToSolanaAsync()`

Non-blocking wrapper around Solana reward sync.

### Guarantees

* ingestion never blocks
* syncing flag always resets
* backend never deadlocks on failed syncs

---

## `syncToSolana()`

Performs actual on-chain reward writes.

### Security Checks

* verifies on-chain ownership
* prevents tampering
* credits reward balance securely

---

## `claimReward()`

Transfers BREEZO tokens from treasury to user wallet.

### Flow

```text
Treasury ATA → User ATA
```

after successful verification.

---

## `getUserDashboard()`

Returns:

* all user nodes
* latest readings
* AQI metrics
* reward balances
* claim states

Used by the React dashboard.

---

# ⛓️ Smart Contract

The Anchor program powers BREEZO’s on-chain reward system.

## Program ID

```text
2CZ1WzjHhgbBFaRaTxhLpdKQKAEsYDFga6bbuQRHfCJu
```

---

## Instructions

| Instruction        | Description                  |
| ------------------ | ---------------------------- |
| `initNode`         | Creates node PDA             |
| `addReward`        | Credits reward balance       |
| `claimReward`      | Transfers SPL tokens to user |
| `buyProduct`       | Premium/API purchases        |
| `withdrawTreasury` | Admin treasury withdrawal    |

---

## NodeAccount Structure

```rust
NodeAccount {
  owner: Pubkey,
  devicePublicKey: Pubkey,
  rewardBalance: u64,
  bump: u8
}
```

---

## Treasury PDA

```text
Seed: ["treasury"]
```

Stores BREEZO token liquidity for claims.

---

## Key Addresses

| Name         | Address                                        |
| ------------ | ---------------------------------------------- |
| Program ID   | `2CZ1WzjHhgbBFaRaTxhLpdKQKAEsYDFga6bbuQRHfCJu` |
| BREEZO Mint  | `soQUnxjoEMCMxBroyS4AvrtVn2JCtPZnR3N53NA5AvU`  |
| Admin Wallet | `4faW5GHsCXwGgQAMmAL7sSENpaezCb63cncWvzGc8iJa` |

---

# 🤖 Telegram & Premium Payments

BREEZO includes a Telegram/WhatsApp payment system for premium AI usage.

---

## Features

* location-aware AQI responses
* AI environmental assistant
* Solana Pay QR generation
* automatic payment verification
* premium usage unlocks

---

## Payment Details

| Property     | Value                     |
| ------------ | ------------------------- |
| Token        | BREEZO                    |
| Cost         | 10 BREEZO                 |
| Unlocks      | 100 premium requests      |
| Expiry       | 15 minutes                |
| Verification | Helius + transaction memo |

---

## Verification Flow

Background poller:

* fetches treasury transactions
* verifies memo
* validates token mint + amount
* upgrades user automatically

---

# 💰 Reward Engine

Rewards incentivize useful air quality data.

```js
function calculateReward(pm25) {
  if (pm25 < 50) return 0.02
  if (pm25 < 100) return 0.01
  if (pm25 < 300) return 0.005
  return 0
}
```

| PM2.5 Range | Air Quality | Reward         |
| ----------- | ----------- | -------------- |
| `< 50`      | 🟢 Clean    | `0.02 BREEZO`  |
| `50 – 100`  | 🟡 Moderate | `0.01 BREEZO`  |
| `100 – 300` | 🔴 Polluted | `0.005 BREEZO` |
| `> 300`     | ⚫ Hazardous | `0 BREEZO`     |

---

# 🔐 Security

BREEZO includes multiple protection layers.

## Replay Attack Protection

Sensor packets older than 60 seconds are rejected.

---

## NaCl Signature Verification

Only legitimate hardware devices can publish readings.

---

## On-Chain Ownership Validation

Backend validates:

```text
node.owner === expectedWallet
```

before syncing rewards.

---

## PDA-Based Treasury Authority

Treasury funds move only through verified program instructions.

---

## Admin-Protected Withdrawals

Only the admin wallet can execute treasury withdrawals.

---

# 📁 Project Structure

```text
/
├── breezo/                    # React frontend
│   ├── package.json
│   ├── vite.config.js
│   └── src/
│
├── server/                    # Node.js backend
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│
├── program/                   # Anchor smart contract
│   ├── programs/
│   ├── tests/
│   └── Anchor.toml
│
└── hardware/                  # ESP32 firmware
    └── sensor-code/
```

---

# 🚀 Getting Started

# Prerequisites

* Node.js ≥ 18
* MongoDB
* Solana CLI
* Anchor CLI
* Funded Solana wallet

---

# Clone Repository

```bash
git clone https://github.com/your-org/breezo-network
cd breezo-network
```

---

# Frontend Setup

```bash
cd breezo
npm install
npm run dev
```

Frontend:

```text
http://localhost:5173
```

---

# Backend Setup

```bash
cd server
npm install
cp .env.example .env
npm run dev
```

Starts:

* Express API
* Socket.IO
* Telegram bot
* payment poller

---

# Smart Contract Deployment

```bash
cd program

anchor build
anchor deploy --provider.cluster devnet
```

---

# Treasury Initialization

```bash
node scripts/initTreasury.js
```

Fund treasury:

```bash
spl-token transfer <MINT> 10000 <TREASURY_ATA> --fund-recipient
```

---

# 🔧 Environment Variables

```env
# MongoDB
MONGODB_URI=mongodb+srv://...

# Solana
SOLANA_RPC=https://api.devnet.solana.com
PROGRAM_ID=2CZ1WzjHhgbBFaRaTxhLpdKQKAEsYDFga6bbuQRHfCJu

# Backend Authority
BACKEND_KEYPAIR=[...]

# Treasury
SOLANA_TREASURY_WALLET=<wallet>

# Token
BREEZO_TOKEN_MINT=soQUnxjoEMCMxBroyS4AvrtVn2JCtPZnR3N53NA5AvU

# Telegram / Payments
TELEGRAM_BOT_TOKEN=<token>
HELIUS_API_KEY=<helius>

# AI
AI_API_KEY=<provider_key>

# Auth
JWT_SECRET=<jwt_secret>
```

---

# 🌐 API Overview

| Endpoint      | Purpose                  |
| ------------- | ------------------------ |
| `/nodes`      | Node management          |
| `/weather/*`  | AQI + environmental APIs |
| `/credit/add` | Reward sync              |
| `/claim`      | Claim rewards            |
| `/dashboard`  | User dashboard           |
| `/telegram/*` | Bot + payments           |

---

# 🧰 Tech Stack

| Layer      | Technology                          |
| ---------- | ----------------------------------- |
| Hardware   | ESP32, C++, NaCl                    |
| Backend    | Node.js, Express, MongoDB, Mongoose |
| Real-Time  | Socket.IO                           |
| Blockchain | Solana, Anchor, SPL Token           |
| Frontend   | React, Vite, Recharts               |
| Wallets    | `@solana/wallet-adapter`            |
| Auth       | JWT                                 |
| Payments   | Solana Pay, Helius                  |
| AI         | LLM Integration                     |
| Deployment | Docker, VPS / Cloud                 |

---

# 🔮 Future Vision

* global hyperlocal AQI network
* on-chain environmental analytics
* DAO-governed sensor incentives
* AI-powered pollution prediction
* carbon and sustainability integrations
* mobile-native citizen dashboards
* decentralized environmental marketplace

---

# 📜 License

MIT License

See `LICENSE` for details.

---

<div align="center">

## 🌿 Built with BREEZO

### Clean Air • Open Data • Decentralized Infrastructure

</div>
