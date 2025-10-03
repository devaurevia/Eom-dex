# EOM DEX — Architecture

> Gen-3 decentralized exchange: non-custodial, **no pooled AMM**, hybrid **orderbook + escrow**, and multi-chain routing.

---

## Core Components

### 1) Smart Contracts
- **EscrowVault.sol** → Locks project/dev allocations, controlled unlocks, burn hook.
- **OrderBook.sol** → Post/cancel orders, proof-based matching, partial fills.
- **Router.sol** → Routes to internal orderbook, external DEXs, or CEX connectors.
- **Governance.sol** → Token-based voting for fees, listings, upgrades.

### 2) Backend Services
- **Matching Engine** → Stateless off-chain matcher, on-chain settlement.
- **Quote Service** → Provides indicative spreads, aggregates CEX/DEX depth.
- **Escrow Monitor** → Tracks unlocks, burns, compliance alerts.

### 3) Frontend (dApp)
- Built with Next.js / React.
- Modules: Swap, Orderbook, Portfolio, Governance.
- Wallet integration: MetaMask, WalletConnect, Phantom/Solana adapters.

---

## System Flow

```mermaid
flowchart TD
  U[User Wallet] -->|Place Order| F[Frontend dApp]
  F --> B[Backend Engine]
  B -->|Validate + Match| C[OrderBook.sol]
  B -->|Escrow Check| E[EscrowVault.sol]
  C --> X[Trade Settlement]
  E --> X
  X --> U

---

Escrow Mechanism

Project/dev tokens are locked in EscrowVault (third-party custodian optional in early phase).

Orders may partially draw from escrow to guarantee fills if counterparties are missing (with daily/global limits).

Unlocks are time-vested, with ≥50% automatically burned at each unlock event.

Fail-safe: Escrow cannot be drained without multi-sig + governance approval.



---

Cross-Chain Design

Phase 1: ETH, BNB, SOL, BTC (wrapped).

Phase 2: Integration with emerging L1/L2 via pluggable router adapters.

Phase 3: Unified aggregator — auto-routing between EOM orderbook, external DEXs, and CEX liquidity.



---

Security Model

Third-party audits (static + manual).

Real-time monitoring of escrow reserves and spreads.

Circuit breaker: per-market halt + withdrawal timelocks on anomalies.

On-chain governance required for protocol-level changes.



---

Parameters (Initial)

Maker fee: 0.10% (long-term goal ≤0.05%).

Taker fee: 0.25% (decreasing with volume milestones).

OTC routing: threshold-based (e.g., ≥10 BTC routed off-book).

Escrow backstop: max 20% of daily unlocks usable for fills.



---

Summary

EOM DEX combines the speed of orderbooks, the safety of escrow, and the reach of multi-chain routing.
This architecture ensures sustainable liquidity without relying on fragile AMM/LP models.

---
