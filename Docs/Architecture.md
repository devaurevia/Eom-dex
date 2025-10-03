# EOM DEX Architecture

## Overview
EOM DEX is designed as a hybrid orderbook exchange with escrow-backed settlement.  
The architecture combines smart contracts, backend services, and frontend dApp to ensure secure, low-slippage trading across multiple chains.

## Core Components
1. **Smart Contracts**
   - **EscrowVault.sol** → Holds project tokens and stablecoins to guarantee order execution.
   - **OrderBook.sol** → On-chain ledger for bids/asks.
   - **Router.sol** → Cross-chain execution and integration with external DEX/CEX liquidity.
   - **Governance.sol** → Token-based voting for listings, fees, upgrades.

2. **Backend Services**
   - **Order Matching Engine** → Off-chain, stateless, runs matching logic before committing to chain.
   - **Quote Service** → Provides indicative spreads and aggregated depth from CEX/DEX connectors.
   - **Escrow Monitor** → Tracks unlocks, burns, and escrow release schedules.

3. **Frontend (dApp)**
   - Built with Next.js / React.
   - Modules: Swap, Orderbook, Portfolio, Governance.
   - Wallet integration: MetaMask, WalletConnect, Solana wallets.

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

## Escrow Mechanism
- Dev/project tokens are locked in **EscrowVault**.  
- Orders draw partially from escrow to guarantee fills if counterparties are missing.  
- Unlocks are time-vested, with **≥50% burned** on each unlock event.  
- Third-party custodian may be used in early phase for compliance.  

---

## Cross-Chain Design
- **Phase 1:** ETH, BNB, SOL, BTC ecosystem (wrapped).  
- **Phase 2:** Integration with emerging L1/L2 via router modules.  
- **Phase 3:** Unified aggregator, auto-routing best price between EOM orderbook, CEX, and external DEX.  

---

## Security Considerations
- Contracts audited by independent third-party firms.  
- Real-time monitoring of escrow reserves.  
- Fail-safe: escrow cannot be drained without governance approval.  

---

## Summary
**EOM DEX** blends the speed of an orderbook, the safety of escrow, and the reach of multi-chain routing.  
This ensures sustainable liquidity without relying on vulnerable LP models.
