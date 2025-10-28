# PV3 Architecture Diagrams

Professional Mermaid diagrams showing the complete PV3 platform architecture.

## How to View

These are **Mermaid** diagrams that render beautifully on:
- GitHub (automatic rendering)
- GitLab (automatic rendering)
- VS Code (with Mermaid extension)
- Online: https://mermaid.live/

## Diagrams

### 1. System Architecture
**File:** `system-architecture.mmd`

**Shows:**
- Frontend Layer (Web + Mobile + Web3)
- Backend Microservices (34+ services)
- Data Layer (PostgreSQL + Redis + LRU Cache)
- Blockchain Layer (Solana + Light Protocol + Wormhole)
- External Services (Sentry, Cloudflare, Railway, Privy)

**View:** [system-architecture.mmd](./system-architecture.mmd)

---

### 2. Match Flow Sequence
**File:** `match-flow.mmd`

**Shows:**
- Complete match lifecycle from creation to payout
- Session vault interactions
- Smart contract escrow
- Game engine validation
- Anti-cheat checks
- Ed25519 signature verification
- Payout distribution

**View:** [match-flow.mmd](./match-flow.mmd)

---

### 3. Session Vault System
**File:** `session-vault.mmd`

**Shows:**
- User wallet to session vault flow
- Gaming balance vs infrastructure credit
- Match escrow mechanisms
- Platform fee distribution
- Treasury and referral pool
- Gas-free gaming architecture

**View:** [session-vault.mmd](./session-vault.mmd)

---

### 4. 6-Layer Anti-Cheat
**File:** `anti-cheat.mmd`

**Shows:**
- Layer 1: Server-side validation
- Layer 2: Timing analysis (bot detection)
- Layer 3: Pattern detection (win rate analysis)
- Layer 4: Replay hash verification
- Layer 5: Manual review queue
- Layer 6: Community reporting
- Penalty system flow

**View:** [anti-cheat.mmd](./anti-cheat.mmd)

---

### 5. Earning Ecosystem
**File:** `earning-ecosystem.mmd`

**Shows:**
- Revenue sources (matches, bridge, withdrawals, tournaments)
- Distribution to treasury and referral pool
- Affiliate tier system (10-25% forever)
- KOL custom deals (up to 30%)
- Developer revenue share (50%)
- Streamer earnings (70/30 subs, 90/10 tips)
- Example monthly earnings at scale

**View:** [earning-ecosystem.mmd](./earning-ecosystem.mmd)

---

### 6. Privacy Layer (Light Protocol)
**File:** `privacy-layer.mmd`

**Shows:**
- Zero-knowledge proof generation
- State compression via Light Protocol
- Merkle tree accumulation
- Nullifier hash for double-spend prevention
- Public verifiability without identity reveal
- Anonymous betting and private winnings
- Cost reduction through compression

**View:** [privacy-layer.mmd](./privacy-layer.mmd)

---

### 7. Cross-Chain Bridge (Wormhole)
**File:** `cross-chain-bridge.mmd`

**Shows:**
- ETH/BSC/Polygon source chains
- Wormhole Guardian Network (19 validators)
- VAA (Verifiable Action Approval) generation
- Multi-signature consensus (13+ guardians)
- Wrapped asset creation on Solana
- Session vault integration
- Atomic swap guarantees
- Security model and fee structure

**View:** [cross-chain-bridge.mmd](./cross-chain-bridge.mmd)

---

### 8. WebSocket Real-Time Architecture
**File:** `websocket-realtime.mmd`

**Shows:**
- 100K+ concurrent connection capacity
- Load-balanced WebSocket gateways
- Redis Pub/Sub message bus
- Distributed state management
- Authoritative game engine
- Connection recovery architecture
- Sub-10ms latency optimization
- Horizontal scaling strategy
- Anti-cheat integration points

**View:** [websocket-realtime.mmd](./websocket-realtime.mmd)

---

### 9. Developer SDK Integration
**File:** `developer-sdk.mmd`

**Shows:**
- Game development workflow
- SDK initialization and configuration
- Automatic anti-cheat wrapping
- Platform deployment process
- Revenue tracking system
- 50% revenue share distribution
- Real-time analytics dashboard
- Example earnings at scale
- Complete developer experience

**View:** [developer-sdk.mmd](./developer-sdk.mmd)

---

## Rendering Locally

### VS Code
1. Install "Markdown Preview Mermaid Support" extension
2. Open any `.mmd` file
3. Right-click → "Open Preview to the Side"

### Command Line
```bash
# Install Mermaid CLI
npm install -g @mermaid-js/mermaid-cli

# Generate PNG
mmdc -i system-architecture.mmd -o system-architecture.png

# Generate SVG (scalable)
mmdc -i system-architecture.mmd -o system-architecture.svg
```

### Online
1. Go to https://mermaid.live/
2. Copy diagram code
3. Edit and export as PNG/SVG/PDF

---

## Diagram Colors

**Standard Color Scheme:**
- 🟣 Purple (#9333EA) - Smart Contracts / Core Platform
- 🟢 Green (#14F195) - Solana Blockchain
- 🔵 Blue (#3B82F6) - Backend Services
- 🔴 Red (#EF4444) - Critical Paths / Escrow
- 🟡 Yellow (#F59E0B) - Infrastructure / Fees
- 🟠 Orange (#F38020) - External Services

---

**These diagrams are production-grade. Use them for:**
- Investor presentations
- Technical documentation
- Developer onboarding
- Hackathon submissions
- Partner discussions
