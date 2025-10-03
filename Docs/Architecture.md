
# EOM DEX Architecture

This document outlines the high-level architecture of **EOM DEX**, a next-generation decentralized exchange that replaces traditional AMM pools with **escrow-backed orderbooks** and **multi-chain routing**.

---

## 1) System Components

### a. Smart Contracts
- **OrderBook.sol** → Manages orders, matching, and escrow references.  
- **EscrowVault.sol** → Holds locked project/dev tokens and partially supplies liquidity for fills.  
- **Governance.sol** → Token-based voting for listings, fee adjustments, and upgrades.  

### b. Backend Services
- **Order Matching Engine** → Stateless off-chain logic that validates and matches orders before committing on-chain.  
- **Quote Service** → Provides indicative prices, spreads, and liquidity depth aggregated from external CEX/DEX connectors.  
- **Escrow Monitor** → Tracks unlock schedules, burns, and escrow release.  

### c. Frontend (dApp)
- Built with **Next.js / React**.  
- Modules: Swap, Orderbook, Portfolio, Governance.  
- Wallet support: MetaMask, WalletConnect, Solana wallets.  

---

## 2) System Flow

| Step | Component            | Action                                    |
|------|----------------------|-------------------------------------------|
| 1    | User Wallet          | Sends order to frontend                   |
| 2    | Frontend dApp        | Forwards request to backend engine        |
| 3    | Backend Engine       | Calls Quote Service & Router              |
| 4    | Router               | Chooses path (Orderbook / DEX / Escrow)   |
| 5    | Contracts            | Executes via OrderBook.sol / EscrowVault  |
| 6    | Settlement           | Final on-chain settlement                 |


Explanation

1. User places an order via the dApp.
2. Backend provides indicative pricing from Quote Service and available routes via Router.
3. Matching logic decides whether to fill from EOM orderbook, external DEX/CEX, or a mix.
4. Settlement finalizes on-chain via OrderBook.sol and EscrowVault.sol for partial fills.

---

3) Order Lifecycle

1. Place → User signs intent (EIP-712). Backend validates margin, limit, expiry.
2. Quote → Backend aggregates prices, returns indicative spread and estimated total cost (fees + gas).
3. Route → Router selects the best path (internal book vs external).
4. Commit → Transaction submitted to OrderBook.sol.
5. Fill/Partial Fill → If counterparties are insufficient, escrow covers partial fills under governance-defined limits.
6. Finalize → Balances, status, and history updated; events emitted for indexers/UI.

---

4) Escrow Mechanism

Project/dev tokens locked inside EscrowVault.sol or optionally via third-party custodians.

Liquidity Assist: Escrow can top-up missing liquidity for orders.
Vesting/Unlocks: Default burn ≥50% of tokens on each unlock event (modifiable via governance).
Safety Controls: Daily/weekly caps, circuit breakers, and allowlisted withdrawal addresses.

---

5) Cross-Chain Design

Phase 1: Ethereum, BNB Chain, Solana, BTC (wrapped). Router optimizes intra-chain routes.

Phase 2: Integration with new L1/L2 via adaptor modules, optional third-party bridges.

Phase 3: Unified aggregator auto-routing between EOM orderbook, external DEX aggregators, and CEX.

---

6) Fees & Pricing

Platform fee: 0.35%, configurable via governance.

Maker/Taker model: e.g., 0.10% / 0.25%.
Referral program: rewards from platform fee.

OTC large trades: lower negotiated fee path.

Best execution: Router evaluates total effective cost (fees + gas + slippage).

---

7) Security & Risk Controls

Audits: Minimum one independent auditor before launch.

Upgradability: Proxy with time-lock and multi-sig.

Monitoring: Alerts for abnormal spread, unlock spikes, or suspicious drains.

Rate Limiting: Per-user API and transaction throttling.

Fail-safes: Market pause and global emergency stop (without freezing user funds).

---

8) Performance Targets

UX confirmation < 1s (optimistic UI).
On-chain settlement tied to underlying chain finality (L2 recommended).
Backend is stateless, horizontally scalable; caching via Redis.

---

9) Environments

Dev: Local anvil/hardhat, mainnet forking.
Test: Sepolia, BNB testnet, Solana devnet.
Staging: Pre-release with anonymized data.
Production: Multi-region, HSM for operational keys.

---

10) External Dependencies

RPC providers (dedicated).
DEX aggregators (1inch, 0x).
Broker APIs for CEX and OTC routing.
Optional custodians for escrow in early phases.

---

11) Roadmap (Technical, Year 1)

M1–M2: Implement OrderBook.sol, EscrowVault.sol, and basic Swap UI.
M3–M4: Add Quote Service, Router v1 (intra-chain), first external audit, public testnet.
M5–M6: Integrate low-cost L2, OTC module, affiliate program, mainnet v1 launch.
M7+: Multi-chain adaptors, optional custodians, fully activated governance.
