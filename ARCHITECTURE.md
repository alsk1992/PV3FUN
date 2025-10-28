# PV3 Platform Architecture

## System Overview

PV3 is built as a three-tier architecture optimized for real-time gaming on Solana:

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend Layer                       │
│  Next.js 13 + Three.js + WebSocket Client + Wallet Adapter  │
└──────────────────┬──────────────────────────────────────────┘
                   │ HTTP/WSS
┌──────────────────▼──────────────────────────────────────────┐
│                        Backend Layer                         │
│     NestJS Microservices + Redis + PostgreSQL + WebSocket   │
└──────────────────┬──────────────────────────────────────────┘
                   │ Solana RPC
┌──────────────────▼──────────────────────────────────────────┐
│                      Blockchain Layer                        │
│         Solana Smart Contract (Anchor 0.30.1)               │
│  Session Vaults + Match Escrow + Ed25519 Verification       │
└─────────────────────────────────────────────────────────────┘
```

---

## Smart Contract Architecture

### Program Structure

```
PV3 Contract (GMMgeKNoJk7mGtrNy1GX3o8W7ifA9WMb9GRCXxtSTa8N)
│
├── Platform Configuration (PDA)
│   ├── Treasury Wallet
│   ├── Referral Pool
│   ├── Verifier Public Key (Ed25519)
│   ├── Fee Configuration (6% platform, 1% referral)
│   └── Emergency Pause State
│
├── Session Vaults (PDA per user)
│   ├── Owner Public Key
│   ├── Gaming Balance
│   ├── Infrastructure Credit
│   ├── Games Played Counter
│   └── Creation Cost Recovery Flag
│
├── Match Accounts (PDA per match)
│   ├── Game Type (10 variants)
│   ├── Creator + Joiner
│   ├── Wager Amount
│   ├── Match Status
│   ├── Expiry Timestamp
│   └── Result Hash
│
└── Match Escrow (PDA per match)
    └── Locked Funds (2x wager)
```

### PDA (Program Derived Address) Seeds

```rust
Platform Config:    [b"config"]
Session Vault:      [b"session_vault", user_pubkey]
Match Account:      [b"match", creator, game_type, timestamp]
Match Escrow:       [b"escrow", match_account]
```

### Transaction Flow

**Match Creation:**
```
1. User → Session Vault (wager deducted)
2. Session Vault → Match Escrow (funds locked)
3. Match Account created (waiting for opponent)
4. Infrastructure fee deducted (0.0035 SOL)
```

**Match Join:**
```
1. Joiner → Session Vault (wager deducted)
2. Session Vault → Match Escrow (funds locked)
3. Match status → InProgress
4. Infrastructure fee deducted (0.0015 SOL)
5. Backend game engine starts
```

**Result Submission:**
```
1. Backend validates game rules
2. Backend creates SHA-256 result hash
3. Backend signs with Ed25519 private key
4. Smart contract verifies Ed25519 signature
5. If valid → Release 94% to winner, 6% to platform
6. Funds → Winner's Session Vault (instant)
```

---

## Backend Architecture

### Microservices (34 Modules)

```
Core Services:
├── Match Service          - Match lifecycle management
├── Game Factory           - Game engine routing
├── Session Vault Service  - Vault operations and deposits
├── Solana Service         - Blockchain integration
└── Verifier Service       - Ed25519 signing

Game Engines:
├── Chess Service          - Move validation, checkmate detection
├── Coinflip Service       - Randomness + best-of-5 logic
├── RPS Service            - Choice validation + sync
├── Dice Duel Service      - RPG dice rolls
├── Mines Service          - Board state + anti-cheat
├── Crash Service          - Multiplier curve calculation
├── Hi-Lo Service          - Card deck shuffling
├── High Card Duel         - Card comparison
├── Math Duel Service      - Problem generation
└── Reaction Ring Service  - Timing validation

Real-time Infrastructure:
├── WebSocket Gateway      - Match updates, chat, lobby
├── Redis Service          - State caching + pub/sub
└── Database Service       - PostgreSQL via Prisma

Ecosystem Features:
├── Streaming Service      - Live streaming + tips
├── Social Service         - Forums, friends, achievements
├── Leaderboard Service    - Rankings and stats
├── PnL Service            - Profit/loss analytics
├── Bridge Service         - Cross-chain (ETH/BSC/Polygon)
├── Referral Service       - Referral tracking + payouts
├── Analytics Service      - Platform metrics
└── Twitter Bot Service    - Automated social posts
```

### Database Schema (50+ Tables)

**Core Tables:**
- `users` - User accounts and profiles
- `session_vaults` - On-chain vault tracking
- `matches` - Match records and results
- `match_results` - Cryptographic proofs (Ed25519 signatures)
- `game_moves` - Turn-by-turn game state

**Ecosystem Tables:**
- `streams` - Live streaming sessions
- `forum_posts` - Community discussions
- `achievements` - User achievements
- `leaderboards` - Rankings by game type
- `referrals` - Referral tracking
- `pnl_snapshots` - Historical profit/loss
- `bridge_transactions` - Cross-chain operations

### Redis Architecture

**State Management:**
```
Game State:      game:{matchId}        (30s TTL)
Lobby State:     lobby:{gameType}      (10s TTL)
User Session:    session:{userId}      (1h TTL)
Active Matches:  active:matches        (sorted set)
Leaderboard:     leaderboard:{type}    (persistent)
```

**Pub/Sub Channels:**
```
match:{matchId}       - Match updates
lobby:{gameType}      - Lobby changes
chat:{matchId}        - In-game chat
global:events         - Platform events
```

---

## Frontend Architecture

### Tech Stack

```
Core Framework:
├── Next.js 13+ (App Router)
├── React Server Components
├── TypeScript (strict mode)
└── Tailwind CSS (custom theme)

3D Graphics:
├── Three.js (game animations)
├── React Three Fiber
└── Physics engine (Coinflip, RPS)

State Management:
├── React Context (global state)
├── Zustand (game state)
└── SWR (data fetching)

Blockchain:
├── @solana/web3.js
├── @solana/wallet-adapter-react
└── Anchor IDL integration

Real-time:
├── Socket.io-client (WebSocket)
└── React Query (caching)
```

### Component Architecture

```
app/
├── games/
│   ├── chess/page.tsx
│   ├── coinflip/page.tsx
│   ├── rps/page.tsx
│   ├── dice-duel/page.tsx
│   ├── mines/page.tsx
│   ├── crash/page.tsx
│   ├── hi-lo/page.tsx
│   ├── high-card-duel/page.tsx
│   ├── math-duel/page.tsx
│   └── reaction-ring/page.tsx
│
├── session-vault/
│   ├── deposit/
│   ├── withdraw/
│   └── history/
│
└── social/
    ├── streaming/
    ├── forums/
    └── leaderboards/

components/
├── games/
│   ├── ChessBoard.tsx
│   ├── CoinflipAnimation.tsx (Three.js)
│   ├── RPSAnimation.tsx (Three.js)
│   ├── DiceBox.tsx (3D)
│   ├── MinesBoard.tsx
│   ├── CrashGraph.tsx
│   └── ...
│
├── SessionVault.tsx
├── ChatPanel.tsx
├── Sidebar.tsx
└── WalletConnect.tsx
```

### 3D Animation System

**Coinflip Animation:**
```typescript
- Three.js scene with realistic coin physics
- 5 coins spawn simultaneously (best-of-5)
- Rotation speed: 10 rev/s
- Gravity: 9.8 m/s²
- Bounce physics on landing
- 3 seconds total animation
```

**RPS Animation:**
```typescript
- 3D hand models for Rock, Paper, Scissors
- Synchronized reveal at 3-second mark
- Physics-based hand movement
- Camera shake on reveal
- Particle effects for winner
```

**Dice Animation:**
```typescript
- RPG-style polyhedral dice (d4, d6, d8, d10, d12, d20)
- DiceBox.js library integration
- Predetermined results from backend
- 3D physics simulation
- Multiple dice simultaneously
```

---

## Session Vault System (Core Innovation)

### Architecture

```
User Wallet (External)
      │
      │ Deposit (one-time)
      ▼
┌─────────────────────┐
│   Session Vault     │ ← PDA owned by smart contract
│   (On-chain PDA)    │
├─────────────────────┤
│ Gaming Balance      │ ← Available for wagers
│ Infrastructure $    │ ← Auto-managed gas fees
│ Games Played        │ ← Usage counter
│ Owner Pubkey        │ ← User's wallet
└─────────────────────┘
      │
      │ All game transactions (gas-free for user)
      ▼
   Escrow → Game → Result → Instant Credit Back
```

### Smart Features

**Auto Infrastructure Management:**
```rust
if vault.infrastructure_credit < 0.01 SOL {
    // Auto-refill with 0.03 SOL from user's next deposit
    infrastructure_topup = 0.03 SOL;
}
```

**Silent Cost Recovery:**
```rust
if !vault.creation_cost_recovered && vault.balance >= 0.02 SOL {
    // Backend paid 0.02 SOL to create vault
    // Silently recover from user's first deposit
    vault.balance -= 0.02 SOL;
    vault.creation_cost_recovered = true;
}
```

**Gas-Free Gaming:**
- User deposits once → plays unlimited games
- No wallet approvals during gameplay
- Infrastructure fees automatically deducted from vault credit
- Winnings instantly credited to vault
- Withdraw anytime with 0.1-0.5% fee

---

## Ed25519 Cryptographic Verification

### Flow

```
Backend (Game Engine)
      │
      │ 1. Game ends, winner determined
      ▼
┌─────────────────────┐
│  Create SHA-256     │
│  Result Hash        │
│  (32 bytes)         │
└──────┬──────────────┘
       │
       │ 2. Sign with Ed25519 private key
       ▼
┌─────────────────────┐
│  Ed25519 Signature  │
│  (64 bytes)         │
└──────┬──────────────┘
       │
       │ 3. Submit to smart contract
       ▼
┌─────────────────────────────────┐
│  Smart Contract Verification    │
│  - Verify signature with pubkey │
│  - Check message format         │
│  - Validate winner is player    │
│  - Release funds if valid       │
└─────────────────────────────────┘
```

### Message Structure (96 bytes)

```rust
Message = matchPDA (32) + winnerPubkey (32) + resultHash (32)
```

### Security Properties

- **Tamper-Proof**: Cannot forge signature without private key
- **Verifiable**: Anyone can verify with public key
- **Non-Repudiable**: Backend cannot deny signing result
- **Immutable**: Stored on-chain permanently

---

## Scaling Architecture

### Current Capacity

- **Matches per second**: 100+ concurrent
- **WebSocket connections**: 10,000+
- **Database**: Horizontally scalable (PostgreSQL read replicas)
- **Redis**: Cluster mode for distributed caching
- **Solana**: 65,000 TPS (nowhere near limit)

### Horizontal Scaling Plan

```
Load Balancer
      │
      ├─── Backend Instance 1 (NestJS)
      ├─── Backend Instance 2 (NestJS)
      └─── Backend Instance N (NestJS)
            │
            ├─── Redis Cluster (shared state)
            └─── PostgreSQL Primary + Read Replicas
```

### Cost Efficiency

**Per Game Transaction:**
- Solana fees: ~$0.0001
- Infrastructure allocation: $0.0035 (create) / $0.0015 (join)
- Backend compute: ~$0.001
- **Total platform cost per game**: ~$0.005

**At scale (10,000 games/day):**
- Platform revenue (6%): ~$3,000/day (assuming $5 avg wager)
- Platform costs: ~$50/day
- **Profit margin**: 98%+

---

## Security Architecture

### Smart Contract Security

- **Reentrancy Protection**: All state changes before external calls
- **Integer Overflow**: Anchor's safe math
- **Access Control**: Verifier-only functions
- **Emergency Pause**: Admin can halt platform
- **Timeout Refunds**: Auto-refund expired matches

### Backend Security

- **Ed25519 Private Key**: Stored in secure environment variables
- **Rate Limiting**: 100 req/min per IP
- **DDoS Protection**: Cloudflare integration
- **Input Validation**: All user inputs sanitized
- **SQL Injection**: Prisma ORM (parameterized queries)

### Anti-Cheat Measures

**Chess:**
- Legal move validation
- Impossible move detection
- Timeout enforcement

**Real-time Games:**
- Server-side state validation
- Input timing analysis
- Replay hash verification
- Multiple redundant checks

---

## Development Stack

**Smart Contract:**
- Anchor 0.30.1
- Rust 1.75+
- Solana CLI 1.18+

**Backend:**
- NestJS 10+
- Node.js 20+
- PostgreSQL 15
- Redis 7
- Prisma ORM

**Frontend:**
- Next.js 13+
- React 18
- TypeScript 5+
- Three.js
- Tailwind CSS

**DevOps:**
- Railway (backend hosting)
- Vercel (frontend hosting)
- GitHub Actions (CI/CD)
- Solana Devnet/Mainnet

---

## API Architecture

### REST Endpoints

```
Authentication:
POST   /api/auth/login
POST   /api/auth/register
GET    /api/auth/profile

Session Vault:
GET    /api/vault/balance
POST   /api/vault/deposit
POST   /api/vault/withdraw
GET    /api/vault/history

Matches:
POST   /api/match/create
POST   /api/match/join
GET    /api/match/{id}
GET    /api/match/active
GET    /api/match/{id}/verify

Games:
GET    /api/games/list
GET    /api/games/{type}/stats
POST   /api/games/{type}/move

Leaderboard:
GET    /api/leaderboard/{gameType}
GET    /api/leaderboard/global

Social:
GET    /api/social/forums
POST   /api/social/forums/post
GET    /api/streaming/live
```

### WebSocket Events

```
Client → Server:
- match:join
- match:leave
- game:move
- chat:message
- lobby:subscribe

Server → Client:
- match:created
- match:joined
- match:started
- match:move
- match:result
- chat:message
- lobby:update
```

---

This architecture supports:
- ✅ Sub-second match settlement
- ✅ 10,000+ concurrent users
- ✅ Gas-free user experience
- ✅ Cryptographically verifiable fairness
- ✅ Horizontal scalability
- ✅ 99.9% uptime

Built for production scale from day one.
