# EOM DEX Architecture

This document describes the high-level system design of **EOM DEX**, a next-generation decentralized exchange built without traditional AMM liquidity pools. Instead, EOM uses **escrow-backed orderbooks** and **multi-chain routing** to ensure sustainable liquidity and user trust.

---

## 1) System Components

### 1. Smart Contracts
- **OrderBook.sol** → Manages orders, matching, and escrow references.  
- **EscrowVault.sol** → Holds locked project/dev tokens and partially supplies liquidity for fills.  
- **Governance.sol** → Token-based voting for listings, fee adjustments, and upgrades.  

### 2. Backend Services
- **Order Matching Engine** → Stateless, off-chain logic that validates and matches orders before committing on-chain.  
- **Quote Service** → Provides indicative prices, spreads, and depth aggregated from external CEX/DEX connectors.  
- **Escrow Monitor** → Tracks unlock schedules, burns, and escrow release.  

### 3. Frontend (dApp)
- Built with **Next.js / React**.  
- Modules: Swap, Orderbook, Portfolio, Governance.  
- Wallet integration: MetaMask, WalletConnect, Solana wallets.  

---

## 2) System Flow

```mermaid
flowchart TD
    U[User Wallet] --> F[Frontend dApp]
    F --> B[Backend Engine]
    B --> Q[Quote Service]
    B --> R[Cross-chain Router]
    B --> C[OrderBook Contract]
    B --> E[EscrowVault Contract]
    R --> C
    R --> D[External DEX/CEX]
    C --> X[On-chain Settlement]
    E --> X
    X --> U
Explanation:

1. User places an order via the dApp.


2. Backend provides indicative pricing from Quote Service and available routes via Router.


3. Matching logic decides whether to fill from EOM’s orderbook, external DEX/CEX, or both.


4. Settlement is finalized on-chain via OrderBook.sol and EscrowVault.sol.




---

3) Order Lifecycle

1. Place → User signs intent (EIP-712); backend validates margin, limit, expiry.


2. Quote → Backend aggregates prices, returns indicative spread and estimated cost (fees, gas, slippage target).


3. Route → Router selects best execution path (internal book vs external).


4. Commit → Transaction committed to OrderBook.sol.


5. Fill/Partial Fill → If counterparty short, EscrowVault supplies partial liquidity.


6. Finalize → Balances and order status updated; events indexed for UI.




---

4) Escrow Mechanism

Project tokens locked in EscrowVault.sol or third-party custodian (early phase).

Liquidity assist: Can top-up missing order fills within set limits.

Vesting/Unlock: Flexible schedule; ≥50% of tokens burned at each unlock event (default, modifiable via governance).

Safety rails: Daily/weekly unlock caps, circuit breaker, allowlist receivers.

Custodian option (phase 1) for compliance; multi-sig enforced.



---

5) Cross-Chain Design

Phase 1: Ethereum, BNB, Solana, BTC (wrapped). Router selects intra-chain + major aggregators.

Phase 2: Support for new L1/L2 via adapter modules; optional third-party bridges for settlement.

Phase 3: Unified aggregator auto-routes between EOM orderbook, external DEX aggregators, and CEX liquidity.



---

6) Fees & Pricing

Platform fee: 0.35% baseline (adjustable by governance).

Maker/Taker model: 0.10% / 0.25% split (configurable per market).

Referral/Affiliate: Share from platform fee.

OTC blocks: Lower fee path for large trades.

Best-execution: Router optimizes net cost (fees + gas + slippage).



---

7) Security & Risk Controls

Audits: At least 1 external auditor before public launch.

Upgradability: Proxy contracts with time-lock + multi-sig.

Monitoring: Alerts on spread anomalies, abnormal unlocks, suspicious withdrawals.

Rate limiting: API + per-address throttling.

Fail-safe: Pause per-market or globally, without locking user funds.



---

8) Performance Targets

UX confirmation <1s (optimistic UI).

On-chain settlement tied to L2 finality (low fees recommended).

Stateless backend with horizontal scaling; Redis for cache.



---

9) Environments

dev → Hardhat/Anvil local fork.

test → Sepolia / BNB Testnet / Solana Devnet.

staging → Pre-release contracts with throttled traffic.

prod → Multi-region, HSM-protected operational keys.



---

10) External Dependencies

RPC providers (dedicated).

DEX aggregators (e.g., 1inch, 0x).

Broker APIs (for CEX routing).

Third-party custodians (optional).



---

11) Roadmap (Technical)

M1–M2: OrderBook.sol, EscrowVault.sol, basic dApp swap/order, indexer.

M3–M4: Quote Service, Router v1, external audit, public testnet.

M5–M6: L2 integration, OTC path, affiliate program, wave-1 mainnet.

M7+: Multi-chain adapters, optional custodians, live governance.
