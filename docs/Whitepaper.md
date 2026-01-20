# YieldLoop + VaultOS — Unified Whitepaper
## Single Source of Truth (Build Spec)
### Modular DeFi Execution Engine + Vault Operating System

---

## Title Page

**Platform Name:** YieldLoop + VaultOS  
**Products Included:** YieldLoop Vaults + VaultOS Goal Vaults (Generationz Vault Type)  
**Document Type:** Unified Whitepaper / Protocol Build Specification (Non-Marketing)  
**Scope:** Complete mechanics, math, flows, constraints, state machines, safety systems, token systems, UX requirements  
**Decision Standard:** Final decisions only — no optionality, no ambiguity, no TBD  
**Author:** Todd Koletsky  
**Version:** 1.0.0  
**Date:** January 20, 2026  
**Status:** Finalized Architecture Spec (Ready for Implementation)  

---

## System Info (Locked Build Targets)

**Primary Chain (Final):** BNB Chain (BSC) — EVM  
**Reason:** Execution + tooling maturity + DEX liquidity + compatibility with PCS/BiSwap  
**DEX Venues (MVP1 Locked):**
- PancakeSwap (PCS)
- BiSwap

**Deposit Asset (All Vault Types):** USDT (BEP-20) — Final  
**Minimum Deposit (All Vault Types):** $100 USDT — Final  

**Initial Trading Inventory (Allowlist / Expandable):**
- BTCB (primary)
- BNB
- ETH (peg)
- XRP (peg)
- SOL (peg)

**Vault Types (Final):**
1) **YieldLoop Vaults**
   - Strategy proposal + user acceptance gate (mandatory)
   - 30-day loop cycle execution model (mandatory)
   - Cycle-end settlement and user choice (mandatory)

2) **VaultOS Goal Vaults (Generationz Vault Type)**
   - Bucket-based savings subaccounts (mandatory)
   - Lock terms: 90 days to 20 years (30-day increments) (mandatory)
   - Maturity withdrawal: penalty-free (mandatory)
   - Early withdrawal: Break-Lock penalty (mandatory)
   - Recovery system: mandatory and implemented by design (mandatory)

**Reward Asset Options (Final Platform Rule):**
- USDT payout option
- LOOP payout option

**Unified Platform Performance Fees (Final):**
- USDT payout performance fee: 20% of verified realized profit
- LOOP payout performance fee: 17.5% of verified realized profit

**Discount Policy (Final):**
- Discounts are applied identically across YieldLoop + VaultOS
- Discounts apply ONLY to performance fees (not penalties)
- Supporter/native discount rules are platform-wide and enforced by DiscountEngine

**Core Principles (Final):**
- Proof-based accounting (realized profit only)
- Deterministic settlement
- Guardrail-first execution (refuse trade > bad trade)
- Transparent fees, reserves, and token support mechanics
- Admin cannot access user funds

---

# Table of Contents (Mechanics-Complete / Final Decisions)

## 0. Executive Summary (Protocol Truth)
0.1 What the platform does (precise, non-marketing)  
0.2 What the platform does not do (explicit exclusions)  
0.3 Locked design choices (chain, assets, venues, vault types)  
0.4 Risk disclosures (mandatory)  
0.5 Module map (engine → vaults → strategies → token systems)  

---

## 1. Definitions, Truth Sources, and Ledger Model (Hard Rules)
1.1 Glossary  
1.2 Truth sources hierarchy (what the system trusts)  
1.3 Ledger types and meanings  
1.4 Profit definition (realized-only, strict verification)  
1.5 Settlement determinism rules  
1.6 Precision/decimals/rounding/dust rules  
1.7 Consent and compliance UX rules (mandatory confirmations)  

---

## 2. Architecture Overview (Full Stack)
2.1 Layer model: Core Engine / Vault Types / Strategy Plugins / Token Systems / UI  
2.2 Contract/module list and responsibilities  
2.3 Keeper/automation services and responsibilities  
2.4 Data model objects: Vault, Bucket, StrategyRun, ProfitEvent, ReserveLedger  
2.5 Global protocol state machine  
2.6 Vault state machine  
2.7 Strategy execution state machine  
2.8 Failure propagation model (safe failure defaults)  
2.9 Transparency logging requirements  

---

## 3. YieldLoopCore Execution Engine (Shared System)
3.1 Engine invariants and enforcement rules  
3.2 Permission model (who can execute what)  
3.3 VaultFactory/VaultRegistry mechanics  
3.4 Capital lifecycle mechanics (USDT → inventory → settle)  
3.5 Trade eligibility gating (mandatory preconditions)  
3.6 Trade execution lifecycle (step-by-step)  
3.7 Settlement lifecycle (step-by-step)  
3.8 GuardrailEngine mechanics (slippage, liquidity, drawdown, size caps, cooldowns)  
3.9 OracleAdapter mechanics (sanity checks + manipulation defense)  
3.10 MEV protection model (mandatory)  
3.11 Venue routing rules (PCS/BiSwap only)  
3.12 Refusal/no-trade rules (mandatory)  
3.13 Keeper job definitions (exact jobs, triggers, schedules)  
3.14 Pause system (what pause affects; withdrawals always preserved)  
3.15 Fallbacks (keeper down, oracle down, DEX degraded, gas spikes)  

---

## 4. Vault Interface Layer (Unified Vault Abstraction)
4.1 Unified vault interface (must be implemented by all vault types)  
4.2 Shared vault fields and ledgers  
4.3 Immutable vs mutable configuration rules  
4.4 Allowed user actions (platform-wide)  
4.5 Claim model (ledger truth + computation)  
4.6 Withdrawal model (platform-wide rules + vault-specific constraints)  
4.7 Notification model (mandatory events and triggers)  
4.8 Transparency model (profit proofs, fees, reserves, floor metrics)  

---

## 5. YieldLoop Vaults (Cycle-Based Vault Mechanics — Final)
5.1 Product rules and user expectations  
5.2 Deposit mechanics  
5.3 Strategy proposal generation mechanics (mandatory)  
5.4 Strategy acceptance mechanics (mandatory)  
5.5 30-day loop cycle mechanics (mandatory)
5.5.1 Cycle start rules  
5.5.2 In-cycle constraints  
5.5.3 Cycle end rules  
5.6 Execution mechanics within cycle  
5.7 Profit event mechanics within cycle  
5.8 Cycle settlement mechanics (exact math)  
5.9 Cycle-end user choice mechanics (compound/withdraw options)  
5.10 YieldLoop vault fallbacks and edge cases  

---

## 6. VaultOS Goal Vaults (Generationz Vault Type — Final)
6.1 Goal bucket model mechanics (subaccounts with labels)  
6.2 Deposit mechanics (USDT only; top-ups allowed)  
6.3 Lock schedule mechanics
6.3.1 Lock range: 90d → 20y  
6.3.2 Lock increments: 30-day increments  
6.3.3 Timestamp computation and enforcement  
6.4 VaultOS execution constraints (conservative baseline)
6.4.1 Allowed strategies (final list)  
6.4.2 Disallowed strategies (final list)  
6.4.3 Risk limits and guardrail tightening  
6.5 Withdrawal mechanics
6.5.1 Maturity withdrawal (penalty-free)  
6.5.2 Early withdrawal: Break Lock (penalty applies)  
6.5.3 Partial withdrawal policy: NOT ALLOWED (final)  
6.6 Break-Lock Penalty Engine (final mechanics)
6.6.1 Penalty schedule definition (20% → 5%)  
6.6.2 Monthly decrement logic  
6.6.3 Penalty preview requirements (mandatory UI + onchain view)  
6.6.4 Penalty routing mechanics  
6.6.5 Anti-gaming rules  
6.7 Recovery system (mandatory)
6.7.1 Unlock code model (hash rules; never plaintext)  
6.7.2 Recovery wallet (mandatory)  
6.7.3 Guardian threshold recovery (mandatory)  
6.7.4 Timelock recovery (mandatory)  
6.7.5 Recovery cancellation rules (mandatory theft defense)  
6.7.6 Beneficiary/inheritance rules (mandatory)  
6.8 Goal completion + gift mechanics (mandatory)
6.8.1 Gift vault transfer rules  
6.8.2 Gift claim rules  
6.8.3 Safe transfer restrictions  

---

## 7. Strategy Framework (Plugin Interface — Final)
7.1 Strategy interface (required functions and logs)  
7.2 Strategy input/output schema  
7.3 Strategy guardrail enforcement requirements  
7.4 Strategy allowlist update process (timelocked)  
7.5 Strategy set (MVP1 final)
7.5.1 Band/Grid strategy mechanics  
7.5.2 PCS ↔ BiSwap arbitrage strategy mechanics  
7.6 Strategy execution limits (final defaults)
7.6.1 Slippage caps  
7.6.2 Trade sizing caps  
7.6.3 Cooldowns  
7.6.4 Drawdown caps  
7.7 Strategy failure behavior and fallbacks  

---

## 8. Profit Events, Settlement, Claims, and Compounding (Unified)
8.1 Profit event definition (strict)  
8.2 Profit verification logic  
8.3 Settlement math (step-by-step formulas)  
8.4 Fee application timing  
8.5 Claim ledger mechanics  
8.6 Compounding ledger mechanics  
8.7 Negative performance rules (no fee; drawdown handling)  
8.8 Worked examples (mandatory)  

---

## 9. Fees, Discounts, and Routing (Final Math)
9.1 Fee types and definitions  
9.2 Performance fee mechanics
9.2.1 USDT payout fee: 20%  
9.2.2 LOOP payout fee: 17.5%  
9.3 DiscountEngine (final rules)
9.3.1 Supporter/native discount eligibility  
9.3.2 Warmup rules  
9.3.3 Reset rules  
9.3.4 Stacking rules (final)  
9.4 Fee routing wallets and splits (final)  
9.5 Routing update rules (timelocked; cannot affect past events)  
9.6 Transparency requirements  
9.7 Worked examples (mandatory)  

---

## 10. LOOP Token System (Floor + Reserve + Redemption — Final)
10.1 LOOP role summary  
10.2 LOOP issuance rules (when LOOP can be created/distributed)  
10.3 LOOP payout mechanics (USDT profit → LOOP swap → distribute)
10.3.1 Allowlisted swap venues  
10.3.2 Slippage caps  
10.3.3 Oracle sanity checks  
10.3.4 Swap failure fallbacks  
10.4 Redemption mechanics (final)
10.4.1 Eligibility rules  
10.4.2 Redemption request flow  
10.4.3 Redemption settlement flow  
10.4.4 Redemption throttling rules  
10.4.5 Redemption failure handling  
10.5 Redemption Reserve mechanics (final)
10.5.1 Funding sources  
10.5.2 Custody rules  
10.5.3 Reserve ledger accounting  
10.5.4 Reserve usage restrictions  
10.5.5 Reserve transparency requirements  
10.6 LOOP floor system mechanics (final)
10.6.1 Floor definition  
10.6.2 Floor enforcement mechanisms  
10.6.3 Floor adjustment mechanics  
10.6.4 Crash/illiquidity edge cases  

---

## 11. Security Model (Final)
11.1 Threat model  
11.2 Reentrancy and external call safety  
11.3 Oracle attack resistance  
11.4 MEV mitigation  
11.5 Keeper abuse prevention  
11.6 Admin compromise mitigation  
11.7 Upgrade safety model (timelock + multisig)  
11.8 Audit scope checklist  

---

## 12. Admin Control Boundaries (Final)
12.1 Admin can do (explicit list)  
12.2 Admin cannot do (explicit list)  
12.3 Emergency actions allowed (explicit limits)  
12.4 Parameter updates (timelock rules)  
12.5 Strategy activation (timelocked)  

---

## 13. UX / Frontend Requirements (Must Match Mechanics)
13.1 Full screen inventory list  
13.2 Deposit UX flows  
13.3 Strategy approval UX flows (YieldLoop)  
13.4 VaultOS bucket UX flows  
13.5 Claim/withdraw UX flows  
13.6 Break-lock UX flows  
13.7 Recovery UX flows  
13.8 Notifications/reminders (mandatory rules)  
13.9 Transparency dashboards (mandatory metrics)  

---

## 14. Roadmap (Final MVP1 Implementation Plan)
14.1 MVP1 scope lock  
14.2 Post-MVP strategy additions  
14.3 Post-MVP chain evaluation criteria (only after MVP1)  

---

# Appendices — Table of Contents (A–J)

- **Appendix A — Full Contract + Module Map**
  - A.1 Contract List and Responsibilities
  - A.2 Dependency Graph
  - A.3 Deployment Order
  - A.4 Addresses Registry Format

- **Appendix B — Vault State Machines**
  - B.1 YieldLoop Vault State Machine (Plain MD)
  - B.2 VaultOS Goal Vault State Machine (Plain MD)
  - B.3 Recovery State Machine (Plain MD)

- **Appendix C — Strategy Execution State Machines**
  - C.1 Universal Strategy Run Lifecycle
  - C.2 Band/Grid Strategy Execution Logic
  - C.3 PCS ↔ BiSwap Arbitrage Execution Logic
  - C.4 Failure Escalation Policy
  - C.5 Required Execution Logs

- **Appendix D — Default Parameter Sets (Conservative, Final)**
  - D.0 Global Defaults and System Constants
  - D.1 Slippage Caps
  - D.2 Liquidity Floors
  - D.3 Trade Sizing Caps
  - D.4 Cooldowns and Execution Frequency
  - D.5 Drawdown Caps and Auto-Pause Thresholds
  - D.6 Arbitrage Threshold Parameters
  - D.7 Band/Grid Strategy Parameters
  - D.8 Token Allowlist (MVP1 Final)
  - D.9 Venue Health Rules
  - D.10 Redemption Throttle Defaults
  - D.11 Floor Policy Defaults
  - D.12 Parameter Override Hierarchy
  - D.13 Parameter Change Governance Rules
  - D.14 Developer Acceptance Criteria

- **Appendix E — Full Worked Examples (Math, Step-by-Step)**
  - E.0 Shared Variables and Definitions
  - E.1 Profit Event — USDT Payout
  - E.2 Profit Event — LOOP Payout (Conversion)
  - E.3 Profit Event — LOOP Payout + Supporter Discount
  - E.4 YieldLoop Cycle Settlement Example (30-Day Cycle)
  - E.5 VaultOS Maturity Withdrawal Example
  - E.6 VaultOS Break-Lock Penalty Example (20% → 5% decay)
  - E.7 Redemption Request + Settlement Example (Queued)
  - E.8 Reserve Ledger Bookkeeping Example
  - E.9 No-Profit Event Example (Fee = 0)
  - E.10 LOOP Conversion Failure Fallback Example

- **Appendix F — Disclosure Templates (Final UI Copy)**
  - F.0 Global Disclosure Rules
  - F.1 YieldLoop Deposit Disclosure
  - F.2 YieldLoop Strategy Proposal Disclosure
  - F.3 YieldLoop Active Cycle Risk Disclosure
  - F.4 YieldLoop Cycle-End Option Disclosure
  - F.5 VaultOS Deposit + Lock Disclosure
  - F.6 VaultOS Break Lock Disclosure
  - F.7 VaultOS Recovery Disclosure
  - F.8 Beneficiary / Inheritance Disclosure
  - F.9 LOOP Payout Disclosure
  - F.10 Redemption Disclosure
  - F.11 No Insurance Notice

- **Appendix G — Full Glossary**
  - G.1 Core Terms
  - G.2 Trading / Strategy Terms
  - G.3 Accounting Terms
  - G.4 Fee and Discount Terms
  - G.5 LOOP Token Terms
  - G.6 VaultOS Terms
  - G.7 Security Terms
  - G.8 UI Terms

- **Appendix H — Event Schema (Final)**
  - H.0 Event Standards
  - H.1 VaultFactory Events
  - H.2 VaultRegistry Events
  - H.3 YieldLoopVault Events
  - H.4 VaultOSGoalVault Events
  - H.5 YieldLoopCore StrategyRun Events
  - H.6 Settlement and Profit Events
  - H.7 FeeRouter / DiscountEngine Events
  - H.8 ReserveVault / Redemption Events
  - H.9 LOOP Conversion Events
  - H.10 Admin / Governance / Timelock Events
  - H.11 Reason Code Enum List

- **Appendix I — Test Plan + Acceptance Criteria**
  - I.1 Unit Test Requirements
  - I.2 Integration Test Requirements
  - I.3 Fork Testing Requirements
  - I.4 Launch Readiness Acceptance Criteria

- **Appendix J — Incident Response Runbooks**
  - J.1 Keeper Down
  - J.2 Oracle Outage / Staleness
  - J.3 DEX Outage
  - J.4 MEV Exploitation Detection
  - J.5 Admin Compromise Suspected
  - J.6 Emergency Pause Procedure
  - J.7 Reserve Drain / Redemption Bank Run
  - J.8 Contract Exploit Found
  - J.9 Communications Protocol
    
---

# 0. Executive Summary (Protocol Truth)

## 0.1 What the platform does (precise, non-marketing)

YieldLoop + VaultOS is a modular, non-custodial vault platform deployed on **BNB Chain (BSC)** that accepts **USDT (BEP-20)** deposits into smart contract vaults and uses a deterministic execution engine (“YieldLoopCore”) to generate **verified realized profit** using allowlisted strategy modules.

The platform provides two vault product types:

1) **YieldLoop Vaults (Cycle Vaults)**  
   - Users deposit USDT into a vault.  
   - The system generates a strategy proposal and presents it to the user.  
   - The user must explicitly accept the proposal prior to execution.  
   - The vault runs on a **fixed 30-day loop cycle**.  
   - Profit events, fees, and settlement are deterministic and ledger-based.  
   - At cycle end, the user selects a final action (compound / partial withdraw / full withdraw), consistent with platform rules.

2) **VaultOS Goal Vaults (Generationz Vault Type)**  
   - Users create labeled “goal buckets” as savings vaults (Xmas, vacation, school, house, engagement ring, legacy, etc.).  
   - Deposits are USDT only, with minimum deposit enforced.  
   - Vault lock terms range from **90 days to 20 years**, in **30-day increments**.  
   - Maturity withdrawal is penalty-free.  
   - Early exit is allowed only through a “Break Lock” action that closes the vault and applies a deterministic penalty schedule.  
   - VaultOS vaults include mandatory recovery and inheritance mechanisms so funds do not become permanently inaccessible.

All value movement and accounting is **proof-based and deterministic**:
- “profit” is only recognized when realized and verified
- fees are applied only to verified realized profit events
- no smoothing, no “estimated yield,” no hidden leverage

Users may select reward payout in:
- **USDT**, or
- **LOOP** (which applies a discounted performance fee)

The platform is strategy-modular: additional strategy plugins can be added later without rewriting the vault architecture.

## 0.2 What the platform does not do (explicit exclusions)

The platform does not:
- guarantee profit, yield, APY, or predictable performance
- provide insurance of deposits or profits
- promise any specific token price outcome
- trade assets outside the allowlist
- operate on non-EVM chains for MVP1
- use cross-chain bridges for MVP1 execution
- allow admin custody or withdrawal of user funds
- apply performance fees to principal (fees apply to verified realized profit only)
- allow partial exits on VaultOS goal vaults (Break Lock is full close)
- allow anytime principal withdrawal on time-locked VaultOS vaults

## 0.3 Locked design choices (chain, assets, venues, vault types)

**Chain:** BNB Chain (BSC) — Final  
**Deposit Asset (all vault types):** USDT (BEP-20) — Final  
**Minimum Deposit:** $100 USDT — Final  
**DEX Venues (MVP1):** PancakeSwap (PCS), BiSwap — Final  
**Trading Inventory Universe:** allowlisted set; BTCB is primary — Final (expandable post-MVP via governance-defined update process)  
**Vault Types:** YieldLoop Vaults + VaultOS Goal Vaults — Final  
**Reward Assets:** USDT payout OR LOOP payout — Final  
**Performance Fees:**  
- USDT payout: 20% of verified realized profit — Final  
- LOOP payout: 17.5% of verified realized profit — Final  
**Break Lock Penalty:** VaultOS-only, deterministic decay schedule — Final

## 0.4 Risk disclosures (mandatory)

This protocol operates in DeFi and inherits unavoidable risks including but not limited to:
- smart contract risk
- oracle manipulation risk
- DEX liquidity disruption and quote instability
- MEV/sandwich attacks and execution interference
- stablecoin depeg events (USDT risk)
- pegged-asset deviation risk (BTCB, ETH-peg, XRP-peg, SOL-peg)
- extreme volatility and market discontinuities
- keeper downtime / delayed execution
- chain congestion/outage (BNB Chain)

Users acknowledge:
- funds are at risk
- no insurance exists
- VaultOS vaults enforce maturity constraints
- early exit is penalized by design and is intentionally inconvenient

## 0.5 Module map (engine → vaults → strategies → token systems)

**Layer 1 — YieldLoopCore (shared execution engine)**
- VaultFactory / VaultRegistry
- ExecutionRouter
- GuardrailEngine
- OracleAdapter
- SettlementLedger + ClaimLedger
- FeeRouter + DiscountEngine

**Layer 2 — Vault products**
- YieldLoopVault (cycle-based wrapper)
- VaultOSGoalVault (lock/penalty/recovery wrapper)

**Layer 3 — Strategy plugins (MVP1)**
- Band/Grid strategy plugin
- PCS ↔ BiSwap arbitrage strategy plugin

**Layer 4 — Token systems**
- LOOP payout conversion module
- Redemption system module
- Reserve ledger + reserve custody module
- Floor support mechanisms module

## 0.6 Core enforcement guarantees (what is mechanically enforced)

The protocol enforces:
- no trade unless guardrails pass
- no fee unless verified realized profit occurs
- no admin withdrawal authority
- deterministic settlement math and ordering
- deterministic VaultOS penalty math and decrement schedule
- immutable maturity timestamps once set
- mandatory recovery and inheritance rules for VaultOS vaults


---

# 1. Definitions, Truth Sources, and Ledger Model (Hard Rules)

This section defines the vocabulary and non-negotiable accounting rules. Any part of the system that violates this section is considered an invalid implementation.

## 1.1 Glossary (core terms)

**Allowlist:** The list of permitted tokens, DEX venues, routers, and strategy modules. Anything not allowlisted is forbidden.  
**Arbitrage:** Buying and selling the same inventory across PCS and BiSwap to exploit a positive spread net of all costs.  
**Break Lock:** A VaultOS-only early exit action that closes the vault and applies the current penalty percentage to the withdrawal amount.  
**Bucket / Goal Vault:** A VaultOS labeled vault used as a savings subaccount (Xmas, vacation, school, etc.).  
**Claimable Balance:** The amount available for a user to withdraw subject to vault constraints (cycle rules or maturity rules).  
**Compounding:** Re-deploying a portion of net profit into the strategy inventory rather than making it claimable.  
**DiscountEngine:** The unified system that computes fee discounts based on Supporter/native discounts and LOOP payout selection.  
**Execution:** The act of performing onchain swaps/trades, including arbitrage trades, under strict guardrails.  
**FeeRouter:** Contract module that routes performance fees to configured destinations.  
**Floor (LOOP):** The protocol-defined price support mechanism for LOOP, backed by reserve and redemption rules.  
**Keeper:** An automated executor authorized to call specific functions to run strategies and settlement.  
**LOOP:** The platform token that can be selected as payout asset and unlocks a discounted performance fee path.  
**Maturity Timestamp:** The time at which a VaultOS goal vault becomes penalty-free to exit.  
**Oracle:** Onchain or offchain price truth reference used for sanity checks; never a standalone trade trigger without guards.  
**PenaltyEngine:** Contract module implementing break-lock penalty math (20% → 5% deterministic schedule).  
**Profit Event:** A verified event where realized profit is computed and committed to the ledger.  
**Realized Profit:** Profit recognized only after trade execution and valuation rules confirm net positive gain.  
**Reserve (Redemption Reserve):** The asset backing pool used to satisfy LOOP redemption mechanics and floor support.  
**Settlement:** The deterministic accounting update step after an execution or at a cycle boundary.  
**Strategy Plugin:** A module implementing an allowlisted strategy interface (band/grid, arbitrage, etc.).  
**Vault:** A contract-controlled container for user funds operating under a fixed vault type rule set.

## 1.2 Truth Sources Hierarchy (what the system trusts)

The system must adhere to this strict hierarchy of truth sources:

1) **Onchain vault balances** (token balances held by vault/engine contracts)  
2) **Deterministic internal ledgers** (SettlementLedger / ClaimLedger / ReserveLedger)  
3) **Onchain DEX executable quotes** (router quotes validated against slippage constraints)  
4) **Oracle sanity checks** (used to validate that DEX quotes and execution are not manipulated)  
5) **Offchain analytics / dashboards** (display-only; never authoritative)

No offchain service, UI, or server is a source of truth for balances, profit, or redemption eligibility.

## 1.3 Ledger Types (mandatory ledgers)

The platform must implement and maintain the following ledgers:

### 1.3.1 SettlementLedger
Tracks:
- principal
- strategy inventory valuation snapshots (USDT terms)
- realized profit events
- fees assessed per profit event
- net profit allocation (compound vs claimable)

### 1.3.2 ClaimLedger
Tracks:
- claimable USDT
- claimable LOOP
- claim history
- payout mode selection
- restrictions imposed by vault type rules

### 1.3.3 ReserveLedger (LOOP support)
Tracks:
- reserve funding inflows (from configured sources)
- reserve balances by token
- reserve usage for redemptions and floor support
- any throttling state applied

### 1.3.4 PenaltyLedger (VaultOS)
Tracks:
- lock start timestamp
- maturity timestamp
- penalty schedule parameters (Pmax=20%, Pmin=5%)
- penalty percent at current time

## 1.4 Profit Definition (strict realized-profit rule)

Profit is defined as:

> **Net positive change in USDT terms** after deducting all execution costs and applying valuation rules.

A profit event can only be committed when:
- the system can calculate valuation deterministically
- the net result is positive
- all guardrails and oracle sanity checks were satisfied for the executions contributing to that event

Profit cannot be defined by:
- unrealized mark-to-market estimates
- theoretical spreads
- offchain “backtested” results
- projected yield

## 1.5 Fee Applicability Rule (non-negotiable)

Performance fees:
- apply ONLY to verified realized profit events
- never apply to principal
- never apply to unrealized gains

Break-lock penalties:
- apply ONLY to VaultOS early exit
- never apply at maturity
- never discounted

## 1.6 Deterministic Settlement Rules

Settlement must:
- be atomic (either succeeds completely or fails completely)
- execute in a fixed order (no conditional fee sequencing)
- compute fees before compounding allocation
- compute discounts before fee routing
- emit standardized events for every profit event, fee assessment, payout conversion, and reserve movement

## 1.7 Precision, decimals, rounding, and dust

The system must:
- normalize token decimals to a standard internal precision
- define rounding direction rules (e.g., round down on user credits to avoid over-crediting)
- accumulate dust into an explicit dust ledger or donate to reserve (final decision: dust routes to reserve ledger)
- avoid precision drift across repeated settlements

## 1.8 Consent and Compliance UX Rules (mandatory enforcement)

The protocol enforces a user-consent model:

- A YieldLoop vault may not enter execution state unless the user has accepted the presented strategy proposal.
- Any material strategy change that impacts risk, asset targets, or withdrawal outcome requires re-consent (YieldLoop).
- VaultOS goal vaults must show lock + penalty disclosures at creation, and require explicit user confirmation.

Offchain UX requirements are not optional: UI must present these confirmations and must store verifiable consent receipts.


---

# 2. Architecture Overview (Full Stack)

This section defines the complete platform architecture and the boundaries between core engine, vault types, and strategy modules.

## 2.1 Layer Model (strict separation)

The platform is composed of:

### Layer A — Core Engine (YieldLoopCore)
The shared system responsible for:
- holding/controlling vault capital
- executing allowlisted strategy actions
- enforcing guardrails and sanity checks
- performing settlement and ledger updates
- routing fees and applying discounts

### Layer B — Vault Types (Product wrappers)
The vault type defines:
- what user actions are permitted
- what withdrawal constraints exist (cycle vs lock)
- what consent/approval gating is required
- what penalties apply (VaultOS only)

### Layer C — Strategy Plugins
Strategy plugins define:
- the decision logic for trade opportunities
- the specific trade sequences performed
- the expected verification signals for profit events
All strategy plugins must call guardrail/oracle checks and must produce standardized logs.

### Layer D — Token Systems
Token systems implement:
- LOOP payout conversion
- redemption and reserve mechanics
- floor support mechanisms

### Layer E — Offchain Services
Offchain services implement:
- keeper automation
- notification delivery (email)
- indexing, dashboards, analytics

Offchain services may not:
- custody funds
- override vault truth
- fabricate profit events
- override reserve ledger entries

## 2.2 Contract/Module Inventory (mandatory modules)

The onchain system consists of:

### 2.2.1 VaultFactory
Responsibilities:
- create vault instances of allowlisted types
- enforce minimum deposit rules
- bind immutable vault configuration (type, timestamps, principal ledger init)

### 2.2.2 VaultRegistry
Responsibilities:
- track all vaults
- map owner → vault list
- enforce vault type interface compliance

### 2.2.3 YieldLoopCore (ExecutionRouter)
Responsibilities:
- receive execution calls from keepers
- call strategy plugins
- enforce pre/post execution requirements
- ensure all routes are allowlisted

### 2.2.4 GuardrailEngine
Responsibilities:
- validate slippage bounds
- validate liquidity floors
- enforce trade sizing caps
- enforce cooldown windows
- enforce drawdown caps
- enforce max executions per window

### 2.2.5 OracleAdapter
Responsibilities:
- provide sanity pricing reference
- detect price manipulation conditions
- reject execution when DEX quotes deviate beyond tolerance

### 2.2.6 SettlementLedger + ClaimLedger
Responsibilities:
- record realized profit events
- compute fee/discount
- record compounding allocations
- update claimable balances by asset (USDT/LOOP)

### 2.2.7 FeeRouter + DiscountEngine
Responsibilities:
- compute fee tiers (20% USDT / 17.5% LOOP)
- apply supporter/native discount
- compute final fee
- route fee to configured wallets

### 2.2.8 VaultOSPenaltyEngine
Responsibilities:
- compute break-lock penalty % at any time
- enforce penalty on early exit
- route penalty amounts deterministically

### 2.2.9 RecoveryEngine + InheritanceEngine (VaultOS mandatory)
Responsibilities:
- enforce recovery methods:
  - recovery wallet
  - guardian threshold recovery
  - timelock recovery
- enforce beneficiary inheritance at maturity or after qualifying conditions
- prevent theft via cancellation rights and notification states

### 2.2.10 LOOP Systems (Reserve + Redemption + Floor)
Responsibilities:
- maintain ReserveLedger
- process redemptions under defined rules
- enforce floor support mechanism constraints

## 2.3 Keeper/Automation Services (mandatory behavior)

Keepers are offchain agents authorized to execute:
- market condition checks
- strategy runs
- settlement finalization calls
- periodic health checks

Keeper constraints:
- must be whitelisted
- must operate under rate limits
- cannot bypass guardrails
- cannot bypass oracle checks
- cannot withdraw user funds

If keepers are offline:
- the system defaults to inactivity
- no forced execution is permitted

## 2.4 Data Objects (schema-level overview)

### 2.4.1 Vault Object (required fields)
- vaultId
- ownerAddress
- vaultType
- principalUSDT
- depositTimestamp
- maturityTimestamp (VaultOS only)
- cycleStartTimestamp / cycleEndTimestamp (YieldLoop only)
- strategyInventoryBalances (token balances)
- claimableUSDT
- claimableLOOP
- payoutMode (USDT or LOOP)
- compoundSelection
- state

### 2.4.2 ProfitEvent Object
- eventId
- vaultId
- timestamp
- realizedProfitUSDT
- feeRateApplied
- feeAmountUSDT
- netProfitUSDT
- compoundedUSDT
- claimableIncreaseUSDT
- claimableIncreaseLOOP (if payoutMode LOOP)

### 2.4.3 ReserveLedger Entry
- timestamp
- source
- amount
- token
- resultingReserveBalance

### 2.4.4 Penalty Snapshot
- vaultId
- timestamp
- penaltyPercent
- monthsElapsed
- monthsTotal
- maturityTimestamp

## 2.5 Global Protocol State Machine

States:
- NORMAL (execution enabled)
- DEGRADED (execution limited; increased guardrails)
- PAUSED (no trading; withdrawals still allowed)
- EMERGENCY (restricted set of admin actions; withdrawals still allowed)

Global state transitions:
- NORMAL → DEGRADED: triggered by defined risk conditions
- DEGRADED → PAUSED: triggered by repeated failures/manipulation detection
- PAUSED → NORMAL: only after explicit unpause condition and time delay

## 2.6 Vault State Machine (per vault)

### YieldLoop Vault States
- CREATED
- PROPOSED (strategy offered)
- ACCEPTED (user accepted)
- ACTIVE_CYCLE
- SETTLING
- CLOSED

### VaultOS Goal Vault States
- CREATED
- ACTIVE_LOCKED
- MATURITY_READY
- BREAK_LOCK_EXITING
- RECOVERY_PENDING
- CLOSED

State enforcement:
- vault type determines allowed transitions
- forbidden transitions revert

## 2.7 Strategy Execution State Machine (per run)

States:
- ELIGIBILITY_CHECK
- PRECHECKS_PASSED
- EXECUTING
- VERIFYING
- SETTLING
- COMPLETED
- FAILED_SAFE (no state change except log)

Rules:
- any guardrail/oracle failure must exit to FAILED_SAFE
- FAILED_SAFE must not mutate user balances

## 2.8 Failure Propagation Model (safe defaults)

All failures must default to “do nothing safely” rather than “force action”:

- oracle failure → no trade
- DEX quote instability → no trade
- slippage exceeds bound → no trade
- keeper downtime → no trade
- gas spike → no trade
- reserve shortage → redemption throttles or pauses per rules; vault trading unaffected unless explicitly defined

Withdrawals remain available according to vault rules even when trading is paused.

## 2.9 Transparency Logging Requirements

Every critical action must emit standardized events:

- VaultCreated
- StrategyProposed
- StrategyAccepted
- StrategyRunStarted
- StrategyRunFailedSafe
- ProfitEventCommitted
- FeeAssessed
- FeeRouted
- PayoutModeSelected
- LOOPSwapExecuted
- LOOPSwapFailedFallback
- ClaimableUpdated
- VaultMatured
- BreakLockExecuted
- PenaltyAssessed
- PenaltyRouted
- RecoveryInitiated
- RecoveryCancelled
- RecoveryExecuted
- RedemptionRequested
- RedemptionSettled
- ReserveUpdated
- FloorSupportAction (if triggered)

No event = no trust.

---

# 3. YieldLoopCore Execution Engine (Shared System)

This section defines the deterministic execution engine used by **all vault types**. If the dev implements only one thing correctly, it must be this engine and its invariants. All vault products are wrappers around this truth layer.

## 3.1 Engine invariants and enforcement rules (non-negotiable)

The YieldLoopCore engine MUST enforce the following invariants:

1) **Non-custodial control**
   - Admin has no authority to withdraw user funds.
   - No function may exist that can sweep, transfer, seize, or redirect user principal.

2) **Profit-based accounting only**
   - The system MUST NOT manufacture yield.
   - Fees MUST only be taken on verified realized profit events.

3) **Deterministic settlement**
   - ProfitEvent math is deterministic.
   - Fee math is deterministic.
   - Discount application order is deterministic.
   - Ledger updates are deterministic and atomic.

4) **Guardrail-first execution**
   - The default behavior is “do nothing safely.”
   - If any safety check fails, execution MUST revert or exit to FAILED_SAFE with no state mutation.

5) **Venue restriction**
   - MVP1 execution is restricted to:
     - PancakeSwap (PCS)
     - BiSwap
   - Any venue outside the allowlist MUST revert.

6) **No hidden leverage**
   - The engine MUST NOT borrow.
   - The engine MUST NOT open margin/leveraged positions.
   - The engine MUST NOT create debt-bearing states.

7) **Mandatory transparency**
   - Every execution attempt, failure, settlement, fee, reserve movement, redemption, and penalty MUST be logged via events.

## 3.2 Permission model (who can execute what)

The engine exposes separate roles:

### 3.2.1 User role
User can call:
- Deposit actions (vault creation and top-ups)
- Configuration actions allowed by vault type
- Claim / withdraw actions allowed by vault type
- YieldLoop strategy acceptance actions (cycle vaults only)
- VaultOS recovery setup actions (goal vaults only)

User cannot call:
- direct trade execution functions

### 3.2.2 Keeper role
Keepers are allowlisted executor addresses. Keepers can call:
- `runStrategy(vaultId, strategyId, params)`
- `settle(vaultId)` (if separated)
- `healthCheck()` / `updateState()` (optional)

Keeper cannot:
- withdraw
- change immutable config
- bypass guardrails
- bypass oracle checks
- change payout mode if vault rules forbid changes

Keeper calls MUST always pass through GuardrailEngine and OracleAdapter.

### 3.2.3 Admin role
Admin can call:
- add/remove keeper addresses (timelocked)
- pause/unpause trading (timelocked)
- update allowlist parameters (timelocked)
- update fee routing wallets (timelocked)
- update strategy parameters (timelocked)

Admin cannot:
- change user principal
- change claimable balances
- change maturity dates
- trigger break-lock withdrawals
- override oracle checks
- force a trade that would otherwise be rejected

All admin actions MUST be logged and timelocked.

## 3.3 VaultFactory/VaultRegistry mechanics

### 3.3.1 VaultFactory responsibilities
VaultFactory is the only creation entrypoint for new vaults and must enforce:

- Minimum deposit: **$100 USDT**
- Deposit asset: **USDT only**
- Vault type must be allowlisted:
  - YieldLoop vault
  - VaultOS goal vault
- Required configuration:
  - YieldLoop: must initialize cycle boundary and set “strategy acceptance required”
  - VaultOS: must set maturity date and required recovery + beneficiary configuration

### 3.3.2 VaultRegistry responsibilities
VaultRegistry stores:
- vaultId → vault contract address
- vaultId → owner
- vaultId → vault type
- owner → vault list
- vault status

No vault should exist without being in the Registry.

## 3.4 Capital lifecycle mechanics (USDT → inventory → settle)

This system uses a USDT-denominated truth ledger.

### 3.4.1 Deposit stage
User deposits USDT into a vault. Vault records:
- `principalUSDT`
- `depositTimestamp`
- vault type configuration snapshot

### 3.4.2 Inventory stage
During execution the vault may hold inventory such as:
- BTCB
- BNB
- ETH (peg)
- XRP (peg)
- SOL (peg)

Inventory holdings are permitted only if:
- token is allowlisted
- conversion was executed through allowlisted router paths
- guardrails approved

### 3.4.3 Settlement stage
Settlement converts execution results into ledger truth:
- realizedProfitUSDT is computed
- fees and discounts applied
- profit allocated into:
  - compounded amount
  - claimable amount

At no point is “profit” declared without settlement.

## 3.5 Trade eligibility gating (mandatory preconditions)

Every strategy run MUST satisfy:

1) Vault is in an executable state
2) Vault type allows execution at this time
3) Vault is not paused
4) Global protocol state is not paused
5) Strategy is allowlisted
6) Required oracle feeds available and sane
7) DEX quotes are within oracle sanity window
8) Liquidity minimums satisfied
9) Slippage bounds satisfied
10) Trade sizing caps satisfied
11) Cooldowns satisfied
12) Drawdown caps not exceeded
13) Gas cap acceptable (if implemented as guardrail)

If any condition fails:
- Do not trade
- Emit failure-safe log
- No state mutation

## 3.6 Trade execution lifecycle (step-by-step)

All strategy execution must follow a standardized lifecycle.

### 3.6.1 Phase 1: Eligibility Check
- validate vault state
- validate strategy allowlist
- validate keeper permissions

### 3.6.2 Phase 2: Quote & Route Selection
- fetch DEX quotes from PCS + BiSwap routers
- compute expected output amounts
- compute effective price
- compute expected slippage
- compute net spread (for arbitrage)

### 3.6.3 Phase 3: Oracle Sanity Validation
- oracle price must be available
- DEX price must not deviate beyond tolerance
- if tolerance exceeded → fail safe

### 3.6.4 Phase 4: Guardrail Enforcement
- max slippage check
- max trade size check
- liquidity floor check
- cooldown check
- drawdown check

### 3.6.5 Phase 5: Execute
- execute swaps/trade sequence
- use allowlisted routers only
- enforce minimum out (slippage protection)
- enforce deadline

### 3.6.6 Phase 6: Verify
After execution:
- verify outputs received match expectations within tolerance
- verify invariant: no forbidden token holds
- verify inventory valuations sanity-check

### 3.6.7 Phase 7: Settle
- record trade outcome
- compute profit delta (realized-only)
- create ProfitEvent if profit > 0
- apply fees + discounts
- update claim ledger
- update reserve ledger if applicable

## 3.7 Settlement lifecycle (step-by-step)

Settlement math is mandatory and fixed order:

1) Determine **realizedProfitUSDT**
2) If realizedProfitUSDT <= 0:
   - no fee
   - record event as non-profitable (optional) OR record nothing (final policy: record non-profitable event log but no ProfitEvent entry)
3) Determine payout mode:
   - USDT payout OR LOOP payout
4) Determine performance fee rate:
   - 20% (USDT payout)
   - 17.5% (LOOP payout)
5) Apply Supporter/native discount via DiscountEngine
6) Compute fee:
   - `feeAmount = realizedProfitUSDT * finalFeeRate`
7) Compute net profit:
   - `netProfit = realizedProfitUSDT - feeAmount`
8) Allocate net profit:
   - `compoundAmount = netProfit * compoundSelection`
   - `claimableAmount = netProfit - compoundAmount`
9) If payoutMode = LOOP:
   - swap `claimableAmount` USDT to LOOP using allowlisted route
   - enforce slippage/oracle checks
   - if swap fails: fallback to USDT claimable
10) Update ledgers:
   - SettlementLedger entry
   - ClaimLedger entry
   - emit standardized events

Settlement must be atomic.

## 3.8 GuardrailEngine mechanics (slippage, liquidity, drawdown, size caps, cooldowns)

GuardrailEngine is mandatory and must provide:

### 3.8.1 Slippage cap
- global max slippage
- per-token slippage overrides
- per-strategy slippage overrides
Default is conservative.

### 3.8.2 Liquidity floor
- minimum liquidity on venue for pair
- minimum expected depth relative to trade size

### 3.8.3 Trade sizing caps
- max % of vault balance per trade
- max absolute USDT size per trade
- max daily turnover % per vault

### 3.8.4 Cooldowns
- minimum time between trades per vault
- minimum time between repeated strategy runs
- prevents overtrading and MEV vulnerability

### 3.8.5 Drawdown caps
- max drawdown threshold
- max consecutive loss tolerance
- breach triggers:
  - DEGRADED or PAUSED state per vault
  - trading suspension

All caps must be enforceable onchain.

## 3.9 OracleAdapter mechanics (sanity checks + manipulation defense)

OracleAdapter exists to prevent trade execution under manipulated DEX prices.

Rules:
- Oracle values do not trigger trades
- Oracle values validate trade safety

Oracle sanity checks:
- DEX quote price within a tolerance band of oracle
- maximum deviation threshold set per token
- if oracle unavailable:
  - fail safe → no trade

## 3.10 MEV protection model (mandatory)

The system MUST implement MEV mitigation. MVP1 must include:

- maximum slippage protection (minOut)
- randomized execution timing (keeper jitter)
- cooldown window enforcement
- refusal logic when volatility spikes
- optionally: private transaction submission (if available)

If MEV defense cannot be guaranteed in a given context:
- trade must be refused

## 3.11 Venue routing rules (PCS/BiSwap only)

Routing policy:
- only allow swaps through PCS and BiSwap routers
- only allow allowlisted pairs
- no multi-hop routes through unknown tokens unless allowlisted explicitly
- any router/pair outside allowlist must revert

## 3.12 Refusal/no-trade rules (mandatory)

Trade is refused if:
- oracle unavailable
- DEX quote deviates beyond tolerance
- slippage > cap
- liquidity below floor
- cooldown not expired
- drawdown breached
- venue unhealthy
- gas exceeds cap
- keeper not allowlisted
- vault not in executable state
- strategy not allowlisted

Refusal must emit:
- reason code
- vaultId
- strategyId
- relevant numeric values

## 3.13 Keeper job definitions (exact jobs, triggers, schedules)

Keepers must perform these jobs:

1) **Market scan job**
   - identifies opportunities for allowlisted strategies
   - produces candidate params

2) **Eligibility job**
   - validates vault is eligible to execute

3) **Execution job**
   - calls `runStrategy()`

4) **Settlement job**
   - calls settle if separated (or integrated)

5) **Health job**
   - monitors failures
   - triggers degrade/pause thresholds

Keeper schedule:
- frequent scanning
- rare execution (conservative)

## 3.14 Pause system (what pause affects; withdrawals always preserved)

Pause affects:
- trade execution
- strategy runs
- swaps for LOOP payout (if dangerous)

Pause does NOT affect:
- user withdrawals (within vault rules)
- maturity withdrawals
- break-lock exits
- redemption requests (unless reserve rules specify throttle/pause)

Pauses must be granular:
- global pause
- strategy pause
- venue pause
- per-vault pause

## 3.15 Fallbacks (keeper down, oracle down, DEX degraded, gas spikes)

Fallback principle: **safe inactivity**.

- keeper down → no trade
- oracle down → no trade
- DEX degraded → no trade
- gas spike → no trade
- repeated failures → auto-degrade or pause

Withdrawals remain available per rules.


---

# 4. Vault Interface Layer (Unified Vault Abstraction)

All vault products must implement a shared “vault interface” so the engine can treat vault products consistently while still enforcing vault-specific constraints.

## 4.1 Unified vault interface (must be implemented by all vault types)

Each vault must expose:

- `getVaultType()`
- `getOwner()`
- `getState()`
- `getDepositAsset()` (always USDT)
- `getInventoryAllowlist()`
- `getPayoutMode()` (USDT or LOOP)
- `getCompoundSelection()`
- `isExecutionAllowedNow()`
- `isWithdrawalAllowedNow()`
- `isMaturityReached()` (VaultOS only)
- `getPenaltyPercentNow()` (VaultOS only)
- `getCycleInfo()` (YieldLoop only)

## 4.2 Shared vault fields and ledgers

All vaults must include:

### Shared immutable fields
- ownerAddress
- vaultType
- depositTimestamp
- principalUSDT
- vaultId

### Shared mutable fields
- payoutMode (USDT/LOOP) (rules for when change allowed are vault-specific)
- compoundSelection (rules for when change allowed are vault-specific)

### Shared ledgers
- SettlementLedger
- ClaimLedger

## 4.3 Immutable vs mutable configuration rules

### Immutable (final)
- vaultType
- depositTimestamp
- principal ledger initialization snapshot
- (VaultOS) maturityTimestamp
- (YieldLoop) cycle duration rules

### Mutable (final)
- payoutMode selection (allowed at specific checkpoints only)
- compoundSelection (allowed at specific checkpoints only)
- notification settings (offchain)

No mutable field may alter:
- principal ownership
- maturity timestamp
- penalties
- discount history integrity

## 4.4 Allowed user actions (platform-wide)

Users are permitted to:
- create vault
- top-up deposits (VaultOS only; YieldLoop top-ups only by creating new vault deposit)
- select payout mode (if allowed by vault type)
- select compound setting (if allowed by vault type)
- withdraw/claim when allowed
- execute break-lock (VaultOS only)
- initiate recovery (VaultOS only)
- update recovery contacts only under strict rules

## 4.5 Claim model (ledger truth + computation)

Claim balances are ledger entries and must never be inferred from “current vault balance” alone.

ClaimLedger maintains:
- claimableUSDT
- claimableLOOP

Claim rules:
- YieldLoop: claimable updated at cycle settlement
- VaultOS: claimable updated at profit event settlement but may be locked until maturity (vault rule)

Claim ledger must include:
- source ProfitEventId
- timestamp
- payout mode used
- conversion route used (if LOOP)

## 4.6 Withdrawal model (platform-wide rules + vault-specific constraints)

Withdrawal is split into:
- **claim** (collect claimable balance)
- **close** (close vault permanently)

Rules:
- all withdrawals are initiated by user
- engine never forces a withdrawal
- vault type defines availability

YieldLoop withdrawals:
- occur at cycle end settlement checkpoints

VaultOS withdrawals:
- maturity withdraw: penalty-free, closes vault
- break-lock withdraw: penalty applied, closes vault

## 4.7 Notification model (mandatory events and triggers)

Notifications are mandatory for:
- vault created
- strategy proposed
- strategy accepted
- profit event committed
- fees assessed
- cycle end reached
- vault maturity reached
- break-lock initiated
- recovery initiated / canceled / executed
- redemption requested / settled

Notifications are delivered:
- in-app
- email

## 4.8 Transparency model (profit proofs, fees, reserves, floor metrics)

Platform must provide:
- profit events list per vault
- fees taken per event
- discounts applied
- claimable balances
- reserve changes
- redemption activity
- floor support actions

This must be accessible in UI and via events.


---

# 5. YieldLoop Vaults (Cycle-Based Vault Mechanics — Final)

YieldLoop vaults are cycle-based products that enforce:
- strategy acceptance gate
- fixed 30-day loop duration
- deterministic settlement at cycle end
- user choice at end of cycle

## 5.1 Product rules and user expectations

YieldLoop vault is a 30-day commitment product:
- user deposits USDT
- user must accept strategy
- vault executes within constraints
- user cannot make mid-cycle changes that would break determinism
- settlement occurs at cycle end

## 5.2 Deposit mechanics

Deposit rules:
- USDT only
- minimum $100
- vault created via VaultFactory
- principal recorded immediately
- cycle start timestamp set at deposit acceptance

Deposit receipts:
- vaultId issued
- deposit event logged

## 5.3 Strategy proposal generation mechanics (mandatory)

Before execution begins, the system must produce a Strategy Proposal containing:

- target inventory tokens (subset of allowlist)
- risk tier selection
- execution cadence expectations (max trades/day, cooldown, etc.)
- slippage caps
- drawdown caps
- profit fee schedule (20% USDT / 17.5% LOOP)
- payout mode default
- compounding default
- all major risk disclosures

Proposal must be:
- shown in UI
- confirmed via explicit acceptance

## 5.4 Strategy acceptance mechanics (mandatory)

User acceptance requires:
- wallet signature confirming strategy proposal fields
- agreement to risk disclosures
- acknowledgment that outcomes are not guaranteed

Acceptance creates:
- StrategyAccepted receipt
- vault enters executable state

No keeper execution is allowed before acceptance.

## 5.5 30-day loop cycle mechanics (mandatory)

### 5.5.1 Cycle start rules
Cycle starts when:
- user deposits
- strategy proposal accepted
- cycleStartTimestamp recorded

### 5.5.2 In-cycle constraints
During cycle:
- user cannot withdraw principal
- user cannot change risk tier
- user cannot change strategy plugin set
- user can only change:
  - end-of-cycle selection (compound/withdraw option)
  - notification preferences

### 5.5.3 Cycle end rules
Cycle ends at:
- `cycleStartTimestamp + 30 days`

At cycle end:
- trading stops
- settlement begins
- results are finalized

## 5.6 Execution mechanics within cycle

Within cycle the engine may:
- trade allowlisted inventory
- execute arbitrage when spread net positive under guardrails
- refuse trading during unstable conditions

The engine must:
- obey all guardrails
- obey cooldown windows
- obey trade sizing caps
- record every attempt and result

## 5.7 Profit event mechanics within cycle

Profit events can occur during the cycle but are not “claimable” until cycle settlement checkpoint.

Profit events:
- must be verified realized profit
- must apply fee/discount
- must allocate to compound or claim ledger as per cycle rules

## 5.8 Cycle settlement mechanics (exact math)

At cycle end, settlement must compute:

- total realized profit for the cycle
- all fees already assessed on profit events
- net profit remaining
- final compounding allocation
- claimable balances

Settlement outputs:
- ClaimLedger updated
- cycle completed
- vault outcome state depends on user selection

## 5.9 Cycle-end user choice mechanics (compound/withdraw options)

At cycle end user selects one of:

1) Compound all
2) Compound 50% / withdraw 50%
3) Withdraw all

Rules:
- choice can be changed only before cycle end cutoff
- after cycle end begins settlement, choice is locked
- withdrawal applies to claimable only, never principal mid-cycle

## 5.10 YieldLoop vault fallbacks and edge cases

Edge cases:
- keeper down → fewer trades, no forced execution
- oracle down → no trade
- DEX down → no trade
- gas spike → no trade
- manipulation detected → pause

Cycle settlement must still proceed using:
- actual onchain balances
- deterministic ledger math
- “no trade” fallback if swaps required for payout conversion fail

YieldLoop vault must never strand funds.

---

# 6. VaultOS Goal Vaults (Generationz Vault Type — Final)

VaultOS goal vaults are purpose-labeled savings vaults with enforced lock terms. They use the same YieldLoopCore engine for execution and profit settlement, but impose additional constraints:
- fixed maturity date
- early exit penalty (break-lock)
- mandatory recovery and inheritance systems
- conservative execution constraints

## 6.1 Goal bucket model mechanics (subaccounts with labels)

A VaultOS goal vault is created as a “bucket” with immutable identity metadata at creation:

### 6.1.1 Required bucket fields (immutable)
- bucketName (string, e.g., “Christmas 2027”, “Tuition Fund”, “Ring Fund”)
- bucketCategory (enum, e.g., SAVINGS / GIFT / EDUCATION / HOUSING / LEGACY / PROJECT)
- bucketId (unique per vault; vaultId may serve)
- ownerAddress
- createdTimestamp

### 6.1.2 Optional display fields (mutable, UI-only)
These do not affect mechanics and are stored offchain or as optional onchain metadata:
- targetAmountUSDT (display only; not enforceable)
- targetDate (display only; not enforceable)
- notes

### 6.1.3 Multiple buckets per user
- A user may create unlimited VaultOS goal vaults (subject to gas and UI constraints).
- Each bucket is mechanically independent.

## 6.2 Deposit mechanics (USDT only; top-ups allowed)

### 6.2.1 Deposit asset and minimum
- Deposit asset: USDT (BEP-20) only.
- Minimum deposit: $100 USDT.

### 6.2.2 Initial deposit creates the vault
- Vault created via VaultFactory.
- VaultRegistry binds owner and vaultType.
- principalUSDT recorded in SettlementLedger.

### 6.2.3 Top-up deposits (allowed)
- Users may top-up a VaultOS goal vault at any time during lock.
- Top-ups:
  - increase principalUSDT
  - do not change maturity timestamp
  - do not reset lock start
  - do not alter penalty schedule parameters
- Each top-up emits:
  - VaultTopUp(vaultId, amountUSDT, timestamp)

### 6.2.4 Top-up accounting rule (strict)
- principalUSDT is the sum of all deposits.
- profit events are computed against the evolving principal and inventory, but **fees still apply only to realized profit**, not principal.

## 6.3 Lock schedule mechanics (final)

### 6.3.1 Lock range and increments
- Lock range: 90 days to 20 years.
- Lock increments: 30-day increments (i.e., 90d, 120d, 150d, … up to 20y).

### 6.3.2 Lock start timestamp
- lockStartTimestamp = timestamp of initial vault creation (first deposit completion).
- Top-ups do not alter lockStartTimestamp.

### 6.3.3 Maturity timestamp computation
- maturityTimestamp = lockStartTimestamp + lockTermDays
- lockTermDays is selected at creation and immutable.

### 6.3.4 Maturity enforcement
- Before maturityTimestamp: maturity withdrawal is forbidden.
- At or after maturityTimestamp: maturity withdrawal is permitted and penalty-free.

### 6.3.5 Immutability
The following are immutable after vault creation:
- lockTermDays
- lockStartTimestamp
- maturityTimestamp
- penalty schedule endpoints (Pmax=20%, Pmin=5%)
- penalty decrement cadence (monthly)

## 6.4 VaultOS execution constraints (conservative baseline)

VaultOS vaults are conservative by design. Their execution policy is locked:

### 6.4.1 Allowed strategies (final list)
VaultOS may run only the following strategy plugins:
- Band/Grid strategy plugin (conservative parameters)
- PCS ↔ BiSwap arbitrage plugin (only when net positive spread passes strict gates)

### 6.4.2 Disallowed strategies (final list)
VaultOS vaults must never run:
- index strategies
- yield farming strategies
- lending strategies
- sniping / kill-zone strategies
- any high-volatility or “unrestricted” strategies
- any strategy that requires cross-chain bridging

### 6.4.3 Guardrail tightening (VaultOS-specific)
VaultOS vaults must use stricter guardrails than YieldLoop vaults:
- lower max slippage
- lower max trade size
- higher liquidity floor
- longer cooldown windows
- tighter oracle deviation tolerance
- stricter drawdown cap and faster pause trigger

### 6.4.4 No-trade default
If conditions are not favorable, VaultOS vaults must do nothing. Safety is prioritized over opportunity.

## 6.5 Withdrawal mechanics (final)

### 6.5.1 Maturity withdrawal (penalty-free; closes vault)
At or after maturityTimestamp:
- user may execute `withdrawAtMaturity(vaultId)`
- vault closes
- user receives claimable balance + principal + net gains
- no break-lock penalty applies

### 6.5.2 Early withdrawal: Break Lock (penalty applies; closes vault)
Before maturityTimestamp:
- the only exit is `breakLockWithdraw(vaultId)`
- breakLock closes vault immediately
- penalty is assessed deterministically (see 6.6)
- penalty is routed (see 6.6.4)
- remainder paid out per payout mode selection (USDT or LOOP)

### 6.5.3 Partial withdrawal policy (final)
Partial withdrawals are NOT allowed for VaultOS goal vaults.
- Maturity withdrawal = full close
- Break Lock withdrawal = full close

## 6.6 Break-Lock Penalty Engine (final mechanics)

Penalty is applied only to early exit. It is intentionally punitive and inconvenient.

### 6.6.1 Penalty schedule endpoints
- Maximum penalty at start: 20%
- Minimum penalty near maturity: 5%
- Penalty at maturity: 0% (because Break Lock is not permitted at maturity; maturity exit is penalty-free)

### 6.6.2 Monthly decrement logic (deterministic)
Penalty is computed monthly as time elapses:

Definitions:
- `T0` = lockStartTimestamp
- `Tm` = maturityTimestamp
- `monthsTotal` = floor((Tm - T0) / 30 days)
- `monthsElapsed` = floor((now - T0) / 30 days), bounded [0, monthsTotal]

Penalty percent:
- If now >= Tm: penaltyPercent = 0 (but breakLock path is disabled; use maturity withdraw)
- Else:
  - linear decay from 20% to 5% across monthsTotal:
  - `penaltyPercent = 0.20 - (monthsElapsed * (0.20 - 0.05) / monthsTotal)`
  - bounded so it never drops below 0.05 before maturity

### 6.6.3 Penalty preview requirements (mandatory UI + onchain view)
- Vault must expose view method:
  - `getPenaltyPercentNow(vaultId)`
  - `getPenaltyAmountNow(vaultId, expectedWithdrawAmount)`
- UI must display:
  - current penalty %
  - penalty amount in USDT terms
  - remaining time to maturity
  - explicit warning: “This permanently closes your vault.”

### 6.6.4 Penalty routing mechanics (final)
Penalty is routed deterministically via PenaltyRouter:
- A fixed percentage to Reserve (supports LOOP system)
- A fixed percentage to Platform Ops (dev/audit/ops)
- A fixed percentage to a public pool (optional in earlier drafts; final policy here: NO public pool — route only to Reserve + Ops for simplicity and audit clarity)

Final routing:
- 50% penalty → Redemption Reserve
- 50% penalty → Ops/Dev/Audit/Marketing routing wallet set
All routing is onchain, timelocked configurable destinations, and logged.

### 6.6.5 Anti-gaming rules (final)
- Penalty percent uses floor-months; users cannot “time” sub-day differences to dodge penalty.
- Top-ups do not reset lock.
- Break Lock is always full close (no partial extraction).

## 6.7 Recovery system (mandatory; final)

VaultOS vaults must include a recovery system that prevents permanent loss of funds due to:
- lost phone
- lost seed phrase
- death
- incapacitation

Recovery is implemented as a mandatory triple-path system.

### 6.7.1 Unlock code model (final)
At vault creation, the system generates a **Withdrawal Unlock Code**:

- Code is user-facing only.
- The code is never stored in plaintext onchain.
- Onchain stores only:
  - `unlockCodeHash = keccak256(code + salt)`
- Code functions:
  - used at maturity withdrawal time OR for recovery initiation
  - acts as an additional factor beyond wallet control

If the user loses the code, recovery paths below still function, but with longer delays and higher friction.

### 6.7.2 Recovery wallet (mandatory)
User must nominate a recovery wallet at creation:
- recoveryWalletAddress (immutable after a short change window; see 6.7.5)
- recoveryWallet can initiate recovery with timelock:
  - `initiateRecovery(vaultId)`
- recovery has minimum delay:
  - 30 days
- during delay, owner can cancel:
  - `cancelRecovery(vaultId)`

### 6.7.3 Guardian threshold recovery (mandatory)
User must nominate guardians (minimum 3) at creation:
- guardianAddresses[3..7] (final allowed range)
- threshold = 2-of-3 minimum; can be 3-of-5 etc at creation
- guardians may jointly initiate recovery with timelock:
  - `initiateGuardianRecovery(vaultId, guardianSignatures)`
- delay:
  - 30 days
- owner can cancel during delay

### 6.7.4 Timelock recovery (mandatory)
If both recovery wallet and guardians are unavailable:
- a “deadman” timelock recovery exists:
  - `initiateTimelockRecovery(vaultId)`
- delay:
  - 180 days
- requires unlock code hash match OR public notarized proof mechanism (final policy: unlock code hash match required; no offchain proofs)

### 6.7.5 Recovery cancellation rules (mandatory theft defense)
- Owner can cancel any pending recovery during delay using their wallet.
- If owner wallet is compromised, guardians can cancel if majority threshold is met (same threshold as recovery).
- Any recovery initiation must trigger notifications to:
  - owner
  - recovery wallet
  - guardians

### 6.7.6 Beneficiary/inheritance rules (mandatory)
At vault creation, user must specify a beneficiary:
- beneficiaryAddress (immutable)
Inheritance rules:
- If a recovery path completes AND owner does not cancel within delay, the vault ownership transfers to beneficiary OR the beneficiary becomes withdrawal recipient at maturity (final: beneficiary becomes withdrawal recipient only after a completed recovery; normal ownership does not auto-transfer).

## 6.8 Goal completion + gift mechanics (mandatory)

### 6.8.1 Gift vault transfer rules (final)
VaultOS goal vault ownership transfer is NOT allowed mid-lock.
Reason: prevents theft and reduces compliance ambiguity.

### 6.8.2 Gift claim rules (final)
Gifting occurs by:
- user providing the withdrawal unlock code + vaultId + dapp link to recipient
- at maturity, recipient (beneficiary) can withdraw only if:
  - beneficiaryAddress matches
  - unlock code verifies OR beneficiary uses completed recovery path

### 6.8.3 Safe transfer restrictions (final)
- No NFT-style transferable vault token in MVP1.
- No marketplace.
- No secondary transfers.

VaultOS is a savings system, not a tradable product.


---

# 7. Strategy Framework (Plugin Interface — Final)

Strategy plugins allow the engine to support multiple strategies without rewriting vault logic. Every strategy must conform to a strict interface and safety contract.

## 7.1 Strategy interface (required functions and logs)

Each strategy plugin must implement:

- `getStrategyId()`
- `getStrategyName()`
- `getSupportedVaultTypes()` (YieldLoop, VaultOS)
- `getSupportedTokens()` (subset of token allowlist)
- `getDefaultParams(vaultType)`
- `buildTradePlan(vaultState, marketData) -> tradePlan`
- `validateTradePlan(tradePlan, guardrails, oracle) -> pass/fail + reason`
- `executeTradePlan(tradePlan) -> executionResult`
- `postVerify(executionResult, oracle) -> pass/fail + reason`
- `computeRealizedDelta(executionResult) -> realizedProfitUSDT`

Logging requirements:
- StrategyRunStarted
- StrategyRunFailedSafe(reasonCode)
- StrategyRunCompleted(realizedDelta)

## 7.2 Strategy input/output schema

### 7.2.1 Inputs
- vaultId
- vaultType
- payoutMode
- compoundSelection
- inventory balances
- vault constraints (cycle or lock)
- guardrail parameters
- oracle parameters
- DEX quote snapshots
- current protocol state (NORMAL/DEGRADED/PAUSED)

### 7.2.2 Outputs
- tradePlan (ordered list of swaps with exact routers)
- minOut values per swap
- deadlines per swap
- expected net delta in USDT terms
- risk tags (for logging only)

## 7.3 Strategy guardrail enforcement requirements

Every strategy must:
- call GuardrailEngine precheck
- call OracleAdapter sanity check
- enforce minOut and deadline
- respect venue allowlist
- refuse execution if any check fails

No strategy is permitted to bypass GuardrailEngine.

## 7.4 Strategy allowlist update process (timelocked)

- New strategies can only be added by admin via timelocked governance action.
- Strategy must be audited before allowlisting.
- Activation requires:
  - StrategyAdded event
  - ActivationDelay elapsed
  - StrategyActivated event

## 7.5 Strategy set (MVP1 final)

MVP1 includes exactly two strategies:

### 7.5.1 Band/Grid strategy mechanics (final)
Purpose:
- accumulate gains from price oscillation within a defined band under conservative constraints

Mechanics:
- define a price band for each token in USDT terms
- place staggered buy levels below current price
- place staggered sell levels above current price
- maintain inventory balance limits
- never exceed trade sizing caps

Rules:
- no martingale doubling
- no leverage
- no band widening mid-run without explicit re-consent (YieldLoop only)
- VaultOS uses tighter band and fewer levels

### 7.5.2 PCS ↔ BiSwap arbitrage strategy mechanics (final)
Purpose:
- capture positive spread between PCS and BiSwap net of all costs

Mechanics:
- fetch executable quotes on both venues
- compute net spread after:
  - swap fees
  - expected slippage
  - gas estimate
- execute only when net spread exceeds minimum threshold
- execute as atomic sequence where possible; otherwise fail safe

Rules:
- minimum spread threshold is a guardrail parameter
- no trade if liquidity below floor
- no trade if oracle deviation is high

## 7.6 Strategy execution limits (final defaults)

Defaults apply platform-wide, with VaultOS using stricter values.

Limits include:
- max slippage %
- max trade size (% of vault)
- max absolute trade size (USDT)
- cooldown seconds between trades
- max trades per day
- max drawdown %

All default values are defined in Appendix D and are not optional.

## 7.7 Strategy failure behavior and fallbacks

Failure handling is standardized:
- any failure → FAILED_SAFE
- no state mutation except logs
- repeated failures trigger vault-level DEGRADED state or PAUSED state per thresholds
- DEX outage → no trade
- oracle outage → no trade
- gas spike → no trade


---

# 8. Profit Events, Settlement, Claims, and Compounding (Unified)

This section defines the unified accounting truth that applies to all vault types.

## 8.1 Profit event definition (strict)

A ProfitEvent is committed only when:
- execution has completed
- post-verification passed
- realizedProfitUSDT > 0 after all costs

Profit events are not “estimated.” They are committed facts.

## 8.2 Profit verification logic (mandatory)

Verification must:
- compute valuation delta in USDT terms
- incorporate:
  - swap fees
  - slippage impact
  - inventory conversion impact
- reject if:
  - oracle sanity fails
  - execution result deviates beyond tolerance
  - forbidden inventory holds exist

## 8.3 Settlement math (step-by-step formulas)

Definitions:
- `P` = realizedProfitUSDT
- `r_base` = base fee rate (0.20 for USDT payout; 0.175 for LOOP payout)
- `d` = discount multiplier from DiscountEngine (0 <= d <= 1)
- `r_final = r_base * (1 - d)`
- `F = P * r_final` (fee amount)
- `N = P - F` (net profit)
- `c` = compoundSelection (0.0 to 1.0)
- `C = N * c` (compounded portion)
- `W = N - C` (claimable portion)

Settlement steps:
1) compute `P`
2) if `P <= 0`: no fee; record non-profit log; exit settlement
3) determine payoutMode (USDT or LOOP)
4) set `r_base`
5) compute `d` using DiscountEngine
6) compute `r_final`
7) compute fee `F`
8) compute net `N`
9) allocate `C` and `W`
10) update ledgers and emit events
11) if payoutMode=LOOP: attempt swap for `W` and credit LOOP; on failure fallback to USDT credit

## 8.4 Fee application timing (final)

Fees are applied:
- immediately at ProfitEvent settlement
- before compounding allocation
- before claim ledger credits

## 8.5 Claim ledger mechanics (final)

ClaimLedger tracks:
- claimableUSDT
- claimableLOOP

Credits occur:
- at ProfitEvent settlement
- at cycle settlement checkpoint (YieldLoop vault) if configured to batch claimability

Debit occurs:
- at successful user claim/withdraw action

## 8.6 Compounding ledger mechanics (final)

Compounded amounts:
- remain inside the vault’s working capital
- increase effective principal for future execution
- are reflected in SettlementLedger snapshots

Compounding is not a separate token; it is a ledger allocation.

## 8.7 Negative performance rules (final)

If realizedProfitUSDT <= 0:
- no performance fee
- no discount application
- no claim credits
- record event log:
  - ProfitEventRejectedOrZero(vaultId, delta)

Drawdown states:
- if drawdown cap breached, vault enters PAUSED state (per vault) and no further trades occur until condition clears and timelocked unpause occurs.

## 8.8 Worked examples (mandatory)

Worked examples are required and will be included in Appendix E:
- profit event with USDT payout
- profit event with LOOP payout and discount
- cycle settlement example
- VaultOS break-lock early exit example
- redemption reserve update example

---

# 9. Fees, Discounts, and Routing (Final Math)

This section defines all fees, discounts, and routing rules as deterministic mechanics. No fee type exists outside this section.

## 9.1 Fee types and definitions (final list)

The platform implements exactly these fee types:

1) **Performance Fee (profit-only)**
   - Applies only to verified realized profit events (`realizedProfitUSDT > 0`)
   - Never applies to principal
   - Applies to both YieldLoop and VaultOS vaults

2) **Break-Lock Penalty (VaultOS-only)**
   - Applies only when a user exits a VaultOS goal vault before maturity
   - Is punitive by design
   - Never discounted

No deposit fees exist in MVP1.

## 9.2 Performance fee mechanics (final)

### 9.2.1 Base rates (locked)
- USDT payout: base fee rate = **20%**
- LOOP payout: base fee rate = **17.5%**

### 9.2.2 Discount application (final)
A discount reduces the base fee rate.

Definitions:
- `r_base` = base fee rate (0.20 or 0.175)
- `d_supporter` = supporter/native discount rate (0.00 to max allowed)
- `d_total = d_supporter` (final: single discount source for MVP1; no stacking beyond supporter/native + payout-mode base discount already reflected in r_base)
- `r_final = r_base * (1 - d_total)`

Notes:
- The LOOP payout discount is not an additional “discount”; it is already encoded as the lower base fee rate (17.5%).
- Supporter/native discount is the only discount multiplier applied on top of the selected base fee.

### 9.2.3 Timing (final)
Performance fee is assessed at ProfitEvent settlement:
- before compounding allocation
- before claim ledger credits
- before payout conversion (if LOOP payout)

## 9.3 DiscountEngine rules (final)

DiscountEngine is a single module used by YieldLoop and VaultOS.

### 9.3.1 Eligibility (final)
Supporter/native discount applies if:
- user holds the required supporter credential (per SupporterVerifier)
- credential is verified onchain at settlement time

### 9.3.2 Discount scope (final)
DiscountEngine affects:
- performance fee only

DiscountEngine does not affect:
- break-lock penalties
- reserve/redemption throttles
- slippage, trade limits, or guardrails

### 9.3.3 Discount percentage (final)
- Discount percentage is fixed by configuration and enforced uniformly platform-wide.
- Discount percentage is not user-selectable.
- Discount percentage changes require timelocked governance action.

### 9.3.4 No stacking beyond scope (final)
There is no discount stacking beyond:
- selecting payout mode (USDT vs LOOP) which selects base fee rate
- supporter/native discount which reduces that base fee rate

## 9.4 Fee routing wallets and splits (final)

All performance fees are routed through FeeRouter into fixed destination wallets.

Routing must:
- be deterministic
- be transparent
- emit event logs for every routed transfer

### 9.4.1 Destination categories (final)
Performance fees route to:
- Dev/Audit/Ops wallet (covers development, audits, operations)
- Marketing/Partnerships wallet (covers advertising and partnerships)
- Reserve wallet (funds redemption reserve and floor system support)

### 9.4.2 Split logic (final)
Fee splits are defined as percentages that sum to 100% of fee amount.

Split values are configurable but must be:
- set before launch
- timelocked for changes
- versioned and logged

### 9.4.3 Change control (final)
Fee routing changes:
- cannot affect already-settled profit events
- apply only to future ProfitEvents after timelock expiry
- must emit:
  - FeeSplitUpdateProposed
  - FeeSplitUpdateActivated

## 9.5 Routing update rules (timelocked; cannot affect past events)

All routing changes require:
- timelock delay (minimum 48 hours)
- onchain proposal + activation event logs
- public visibility in UI

## 9.6 Transparency requirements (final)

For each vault, the UI and events must provide:
- total realized profit
- total fees assessed
- fee rate used
- discount used
- fees routed per destination
- claimable USDT/LOOP totals

## 9.7 Worked examples (mandatory)
The platform must include worked examples showing:
- USDT payout fee computation
- LOOP payout fee computation
- supporter discount application
- fee routing splits
(Expanded examples are in Appendix E.)


---

# 10. LOOP Token System (Floor + Reserve + Redemption — Final)

This section defines the LOOP token value-support mechanics: payout conversion, reserve, redemption, and floor support. All behavior is deterministic and logged.

## 10.1 LOOP role summary (final)

LOOP exists to:
- serve as an optional payout asset for claimable profit
- provide a lower base performance fee path (17.5% vs 20%)
- be supported by a redemption reserve and redemption mechanics
- be supported by floor mechanisms defined here

LOOP is not:
- required for deposits
- used as a required entry token for VaultOS
- used to custody user principal

## 10.2 LOOP issuance rules (when LOOP can be created/distributed)

LOOP may enter circulation only via:
- payout conversion from claimable USDT profit into LOOP at settlement time
- reserve/floor operations that require acquiring LOOP on market (if defined below)

LOOP is not minted arbitrarily by admin in MVP1 unless explicitly defined under 10.2.1.  
Final policy for MVP1: **LOOP is NOT minted from thin air; it is acquired via market swap when paid out.**

### 10.2.1 No free mint policy (final)
- No protocol function may mint LOOP without an equal-value backing action that updates ReserveLedger.
- Admin cannot “print” LOOP.

## 10.3 LOOP payout mechanics (USDT profit → LOOP swap → distribute)

When payoutMode = LOOP, claimable USDT profit is converted into LOOP at settlement time.

### 10.3.1 Inputs
- `W` = claimable USDT amount after fees and compounding allocation

### 10.3.2 Allowed swap venues (final)
- PCS router (allowlisted routes only)
- BiSwap router (allowlisted routes only)

### 10.3.3 Swap execution requirements (final)
Swap must enforce:
- minOut based on slippage cap
- deadline
- oracle sanity check vs quoted price
- route allowlist validation
- liquidity floor validation

### 10.3.4 Swap failure fallbacks (final)
If LOOP conversion fails:
- claimable is credited in USDT instead
- emit LOOPSwapFailedFallback(vaultId, amountUSDT, reasonCode)

No retry loop that can strand funds is permitted.

## 10.4 Redemption mechanics (final)

Redemption enables LOOP holders to exchange LOOP for USDT from the Redemption Reserve under deterministic rules.

### 10.4.1 Redemption eligibility rules (final)
- Any address holding LOOP may request redemption if:
  - redemption is not globally paused
  - reserve has sufficient available USDT
  - requested amount does not exceed per-window cap

### 10.4.2 Redemption request flow (final)
User calls:
- `requestRedemption(amountLOOP, minOutUSDT)`

System:
- records redemption request
- computes expected USDT output using redemption price rules
- emits RedemptionRequested event

### 10.4.3 Redemption settlement flow (final)
Redemption settles via:
- immediate settlement if available liquidity and throttles permit, OR
- queued settlement via keeper job

Final policy: **Queued settlement via keeper is used** to enforce throttling and protect reserve from sudden drains.

Keeper calls:
- `settleRedemption(requestId)`

Settlement:
- burns or locks redeemed LOOP (final policy: LOOP is burned on redemption settlement)
- transfers USDT from reserve to user
- updates ReserveLedger
- emits RedemptionSettled

### 10.4.4 Redemption throttling rules (final)
Redemption is throttled to prevent bank-run dynamics.

Throttle rules:
- maximum USDT outflow per 24h window (ReserveOutflowCap)
- per-wallet redemption cap per 24h window (UserOutflowCap)
- if caps exceeded:
  - request is queued until window resets

Caps are configuration values, timelocked for changes, and logged.

### 10.4.5 Redemption failure handling (final)
If reserve is insufficient or redemption is paused:
- request remains pending but cannot settle
- user may cancel request and retain LOOP (final)
- emit RedemptionPausedOrInsufficient(requestId)

## 10.5 Redemption Reserve mechanics (final)

The Redemption Reserve is a pool of USDT (and optionally other stable assets) used to satisfy redemptions and support floor actions.

### 10.5.1 Reserve funding sources (final)
Reserve is funded by:
- a fixed split of performance fees (see FeeRouter)
- a fixed split of break-lock penalties (VaultOS only, 50% to reserve per section 6.6.4)

No other reserve funding sources exist in MVP1.

### 10.5.2 Reserve custody rules (final)
Reserve is held:
- onchain in a ReserveVault contract

Reserve assets allowed:
- USDT only for MVP1 (final; simplifies redemption and audit)

Reserve may not be deployed into external yield strategies in MVP1.

### 10.5.3 Reserve accounting ledger (final)
ReserveLedger tracks:
- inflows (from fees/penalties)
- outflows (redemptions/floor actions)
- current available reserve balance
- locked/queued redemption commitments

### 10.5.4 Reserve usage restrictions (final)
Reserve may be used only for:
- LOOP redemptions
- floor support actions (defined in 10.6)

Reserve may not be used for:
- operating expenses
- marketing spend
- dev spend
(those must be paid from their fee-routing wallets, not reserve)

### 10.5.5 Reserve transparency requirements (final)
UI must display:
- total reserve balance
- 24h redemption outflow
- queued redemption amounts
- reserve inflows by source

All reserve movements emit ReserveUpdated events.

## 10.6 LOOP floor system mechanics (final)

The LOOP floor is supported by a combination of redemption and reserve-backed actions.

### 10.6.1 Floor definition (final)
Floor is defined as a **protocol-declared minimum redemption value** per LOOP, subject to throttling.

- Floor is not a guaranteed market price.
- Floor is an enforceable redemption policy backed by the reserve, limited by caps.

### 10.6.2 Floor enforcement mechanisms (final)
The floor is enforced by:
1) **Redemption mechanism**: allows LOOP to be redeemed for USDT at a defined redemption price.
2) **Market support action** (only when configured): the protocol may buy LOOP using reserve funds under strict constraints.

Final MVP1 enforcement:
- The floor is enforced primarily by redemption.
- Market buybacks are permitted only when redemption demand indicates persistent undervaluation, and only within strict daily caps.

### 10.6.3 Floor adjustment mechanics (final)
Floor value changes require:
- timelocked governance action
- public notice via events
- cannot retroactively affect already-requested redemptions (requests settle under floor at request time)

Floor can only move upward by design in the configured schedule (final: upward-only adjustment; downward changes forbidden).

### 10.6.4 Crash/illiquidity edge cases (final)
If reserve is insufficient to satisfy all redemptions:
- redemption throttling queues requests
- reserve outflow cap prevents total drain
- redemptions settle over time as reserve refills via fees/penalties

If DEX liquidity is disrupted:
- market support actions are disabled
- redemption remains available subject to reserve and caps

If oracle failure occurs:
- floor actions pause until oracle sanity restored
- redemption may continue using last-sane pricing policy (final: redemption pauses if oracle is unavailable; no stale-price redemption)

## 10.7 Events required for LOOP system (final)

- LOOPSwapExecuted(vaultId, amountUSDT, amountLOOP)
- LOOPSwapFailedFallback(vaultId, amountUSDT, reasonCode)
- RedemptionRequested(requestId, user, amountLOOP, floorAtRequest)
- RedemptionSettled(requestId, user, amountLOOP, amountUSDT)
- RedemptionCancelled(requestId)
- RedemptionPausedOrInsufficient(requestId, reasonCode)
- ReserveUpdated(source, token, delta, newBalance)
- FloorUpdateProposed(newFloor, activationTime)
- FloorUpdateActivated(newFloor)

---

# 11. Security Model (Final)

This section defines the threat model and required defenses. Any implementation missing items in this section is considered insecure and incomplete.

## 11.1 Threat model (explicit)

YieldLoop + VaultOS must be designed to withstand:

### 11.1.1 Smart contract threats
- reentrancy attacks
- integer overflow/underflow
- approval abuse
- external call hijacking
- ERC20 non-standard behavior
- malicious token contracts
- delegatecall / proxy upgrade abuse

### 11.1.2 Market execution threats
- MEV sandwiching
- backrunning
- quote manipulation
- liquidity spoofing
- flashloan-based price manipulation
- bait liquidity pools

### 11.1.3 Oracle threats
- oracle feed manipulation
- stale or frozen oracle data
- oracle outage (no price)
- DEX/oracle divergence

### 11.1.4 Keeper/automation threats
- keeper downtime
- keeper abuse (spamming execution attempts)
- keeper collusion
- keeper key compromise

### 11.1.5 Admin/governance threats
- admin key compromise
- malicious parameter updates
- malicious keeper allowlisting
- malicious strategy allowlisting
- upgrade abuse

### 11.1.6 User threats
- phishing
- lost seed phrase
- dead/inaccessible owner wallet
- coercion attacks on recovery guardians

VaultOS recovery and inheritance mechanics exist specifically to reduce loss from user-side threats.

---

## 11.2 Reentrancy & external call safety (mandatory rules)

### 11.2.1 Reentrancy protection (mandatory)
All functions that:
- transfer tokens
- call DEX routers
- update claim balances
- update reserve balances
must use:
- Checks-Effects-Interactions pattern
- ReentrancyGuard

### 11.2.2 External call constraints
External calls are limited to:
- PCS router
- BiSwap router
- allowlisted ERC20 tokens
- oracle adapter

Any external call outside allowlist must revert.

### 11.2.3 Token approval discipline
Approval rules:
- never approve unlimited to unknown routers
- only approve allowlisted routers
- approvals should be:
  - per trade OR
  - bounded allowance with revocation and renewal cycle

Final policy:
- bounded allowance system (max allowance per router per token)
- optional revoke job via admin/keeper (timelocked)

---

## 11.3 Oracle attack resistance (mandatory)

### 11.3.1 Oracle usage policy
Oracle is used only for:
- sanity checks
- manipulation detection
- trade refusal triggers

Oracle is never used as:
- sole trade trigger
- sole valuation source for redemption without validation

### 11.3.2 Sanity check mechanics
A trade is rejected if:
- |DEXPrice - OraclePrice| > deviationThreshold

Deviation thresholds must be:
- set per token
- lower/tighter for VaultOS

### 11.3.3 Stale oracle rejection
If oracle data is stale beyond max age:
- reject trade
- reject floor actions
- reject redemption settlement (final policy)

### 11.3.4 Oracle outage fallback
If oracle unavailable:
- fail safe: no trade
- reserve actions paused
- redemptions paused
User withdrawals remain allowed.

---

## 11.4 MEV mitigation (mandatory)

The system must assume it is being watched and attacked.

Required MEV defenses:

### 11.4.1 Slippage caps enforced onchain
Every swap must include:
- minOut
- deadline

### 11.4.2 Cooldowns
Cooldown windows reduce predictability and repeated trading patterns.

### 11.4.3 Trade sizing constraints
Max trade size caps reduce MEV profitability and reduce impact.

### 11.4.4 Execution refusal under unstable conditions
If volatility or quote instability is detected:
- refuse execution

### 11.4.5 Transaction submission policy
Final requirement:
- keeper must support private relay submission when available
- if unavailable, enforce stronger caps and reduced frequency

---

## 11.5 Liquidity floor + price impact limits (mandatory)

Trades must be refused if:
- pool liquidity below threshold
- expected price impact above threshold
- route includes low-liquidity intermediary token

Route restrictions:
- no unknown multi-hop
- no “route discovery”
- only explicit allowlisted routes

---

## 11.6 Keeper abuse prevention (mandatory)

### 11.6.1 Keeper allowlist
Only allowlisted keepers may execute.

### 11.6.2 Rate limits
Engine must enforce:
- max executions per keeper per time window
- max executions per vault per time window

### 11.6.3 Mandatory logs
Every keeper attempt must emit:
- vaultId
- keeper address
- strategyId
- success/failure
- reason codes

### 11.6.4 Keeper compromise response
If keeper compromise suspected:
- admin can remove keeper via timelocked action (emergency path allowed with shorter timelock)
- protocol may enter PAUSED mode

---

## 11.7 Admin compromise mitigation (mandatory)

### 11.7.1 Multisig requirement
Admin must be a multisig wallet.

Final requirement:
- minimum 2-of-3 multisig for MVP1
- upgrade to 3-of-5 recommended post-MVP (not implemented now)

### 11.7.2 Timelock requirement
All admin actions except emergency pause must be timelocked.

Final timelock policy:
- 48 hours minimum delay for:
  - fee split updates
  - strategy allowlist updates
  - keeper allowlist updates
  - reserve parameter updates
  - floor adjustments

Emergency pause:
- may be executed immediately
- must be logged
- unpause must be timelocked

### 11.7.3 Admin limitations enforced by code
Admin cannot:
- withdraw user funds
- change principal ownership
- change maturity timestamps
- modify user claim balances
- force break-lock
- forge ProfitEvents
- mint LOOP arbitrarily

---

## 11.8 Upgrade safety model (timelock + multisig) (mandatory)

### 11.8.1 Upgradeability policy
Final MVP1 policy:
- Contracts may be upgradeable ONLY if:
  - all upgrades are timelocked
  - upgrade events are emitted
  - upgrade admin is multisig
  - upgrade process is visible in UI

### 11.8.2 Upgrade restrictions
Upgrades must not:
- change historical ledger meaning
- retroactively change fee rules
- change reserve accounting logic in a way that breaks transparency
- alter maturity timestamps

### 11.8.3 Emergency upgrade
No emergency instant-upgrade allowed.

---

## 11.9 Audit scope checklist (final)

Audit scope must cover:
- VaultFactory/VaultRegistry
- YieldLoopCore execution router
- GuardrailEngine
- OracleAdapter
- SettlementLedger/ClaimLedger correctness
- FeeRouter/DiscountEngine correctness
- VaultOSPenaltyEngine correctness
- Recovery/Inheritance engines correctness
- ReserveLedger + Redemption + Floor actions correctness
- event completeness
- role access boundaries
- upgradeability + timelock correctness

No audit = no launch.

---

# 12. Admin Control Boundaries (Final)

This section defines exactly what admin can do, cannot do, and how emergency actions work. The rules here are enforceable constraints, not intentions.

## 12.1 Admin can do (explicit list)

Admin is allowed to:

### 12.1.1 Manage keeper allowlist
- add keeper (timelocked)
- remove keeper (timelocked; emergency removal allowed with immediate effect if compromise suspected)

### 12.1.2 Manage strategy allowlist
- add strategy module (timelocked, audited)
- activate/deactivate strategy module (timelocked)
- adjust strategy parameters (timelocked)

### 12.1.3 Manage venue allowlist
- enable/disable PCS router
- enable/disable BiSwap router
- enable/disable specific token pairs (timelocked)

### 12.1.4 Manage fee routing configuration
- update routing destination wallets (timelocked)
- update routing splits (timelocked)

### 12.1.5 Manage reserve and floor parameters
- set redemption throttle caps (timelocked)
- set floor adjustment schedule (timelocked)
- pause redemptions (emergency immediate pause allowed)
- unpause redemptions (timelocked)

### 12.1.6 Pause trading (emergency)
Admin may:
- pause trading immediately (global or per strategy or per venue)
Pause must:
- preserve withdrawals
- preserve recovery
- preserve break-lock exits (VaultOS)
- preserve maturity exits (VaultOS)

---

## 12.2 Admin cannot do (explicit list)

Admin is forbidden from:

### 12.2.1 Any custody or seizure
- withdrawing user funds
- transferring user principal
- redirecting vault balances
- sweeping balances from vaults

### 12.2.2 Altering user commitments
- changing maturityTimestamp
- changing lockTermDays
- changing penalty schedule endpoints (20% → 5%)
- changing a vault type once created

### 12.2.3 Altering user claim rights
- editing ClaimLedger balances
- forcing conversion of claimable USDT to LOOP or vice versa
- preventing a valid maturity withdrawal
- preventing break-lock exit (VaultOS)
- preventing valid YieldLoop cycle-end withdrawal checkpoint

### 12.2.4 Forging accounting
- creating fake ProfitEvents
- editing SettlementLedger history
- rerouting already-settled fees retroactively

### 12.2.5 Printing LOOP
- minting LOOP without reserve-backed mechanism
- airdropping LOOP as compensation (not allowed)

---

## 12.3 Emergency actions allowed (explicit limits)

### 12.3.1 Emergency pause
Admin may emergency-pause:
- trading execution
- strategy modules
- venues
- redemptions

Admin may NOT emergency-pause:
- user withdrawals permitted by vault rules
- maturity exits
- break-lock exits
- recovery cancellation rights

### 12.3.2 Emergency disable of a strategy
If a strategy is found unsafe:
- admin can disable it immediately
- settlement and withdrawals still work
- vaults remain intact but do not trade

### 12.3.3 Emergency keeper removal
If keeper compromise is suspected:
- remove keeper immediately
- log event
- timelocked process required for re-add

---

## 12.4 Parameter updates (timelock rules)

Any parameter change must follow:

1) Proposal event emitted
2) Timelock delay elapses
3) Activation event emitted
4) Change visible in UI

Parameters requiring timelock:
- fee splits
- fee routing wallets
- strategy params
- guardrail params
- oracle thresholds
- reserve caps
- floor schedule

Emergency exception:
- only pauses are immediate

---

## 12.5 Strategy activation governance (timelocked)

A strategy may only become executable if:
- it is allowlisted
- it is activated after timelock
- its parameters are set and logged

A strategy may be deactivated:
- immediately if unsafe (emergency)
- otherwise via timelock

---

## 12.6 Admin event transparency (mandatory)

Admin actions must emit:
- AdminActionProposed(actionType, payloadHash, activationTime)
- AdminActionActivated(actionType, payloadHash)
- EmergencyPauseActivated(scope)
- EmergencyPauseReleased(scope)
- KeeperAdded/Removed
- StrategyAdded/Activated/Deactivated
- VenueEnabled/Disabled
- FeeSplitUpdateProposed/Activated
- ReserveParamUpdateProposed/Activated
- FloorUpdateProposed/Activated

No hidden admin actions are allowed.

---

# 13. UX / Frontend Requirements (Must Match Mechanics)

This section is non-negotiable. The frontend is part of compliance, consent enforcement, and deterministic user control.
If the UI does not implement this section exactly, the protocol is considered incomplete.

## 13.1 UX principles (final)

The UI must enforce:

1) **Truth-first UX**
   - UI must display only what the chain says.
   - UI must never show “estimated profit” as profit.
   - UI must never suggest guarantees.

2) **Consent enforcement**
   - YieldLoop requires explicit strategy acceptance before execution.
   - VaultOS requires explicit lock + penalty acceptance before creation.
   - Any material change requires re-consent when applicable.

3) **Deterministic user choice**
   - User must always know:
     - what can be claimed
     - when it can be claimed
     - what penalties apply
     - what fees apply
   - The system must never surprise the user.

4) **Transparency as a feature**
   - Every fee, routing split, reserve movement, and floor event must be visible.

5) **Recovery and inheritance are first-class**
   - VaultOS vault creation is not permitted without completing recovery setup.

---

## 13.2 Full screen inventory list (final)

The DApp must include exactly these screens.

### 13.2.1 Global / core screens
1) Landing / Home  
2) Connect Wallet / Authentication  
3) Dashboard (global overview)  
4) Notifications Center  
5) Settings (UI settings only)  
6) Support / Help Center (knowledge base + contact)

### 13.2.2 YieldLoop screens (Cycle vault product)
7) YieldLoop Vaults Overview  
8) Create YieldLoop Vault (Deposit)  
9) Strategy Proposal Review (mandatory)  
10) Strategy Acceptance Signature Screen (mandatory)  
11) YieldLoop Cycle Live View (Active cycle dashboard)  
12) YieldLoop Cycle Settlement View (cycle end status)  
13) YieldLoop Cycle End Choice Screen (compound/withdraw options)  
14) YieldLoop Claims Screen (claim USDT / claim LOOP)  
15) YieldLoop Vault History Screen (profit events + fees)

### 13.2.3 VaultOS screens (Generationz goal vault product)
16) VaultOS Overview (Goal Buckets list)  
17) Create Goal Vault (Bucket creation)  
18) Lock Term Selection Screen (mandatory)  
19) Break-Lock Penalty Disclosure Screen (mandatory)  
20) Recovery Setup Screen (mandatory)  
21) Beneficiary Setup Screen (mandatory)  
22) VaultOS Active Bucket Screen (per bucket view)  
23) VaultOS Maturity Screen (ready to withdraw)  
24) VaultOS Break Lock Screen (punitive funnel, mandatory friction)  
25) VaultOS Claim/Close Screen (maturity withdrawal)  
26) VaultOS Recovery Screen (initiate/cancel/review recovery status)

### 13.2.4 LOOP / Reserve / Redemption screens
27) LOOP Overview Screen  
28) Payout Mode Selection Screen (USDT vs LOOP)  
29) Redemption Screen (request/cancel/status)  
30) Reserve Transparency Screen  
31) Floor Status Screen

### 13.2.5 Transparency / audit screens
32) Profit Events Explorer (all vaults)  
33) Fees Explorer (all fees paid + routing)  
34) Contract Addresses & Module Registry  
35) Events Log Viewer (raw onchain events)

Total: **35 screens**.

---

## 13.3 Deposit UX flows (final)

### 13.3.1 Deposit rules shown everywhere
Whenever a user deposits, UI must show:
- deposit asset: USDT only
- min deposit: $100
- no insurance
- funds at risk
- fees on realized profit only
- LOOP payout option available

User must click: “I Understand” before deposit continues.

### 13.3.2 YieldLoop vault deposit flow
Flow:
1) Select product: YieldLoop Vault
2) Enter deposit amount (>= 100 USDT)
3) Confirm deposit
4) Create vault transaction
5) Vault created → redirect to Strategy Proposal Review

### 13.3.3 VaultOS goal vault deposit flow
Flow:
1) Select product: VaultOS Goal Vault
2) Create bucket label + category
3) Enter deposit amount (>= 100 USDT)
4) Select lock term
5) Show penalty disclosure (mandatory accept)
6) Configure recovery (mandatory)
7) Configure beneficiary (mandatory)
8) Create vault transaction
9) Vault created → redirect to Active Bucket view

No VaultOS goal vault may be created without completing steps 5–7.

---

## 13.4 YieldLoop Strategy Approval UX (mandatory)

YieldLoop execution is forbidden unless strategy is accepted.

### 13.4.1 Strategy Proposal Review Screen requirements
UI must display:

**Proposal Summary**
- inventory targets (e.g., BTCB primary)
- strategy modules to be used (band/grid + arbitrage)
- maximum trade frequency
- guardrail limits summary
- payout mode default
- compound selection default

**Risk and disclosure**
- DeFi risks
- no guarantee
- stop/decline option

**Fee disclosure**
- USDT payout fee: 20%
- LOOP payout fee: 17.5%
- supporter/native discount if eligible

User controls:
- Accept and sign
- Decline (vault stays idle; no execution)

### 13.4.2 Strategy Acceptance Signature Screen
User must sign:
- proposal hash
- vaultId
- timestamp
- key parameters

Result:
- StrategyAccepted receipt shown
- vault enters executable state

---

## 13.5 YieldLoop Active Cycle UX (final)

Active cycle view must show:

- cycle start timestamp
- cycle end timestamp
- vault state (ACTIVE_CYCLE)
- current inventory balances
- realized profit to date (settled only)
- profit events list
- fees paid list
- claimable balances (may be locked until cycle settlement)
- current chosen cycle-end option:
  - compound all
  - 50/50
  - withdraw all

The user may change the cycle-end option during cycle (until cutoff).

---

## 13.6 YieldLoop Cycle Settlement UX (final)

At cycle end:
- UI must block new settings changes
- UI must show settlement status:
  - settling
  - completed
  - delayed due to chain conditions

Once settled:
- show final realized profit
- show fees paid
- show claimable balance
- show compounding applied

User then moves to claims screen.

---

## 13.7 VaultOS bucket UX mechanics (mandatory)

VaultOS Active Bucket Screen must show:

- bucket name
- lock start timestamp
- maturity timestamp
- remaining time to maturity
- current penalty percent if break-lock executed now
- principalUSDT
- realized profit (settled only)
- claimable balances (subject to maturity rules)
- payout mode (USDT or LOOP)
- recovery configuration status (must be ACTIVE)
- beneficiary address (must be set)

VaultOS must always present:
- “Break Lock” button
- “Recovery” button
- “Maturity Withdraw” button (only appears once mature)

---

## 13.8 Break-lock UX funnel (warnings + friction) (mandatory)

Break-lock is designed to be punitive and inconvenient.

Break-lock flow:
1) Screen 1: Warning
   - “This permanently closes your vault.”
   - display current penalty % and penalty amount
   - show remaining time to maturity

2) Screen 2: Confirmation
   - require checkbox: “I accept the penalty.”
   - require checkbox: “I understand I lose the long-term plan benefits.”

3) Screen 3: Final review
   - show exact breakdown:
     - total withdrawal amount
     - penalty deducted
     - remaining payout
     - routing destinations summary

4) Screen 4: Execute
   - user signs transaction

If user cancels at any screen:
- return to bucket view
- no state changes

---

## 13.9 Recovery UX flows (VaultOS mandatory)

### 13.9.1 Recovery Setup Screen
Must enforce:
- recovery wallet set
- guardians set (min 3)
- threshold set
- beneficiary set

UI must provide:
- “Export Recovery Kit” (downloadable PDF or printable format)
- show unlock code generation instructions
- show warning: “We cannot recover your code for you.”

### 13.9.2 Recovery Initiation Screen
Recovery can be initiated by:
- recovery wallet OR
- guardians threshold OR
- timelock path

UI must show:
- time delay remaining
- ability for owner to cancel
- notifications that were sent

### 13.9.3 Recovery Cancellation Screen
Owner can cancel pending recovery:
- show “Cancel Recovery” button
- require signature transaction
- show confirmation receipt

---

## 13.10 Notifications & reminders (mandatory rules)

Notifications must be delivered:
- in-app
- email

Mandatory triggers:
- vault creation
- strategy proposal ready (YieldLoop)
- strategy accepted
- profit event committed
- cycle end approaching (7 days, 3 days, 24 hours)
- cycle settled
- VaultOS maturity approaching (90 days, 30 days, 7 days)
- VaultOS matured
- break-lock initiated
- recovery initiated
- recovery cancellation executed
- redemption requested/settled
- reserve parameter updates (admin)
- floor changes (admin)

VaultOS monthly reminders:
- Must send monthly reminder email for every active VaultOS vault.
- Email must include:
  - vault name
  - maturity date
  - penalty today
  - current claimable
  - link to DApp

---

## 13.11 Transparency dashboards (mandatory metrics)

Dashboard must display:

### 13.11.1 Per vault
- deposit history
- inventory
- profit events
- fees paid
- discounts applied
- claimable balances
- state machine state
- next event date (cycle end or maturity)

### 13.11.2 Platform-wide
- total TVL (USDT terms)
- total realized profit (settled)
- total fees collected
- reserve total
- redemptions queued
- redemptions settled last 24h
- floor value current
- floor adjustment timeline

---

## 13.12 Contract address registry screen (mandatory)

UI must list:
- VaultFactory address
- VaultRegistry address
- YieldLoopCore address
- GuardrailEngine address
- OracleAdapter address
- FeeRouter address
- DiscountEngine address
- ReserveVault address
- Redemption module address
- Strategy module addresses
- Timelock address
- Multisig admin address

Every address must include:
- copy button
- explorer link
- contract verified status indicator

---
# 14. Roadmap (Final MVP1 Implementation Plan)

This section is not “marketing roadmap.”  
It is the execution plan for building YieldLoop + VaultOS as a working product with a clean upgrade path.

## 14.1 MVP1 scope lock (final)

MVP1 ships ONLY the following, with zero deviation:

### 14.1.1 Chain + venues
- Chain: **BNB Chain (BSC)**
- Venues: **PCS + BiSwap only**
- No bridges
- No cross-chain

### 14.1.2 Vault products (exactly two)
1) **YieldLoop Vaults**
   - 30-day cycle vault
   - strategy proposal + acceptance gate required
   - cycle settlement
   - end-of-cycle user choice:
     - compound all
     - 50/50
     - withdraw all

2) **VaultOS Goal Vaults (Generationz vault type)**
   - bucket/goal system
   - lock terms: 90 days → 20 years in 30-day increments
   - maturity withdrawal penalty-free
   - early exit via Break Lock with penalty schedule (20% → 5% decay)
   - mandatory recovery + guardians + beneficiary

### 14.1.3 Strategy plugins (exactly two)
- Band/Grid strategy (conservative)
- PCS ↔ BiSwap arbitrage strategy (strict spread threshold)

No additional strategy is permitted in MVP1.

### 14.1.4 Token system (mandatory)
- Payout in USDT or LOOP
- LOOP payout discount: 17.5% base performance fee
- USDT payout: 20% base performance fee
- Supporter/native discount enforced by DiscountEngine
- Reserve + redemption + floor enforcement mechanics implemented

### 14.1.5 Security + ops (mandatory)
- multisig admin
- timelock governance
- keeper allowlist
- event logging completeness
- pause system (trading pauses but withdrawals preserved)
- full UI transparency screens as specified

### 14.1.6 No insurance (final)
- no insurance product
- no “guaranteed yield”
- no “risk free” claims

---

## 14.2 MVP1 build order (final)

The correct build order:

### Phase 1 — Core contracts (must pass unit tests)
1) VaultFactory
2) VaultRegistry
3) YieldLoopCore
4) SettlementLedger / ClaimLedger
5) GuardrailEngine
6) OracleAdapter

### Phase 2 — Fee + discount + routing
7) FeeRouter
8) DiscountEngine
9) SupporterVerifier (native/supporter discount credential verification)

### Phase 3 — Strategy plugins
10) Band/Grid plugin
11) PCS↔BiSwap Arbitrage plugin

### Phase 4 — Vault products
12) YieldLoopVault wrapper
13) VaultOSGoalVault wrapper (Generationz)

### Phase 5 — VaultOS penalty + recovery/inheritance
14) VaultOSPenaltyEngine
15) RecoveryWallet path
16) Guardian threshold path
17) Timelock recovery path
18) Beneficiary/inheritance rules

### Phase 6 — LOOP system
19) ReserveVault + ReserveLedger
20) Redemption module (queued settlement)
21) Floor support mechanisms

### Phase 7 — Automation + indexing
22) Keeper network
23) Indexer
24) Event-to-dashboard pipeline
25) Notification service (email + in-app)

### Phase 8 — Full UI
26) All required screens (35 total)
27) Consent and signature flows
28) Transparency dashboards

### Phase 9 — Audit + hardening
29) internal security review
30) external audit (SourceHat)
31) fix round

### Phase 10 — Launch
32) mainnet deploy
33) verified contracts
34) production monitoring
35) incident response runbooks

---

## 14.3 Post-MVP strategy additions (locked policy)

After MVP1, the system may add strategies ONLY if:

- strategy module follows plugin interface
- strategy is audited
- strategy is allowlisted via timelock
- vault type compatibility is explicitly declared
- UI disclosures updated
- worked examples added to appendices

Approved post-MVP candidate strategies:
- Index strategy (conservative)
- Yield farming strategy (low risk pools only)
- Lending strategy (collateralized only)
- Advanced arbitrage routing (still venue allowlisted)

Not approved unless a separate spec is written and audited:
- sniping
- kill-zone behavior
- unrestricted risk tiers for VaultOS
- cross-chain execution

---

## 14.4 Post-MVP chain evaluation criteria (only after MVP1)

BNB Chain is locked for MVP1.

A chain change (or multi-chain expansion) is allowed ONLY if the dev can prove:

1) equal or better execution determinism
2) equal or better DEX liquidity
3) equal or better oracle support
4) equal or better audit ecosystem
5) equal or better keeper reliability
6) no increase in user friction
7) clear improvement in cost or performance

IBC chains (Osmosis) may be considered only if:
- the product becomes IBC-native
- the liquidity + execution model provides proven advantage
- cross-chain friction is eliminated (not reduced—eliminated)
- security model remains deterministic and auditable

Final position:
- **EVM (BNB Chain) is best for MVP1 and likely MVP2.**
- Chain expansion is optional only after MVP1 completion.

---

# Appendix A — Full Contract + Module Map (Final)

This appendix defines the complete contract inventory, responsibilities, boundaries, and deployment architecture.

---

## A.1 Contract List and Responsibilities (Final)

### A.1.1 Core Registry Layer

#### 1) VaultFactory
**Purpose:** Single creation entrypoint for all vault types.  
**Responsibilities:**
- enforce minimum deposit ($100 USDT)
- enforce deposit asset (USDT only)
- create vault instance (YieldLoopVault or VaultOSGoalVault)
- register vault in VaultRegistry
- initialize immutable config:
  - vaultType
  - owner
  - deposit timestamp
  - principalUSDT
  - YieldLoop: cycle window definition
  - VaultOS: lock term and maturity timestamp
- emit VaultCreated event

**Forbidden:**
- holding custody long-term (factory should not hold funds)
- any withdrawal logic
- any trading logic

---

#### 2) VaultRegistry
**Purpose:** Global registry of vaults and ownership mapping.  
**Responsibilities:**
- map vaultId → vaultAddress
- map vaultId → vaultType
- map vaultId → owner
- map owner → vaultId list
- store vault status (active/closed)
- expose query views for UI/indexers

**Forbidden:**
- moving funds
- setting fees
- executing strategies

---

### A.1.2 Shared Execution Layer

#### 3) YieldLoopCore (ExecutionRouter)
**Purpose:** Shared execution engine for all vault types.  
**Responsibilities:**
- allowlisted keepers execute strategies
- enforce vault state eligibility
- enforce global protocol state eligibility
- call Strategy plugins
- call GuardrailEngine and OracleAdapter
- enforce DEX venue allowlist
- enforce inventory allowlist
- call settlement functions and commit ledgers

**Inputs:**
- vaultId
- strategyId
- strategyParams
- keeper signatures / allowlist membership

**Outputs:**
- ProfitEvent committed OR failure-safe event

**Forbidden:**
- bypassing guardrails/oracle checks
- any admin-custody function
- direct user withdrawal

---

#### 4) GuardrailEngine
**Purpose:** Prevent bad execution. Always errs toward refusal.  
**Responsibilities:**
- slippage caps (global and per vault type)
- liquidity floors
- trade sizing caps
- cooldown windows
- drawdown caps
- minimum spread thresholds for arbitrage

**Forbidden:**
- executing swaps itself
- updating user claim balances

---

#### 5) OracleAdapter
**Purpose:** Manipulation defense and sanity checks.  
**Responsibilities:**
- return oracle prices for allowlisted inventory tokens
- enforce max oracle staleness
- provide deviation checks: DEX quote vs oracle
- reject execution under suspicious divergence

**Forbidden:**
- predicting price
- triggering trades
- controlling execution frequency

---

### A.1.3 Accounting Layer

#### 6) SettlementLedger
**Purpose:** Deterministic profit event settlement truth.  
**Responsibilities:**
- store ProfitEvent entries:
  - realizedProfitUSDT
  - fee rate used
  - discount applied
  - fee amount
  - net profit
  - compounded amount
  - claimable amount
  - payout mode
- store cycle settlement snapshots (YieldLoop)
- store vault-level running totals

**Forbidden:**
- manual edits by admin
- retroactive modifications

---

#### 7) ClaimLedger
**Purpose:** Deterministic claimable balance truth.  
**Responsibilities:**
- store claimableUSDT
- store claimableLOOP
- store claim history
- store payout conversion receipts
- store “locked claimability” policy enforcement:
  - YieldLoop: claim at cycle checkpoint
  - VaultOS: maturity constraints enforced by vault rules

---

### A.1.4 Fees + Discounts Layer

#### 8) FeeRouter
**Purpose:** Receive fee amounts from settlement and route to destinations.  
**Responsibilities:**
- apply configured split
- route to:
  - Dev/Audit/Ops wallet
  - Marketing/Partnerships wallet
  - ReserveVault
- emit FeeRouted events

**Forbidden:**
- pulling fees from vault without settlement authorization
- retroactive routing changes

---

#### 9) DiscountEngine
**Purpose:** Compute supporter/native discounts.  
**Responsibilities:**
- verify eligibility via SupporterVerifier
- return discount multiplier
- apply only to performance fee

---

#### 10) SupporterVerifier
**Purpose:** Verify supporter/native credential eligibility.  
**Responsibilities:**
- check NFT or credential contract (allowlisted)
- return boolean eligibility
- return tier/discount class if needed

---

### A.1.5 Vault Product Layer

#### 11) YieldLoopVault (Cycle Vault Wrapper)
**Purpose:** Enforce 30-day cycle + acceptance gate.  
**Responsibilities:**
- store cycleStartTimestamp, cycleEndTimestamp
- enforce strategy acceptance required
- block withdrawals mid-cycle
- enforce cycle-end settlement
- enforce end-of-cycle user choice
- expose UI views

---

#### 12) VaultOSGoalVault (Generationz Wrapper)
**Purpose:** Enforce lock term, maturity, break-lock.  
**Responsibilities:**
- store lockStartTimestamp
- store maturityTimestamp
- enforce no partial withdrawals
- enforce break-lock exit only before maturity
- enforce maturity withdrawal after maturity
- enforce recovery/inheritance requirements

---

### A.1.6 VaultOS Penalty + Recovery + Inheritance Layer

#### 13) VaultOSPenaltyEngine
**Purpose:** Deterministic penalty schedule and computation.  
**Responsibilities:**
- compute penalty percent now
- compute penalty amount for expected withdrawal
- enforce penalty application at break-lock exit
- route penalty through PenaltyRouter
- emit PenaltyAssessed

---

#### 14) PenaltyRouter
**Purpose:** Route penalty amounts deterministically.  
**Responsibilities:**
- 50% → ReserveVault
- 50% → Ops/Dev wallet (or FeeRouter’s ops wallet)
- emit PenaltyRouted

---

#### 15) RecoveryEngine
**Purpose:** Recovery initiation and cancellation logic.  
**Responsibilities:**
- allow recovery wallet initiation
- allow guardian threshold initiation
- allow timelock initiation
- enforce delay windows (30 days / 180 days)
- allow cancellation by owner and/or guardians
- emit Recovery events

---

#### 16) InheritanceEngine
**Purpose:** Beneficiary withdrawal rights.  
**Responsibilities:**
- store beneficiary address per VaultOS vault
- enforce beneficiary-only claim path after completed recovery
- prevent mid-lock “transfer ownership”

---

### A.1.7 LOOP System Layer

#### 17) ReserveVault
**Purpose:** Hold reserve USDT.  
**Responsibilities:**
- receive fee/penalty inflows
- store reserve balance
- allow reserve outflows ONLY for redemption + floor actions
- enforce outflow caps and throttles
- emit ReserveUpdated events

---

#### 18) RedemptionModule
**Purpose:** LOOP redemption requests and settlement.  
**Responsibilities:**
- accept redemption request
- queue requests
- settle requests (keeper-driven)
- burn LOOP on redemption settle
- transfer USDT from ReserveVault
- throttle outflows
- allow cancellation if pending
- emit Redemption events

---

#### 19) FloorSupportModule
**Purpose:** Floor policy enforcement actions.  
**Responsibilities:**
- execute market buybacks only under strict constraints
- enforce daily caps
- stop when oracle unavailable
- stop when DEX liquidity unhealthy
- emit FloorSupportAction

---

### A.1.8 Governance / Admin Ops Layer

#### 20) TimelockController
**Purpose:** Timelocked activation of changes.  
**Responsibilities:**
- queue admin actions
- enforce delay minimums
- execute after delay
- emit Proposed/Executed events

---

#### 21) AdminMultisig
**Purpose:** Admin key control.  
**Requirement:** multisig only.  
**Responsibilities:**
- propose timelocked updates
- emergency pause only

---

## A.2 Dependency Graph (Final)

- VaultFactory → VaultRegistry
- VaultFactory → YieldLoopVault / VaultOSGoalVault
- YieldLoopCore → Strategy Plugins
- YieldLoopCore → GuardrailEngine
- YieldLoopCore → OracleAdapter
- YieldLoopCore → SettlementLedger + ClaimLedger
- SettlementLedger → FeeRouter
- FeeRouter → ReserveVault
- VaultOSGoalVault → PenaltyEngine + RecoveryEngine + InheritanceEngine
- PenaltyRouter → ReserveVault
- RedemptionModule → ReserveVault
- FloorSupportModule → ReserveVault
- TimelockController governs:
  - allowlist updates
  - fee routing updates
  - keeper updates
  - reserve/floor updates
  - strategy updates

---

## A.3 Deployment Order (Final)

Deployment must follow:

1) TimelockController
2) AdminMultisig (or configure existing)
3) VaultRegistry
4) GuardrailEngine
5) OracleAdapter
6) SettlementLedger
7) ClaimLedger
8) FeeRouter
9) DiscountEngine
10) SupporterVerifier
11) ReserveVault
12) RedemptionModule
13) FloorSupportModule
14) PenaltyEngine
15) PenaltyRouter
16) RecoveryEngine
17) InheritanceEngine
18) Strategy Plugins (Grid + Arbitrage)
19) YieldLoopCore
20) VaultFactory
21) YieldLoopVault template
22) VaultOSGoalVault template

Then:
- configure allowlists
- configure keepers
- configure fee splits
- configure reserve caps

---

## A.4 Addresses Registry Format (Final)

The UI must publish a registry file format:

### A.4.1 JSON schema
- chainId
- deployedAtTimestamp
- contracts:
  - VaultFactory
  - VaultRegistry
  - YieldLoopCore
  - GuardrailEngine
  - OracleAdapter
  - SettlementLedger
  - ClaimLedger
  - FeeRouter
  - DiscountEngine
  - SupporterVerifier
  - ReserveVault
  - RedemptionModule
  - FloorSupportModule
  - PenaltyEngine
  - PenaltyRouter
  - RecoveryEngine
  - InheritanceEngine
  - StrategyPlugins[]

### A.4.2 Verification requirements
Every address must include:
- explorer link
- verified source status
- deployment tx hash

---

# Appendix B — Vault State Machines (Final, No Diagram Code)

This appendix defines all state machines in plain-English + dev-implementable transition rules.
These state machines are mandatory and enforceable by contract logic.

---

## B.1 YieldLoop Vault State Machine (Cycle Vault)

### B.1.1 YieldLoop Vault States (Final)

1) **CREATED**
   - Vault exists.
   - Deposit confirmed.
   - No strategy proposal accepted yet.

2) **PROPOSED**
   - Strategy proposal has been generated.
   - User has not accepted it yet.
   - Trading is forbidden.

3) **IDLE_DECLINED**
   - User declined the current strategy proposal.
   - Trading is forbidden.
   - Vault remains open and funds remain held under vault control.

4) **ACCEPTED**
   - User accepted the strategy proposal via signature.
   - Vault is eligible for cycle start.
   - Trading may begin only after entering ACTIVE_CYCLE.

5) **ACTIVE_CYCLE**
   - 30-day loop is active.
   - Trading/arbitrage allowed under guardrails.
   - Withdrawals are forbidden.

6) **PAUSED**
   - Trading halted for safety.
   - Withdrawals still follow vault rules (YieldLoop withdrawals still occur only at cycle settlement checkpoint).
   - Pause does not equal closure.

7) **SETTLING**
   - Cycle ended.
   - System is settling final results.
   - No trading.

8) **CLOSED**
   - Vault cycle is finalized.
   - Final settlement complete.
   - Vault outcome completed and vault is closed.

---

### B.1.2 YieldLoop Vault Transition Rules (Final)

#### Transition list (event → from → to)

1) Strategy proposal generated  
   - CREATED → PROPOSED

2) User accepts proposal (signature receipt committed)  
   - PROPOSED → ACCEPTED

3) User declines proposal  
   - PROPOSED → IDLE_DECLINED

4) Proposal regenerated / refreshed  
   - IDLE_DECLINED → PROPOSED

5) Cycle start triggered  
   - ACCEPTED → ACTIVE_CYCLE  
   Condition: acceptance exists + vault not paused + global not paused

6) Cycle end reached  
   - ACTIVE_CYCLE → SETTLING  
   Condition: now >= cycleEndTimestamp

7) Settlement completes  
   - SETTLING → CLOSED  
   Condition: settlement success + ledgers updated + events emitted

8) Risk condition detected (oracle manipulation, DEX outage, repeated failures, drawdown breach)  
   - ACTIVE_CYCLE → PAUSED
   - SETTLING → PAUSED (allowed if settlement is unsafe due to oracle/DEX problems)

9) Unpause after delay and safety restored  
   - PAUSED → ACTIVE_CYCLE (if cycle still active and trading safe)
   - PAUSED → SETTLING (if cycle ended and settlement can resume)

10) User closes vault without accepting proposal  
   - CREATED → CLOSED (allowed)
   - PROPOSED → CLOSED (allowed)
   - IDLE_DECLINED → CLOSED (allowed)  
   Condition: no active cycle exists

---

### B.1.3 YieldLoop Vault Permission Matrix (Final)

| State          | Trading Allowed | Withdraw Allowed | Strategy Accept Allowed |
|----------------|-----------------|------------------|--------------------------|
| CREATED        | No              | Yes (close only) | No                       |
| PROPOSED       | No              | Yes (close only) | Yes                      |
| IDLE_DECLINED  | No              | Yes (close only) | No                       |
| ACCEPTED       | No (not yet)    | No               | No                       |
| ACTIVE_CYCLE   | Yes             | No               | No                       |
| PAUSED         | No              | No (until settle)| No                       |
| SETTLING       | No              | Only as allowed  | No                       |
| CLOSED         | No              | N/A              | No                       |

Notes:
- YieldLoop withdrawals do not occur mid-cycle.
- Settlement may need multiple transactions; SETTLING must tolerate delayed completion.

---

## B.2 VaultOS Goal Vault State Machine (Generationz Vault Type)

### B.2.1 VaultOS Goal Vault States (Final)

1) **CREATED**
   - Vault is created, deposit accepted.
   - Lock term is set.
   - Recovery + beneficiary config is finalized.
   - Must move immediately to ACTIVE_LOCKED.

2) **ACTIVE_LOCKED**
   - Lock is active.
   - Maturity not reached.
   - Trading may occur (conservative strategy only).
   - User withdrawal forbidden except Break Lock.

3) **MATURITY_READY**
   - now >= maturityTimestamp
   - Vault becomes eligible for penalty-free withdrawal.
   - Trading is halted by default (final: stop executing new trades once maturity reached).

4) **BREAK_LOCK_EXITING**
   - User initiated early exit.
   - Penalty must be computed and applied.
   - Vault is closing.

5) **RECOVERY_PENDING**
   - Recovery has been initiated (wallet/guardian/timelock).
   - Countdown delay is active.
   - Owner may cancel during delay window.

6) **BENEFICIARY_READY**
   - Recovery delay elapsed.
   - Beneficiary is authorized to withdraw and close vault.

7) **CLOSED**
   - Vault is closed.
   - Funds distributed to owner or beneficiary.
   - No further execution possible.

---

### B.2.2 VaultOS Goal Vault Transition Rules (Final)

#### Transition list (event → from → to)

1) Deposit confirmed and lock set  
   - CREATED → ACTIVE_LOCKED  
   Condition: vault creation complete + config complete

2) Maturity reached  
   - ACTIVE_LOCKED → MATURITY_READY  
   Condition: now >= maturityTimestamp

3) Maturity withdrawal executed  
   - MATURITY_READY → CLOSED  
   Condition: full close executed penalty-free

4) Break lock initiated  
   - ACTIVE_LOCKED → BREAK_LOCK_EXITING  
   Condition: now < maturityTimestamp

5) Break lock completion  
   - BREAK_LOCK_EXITING → CLOSED  
   Condition: penalty applied + routed + payout completed

6) Recovery initiated  
   - ACTIVE_LOCKED → RECOVERY_PENDING  
   Condition: valid recovery initiation path used

7) Recovery canceled by owner  
   - RECOVERY_PENDING → ACTIVE_LOCKED  
   Condition: cancel action executed before delay elapses

8) Recovery completes after delay  
   - RECOVERY_PENDING → BENEFICIARY_READY  
   Condition: delay elapsed without cancellation

9) Beneficiary withdrawal executed  
   - BENEFICIARY_READY → CLOSED

---

### B.2.3 VaultOS Permission Matrix (Final)

| State             | Trading Allowed | Withdrawal Allowed | Break Lock Allowed | Recovery Initiation Allowed |
|------------------|-----------------|--------------------|--------------------|-----------------------------|
| CREATED          | No              | No                 | No                 | No                          |
| ACTIVE_LOCKED    | Yes             | No                 | Yes                | Yes                         |
| MATURITY_READY   | No (final)      | Yes (close only)   | No                 | Yes (only if needed)        |
| BREAK_LOCK_EXITING | No            | Closing only       | N/A                | No                          |
| RECOVERY_PENDING | No              | No                 | No                 | No                          |
| BENEFICIARY_READY| No              | Yes (beneficiary)  | N/A                | No                          |
| CLOSED           | No              | N/A                | N/A                | No                          |

---

## B.3 Recovery State Machine (VaultOS Only)

Recovery is designed to prevent permanent loss of funds and enable inheritance outcomes.

### B.3.1 Recovery Modes (Final)

Recovery may be initiated through one of three modes:

1) **Recovery Wallet Mode**
   - Initiated by recoveryWalletAddress
   - Delay: 30 days

2) **Guardian Threshold Mode**
   - Initiated by guardian quorum signatures
   - Delay: 30 days

3) **Timelock Mode**
   - Initiated by anyone (or beneficiary) but requires unlock code verification
   - Delay: 180 days

---

### B.3.2 Recovery States (Final)

1) **RECOVERY_NONE**
   - No recovery active.

2) **RECOVERY_PENDING**
   - Recovery initiated.
   - Delay countdown active.
   - Owner cancellation permitted.

3) **RECOVERY_CANCELLED**
   - Recovery was canceled.
   - System returns to RECOVERY_NONE effectively (can be represented as RECOVERY_NONE; state included for logging clarity).

4) **RECOVERY_COMPLETED**
   - Recovery delay elapsed without cancellation.
   - Beneficiary becomes eligible for withdrawal.

---

### B.3.3 Recovery Transition Rules (Final)

1) Initiate recovery  
   - RECOVERY_NONE → RECOVERY_PENDING  
   Inputs:
   - modeType (WALLET / GUARDIAN / TIMELOCK)
   - initiator address
   - signatures if guardian mode
   - unlock code proof if timelock mode

2) Cancel recovery  
   - RECOVERY_PENDING → RECOVERY_CANCELLED  
   Condition:
   - owner cancels within delay window  
   OR
   - guardians cancel if owner wallet compromised (same threshold as guardian recovery)

3) Complete recovery  
   - RECOVERY_PENDING → RECOVERY_COMPLETED  
   Condition:
   - delay elapsed and no cancellation

4) Beneficiary withdraws after completed recovery  
   - RECOVERY_COMPLETED → vault CLOSED (vault state change)

---

### B.3.4 Mandatory Recovery Logging (Final)

Every recovery action must emit events:

- RecoveryInitiated(vaultId, modeType, initiator, startTimestamp, endTimestamp)
- RecoveryCancelled(vaultId, cancelledBy, timestamp)
- RecoveryCompleted(vaultId, timestamp)
- BeneficiaryWithdrawalExecuted(vaultId, beneficiary, amount, timestamp)

---

# Appendix C — Strategy Execution State Machines (Final, No Diagram Code)

This appendix defines the **exact strategy execution state machine** used by YieldLoopCore for all strategies.
No strategy is allowed to implement its own “custom execution lifecycle.”  
All strategies must conform to this standardized lifecycle to preserve safety, determinism, and auditability.

---

## C.1 Universal Strategy Run State Machine (Applies to ALL Strategies)

### C.1.1 Strategy Run States (Final)

Every execution attempt (a “StrategyRun”) must move through these states:

1) **RUN_CREATED**
   - A keeper (or authorized executor) has initiated a run request.

2) **ELIGIBILITY_CHECK**
   - Validate that the vault and protocol are eligible to execute.

3) **QUOTE_COLLECTION**
   - Collect DEX executable quotes for required swap legs.

4) **ORACLE_SANITY_CHECK**
   - Validate DEX quote prices against oracle sanity band.

5) **GUARDRAIL_CHECK**
   - Validate slippage, liquidity, sizing, cooldown, drawdown, spread threshold.

6) **TRADEPLAN_FINALIZED**
   - TradePlan is locked: route, legs, minOut, deadlines, token allowlist pass.

7) **EXECUTING**
   - Swaps are executed via allowlisted routers.

8) **POST_VERIFY**
   - Validate execution result:
     - received amounts
     - inventory allowlist
     - no abnormal deviations

9) **DELTA_COMPUTE**
   - Compute realized delta in USDT terms.

10) **SETTLEMENT**
   - If delta > 0, commit ProfitEvent and update ledgers.
   - If delta <= 0, log and exit (no fee).

11) **RUN_COMPLETED**
   - Run finished successfully (profit or no-profit).

12) **FAILED_SAFE**
   - Run refused or aborted safely.
   - No state mutation except logs.

---

### C.1.2 Universal Transition Rules (Final)

The allowed transitions are:

- RUN_CREATED → ELIGIBILITY_CHECK
- ELIGIBILITY_CHECK → QUOTE_COLLECTION
- QUOTE_COLLECTION → ORACLE_SANITY_CHECK
- ORACLE_SANITY_CHECK → GUARDRAIL_CHECK
- GUARDRAIL_CHECK → TRADEPLAN_FINALIZED
- TRADEPLAN_FINALIZED → EXECUTING
- EXECUTING → POST_VERIFY
- POST_VERIFY → DELTA_COMPUTE
- DELTA_COMPUTE → SETTLEMENT
- SETTLEMENT → RUN_COMPLETED

Failure-safe exits allowed from ANY step:
- ANY_STATE → FAILED_SAFE

---

### C.1.3 Strategy Run Permission Rules (Final)

A StrategyRun may proceed only if:

- caller is allowlisted keeper
- strategyId is allowlisted and activated
- vault is in an executable state:
  - YieldLoop: ACTIVE_CYCLE
  - VaultOS: ACTIVE_LOCKED only
- global protocol state is not PAUSED
- relevant venue is enabled
- oracle is available and not stale

If any condition fails:
- transition to FAILED_SAFE

---

## C.2 Eligibility Check State (ELIGIBILITY_CHECK)

### C.2.1 Inputs required (Final)

- vaultId
- strategyId
- vault type
- vault state
- global protocol state
- keeper allowlist membership
- cooldown timer info
- drawdown state info
- venue enable status
- oracle availability status

### C.2.2 Conditions that MUST FAIL SAFE (Final)

ELIGIBILITY_CHECK fails safe if:
- vault not found
- vault not active
- vault state not eligible
- strategy not allowlisted
- keeper not allowlisted
- vault paused
- global paused
- DEX venue paused
- oracle stale or unavailable
- required token feeds missing

---

## C.3 Quote Collection State (QUOTE_COLLECTION)

### C.3.1 Required quote types (Final)

- executable router quote from PCS
- executable router quote from BiSwap
- if multi-leg route: quote each leg

### C.3.2 Quote rejection conditions (Final)

- quote returns zero output
- quote cannot be fetched
- liquidity implied by quote below threshold
- route includes non-allowlisted token hops

If rejected → FAILED_SAFE

---

## C.4 Oracle Sanity Check State (ORACLE_SANITY_CHECK)

### C.4.1 Required oracle validation (Final)

For each token in the trade plan:
- oracle price available
- oracle freshness within max age
- DEX quote price within deviationThreshold

If any fails → FAILED_SAFE

### C.4.2 Deviation calculation (Final)

- `dexPrice = quoteOut / quoteIn` normalized
- `oraclePrice = oracleAdapter.getPrice(token)`
- deviation = abs(dexPrice - oraclePrice) / oraclePrice

Reject if:
- deviation > configuredThreshold(token, vaultType)

---

## C.5 Guardrail Check State (GUARDRAIL_CHECK)

Guardrails enforce refusal-before-loss.

### C.5.1 Mandatory checks (Final)

1) **Max slippage**
2) **Liquidity floor**
3) **Trade sizing cap**
4) **Cooldown satisfied**
5) **Drawdown cap not exceeded**
6) **Max trades per day not exceeded**
7) **Arbitrage minimum spread threshold** (if arbitrage strategy)
8) **Gas cap** (if enabled)

Failure of any check → FAILED_SAFE

---

## C.6 TradePlan Finalization State (TRADEPLAN_FINALIZED)

### C.6.1 TradePlan is immutable after this state (Final)

TradePlan includes:
- ordered list of swaps
- router used per swap (PCS or BiSwap only)
- tokenIn/tokenOut
- amountIn
- minOut
- deadline
- oracle sanity snapshot references
- guardrail snapshot references

Once finalized:
- it cannot be modified mid-run
- any condition change triggers full fail safe and restart later

---

## C.7 Executing State (EXECUTING)

### C.7.1 Execution constraints (Final)

Execution MUST:
- call allowlisted router only
- pass minOut
- pass deadline
- use safe transfer functions
- follow Checks-Effects-Interactions
- enforce reentrancy protection

### C.7.2 Execution failure conditions (Final)

If any swap reverts or fails:
- entire run must exit FAILED_SAFE
- no partial completion is acceptable unless explicitly safe and settled

Final policy:
- enforce atomic behavior where possible
- if atomicity is not possible, the strategy must be designed so partial completion cannot strand forbidden inventory

---

## C.8 Post Verify State (POST_VERIFY)

### C.8.1 Required post-verification checks (Final)

Verify:
- token balances changed as expected
- no non-allowlisted token present
- received output ≥ minOut tolerance
- no abnormal deviation vs expected execution result

If fails:
- FAILED_SAFE, and vault may auto-enter PAUSED depending on severity

---

## C.9 Delta Compute State (DELTA_COMPUTE)

### C.9.1 Realized delta definition (Final)

Realized delta is computed only from executed balances.

- Convert inventory value to USDT terms using:
  - direct router quote snapshots OR oracle valuation with DEX sanity constraints
- realizedProfitUSDT must be conservative:
  - prefer rounding down
  - do not include unrealized mark-ups

### C.9.2 Outcomes (Final)

- If realizedProfitUSDT > 0 → proceed to SETTLEMENT
- If realizedProfitUSDT <= 0 → proceed to SETTLEMENT but as “no-profit settlement”

---

## C.10 Settlement State (SETTLEMENT)

### C.10.1 Settlement paths (Final)

Settlement has two paths:

**Path A: Profit settlement**
- commit ProfitEvent
- compute fees
- apply discount
- allocate compounding vs claimable
- attempt LOOP payout conversion if selected
- update ledgers
- route fees

**Path B: No-profit settlement**
- do not assess fee
- do not allocate claim
- record failure/no-profit event log

### C.10.2 Failure handling (Final)
If settlement fails:
- revert the transaction
- do not partially update ledgers

No partial settlement is allowed.

---

## C.11 FAILED_SAFE State (Global Refusal State)

FAILED_SAFE means:
- do nothing safely
- no state mutation except logs
- preserve user funds

### C.11.1 Reason code requirements (Final)

FAILED_SAFE must emit:
- vaultId
- strategyId
- reasonCode enum
- relevant numeric values

Mandatory reason codes include:
- NOT_ELIGIBLE_STATE
- VAULT_PAUSED
- GLOBAL_PAUSED
- STRATEGY_NOT_ALLOWED
- KEEPER_NOT_ALLOWED
- ORACLE_UNAVAILABLE
- ORACLE_STALE
- ORACLE_DEVIATION_TOO_HIGH
- SLIPPAGE_TOO_HIGH
- LIQUIDITY_TOO_LOW
- COOLDOWN_ACTIVE
- DRAWDOWN_CAP_EXCEEDED
- SPREAD_TOO_LOW
- ROUTE_NOT_ALLOWLISTED
- EXECUTION_REVERTED
- POST_VERIFY_FAILED
- GAS_TOO_HIGH

---

# Appendix C.2 — Band/Grid Strategy Execution State Machine (Specific)

This section defines the Band/Grid strategy’s specific decision states on top of the universal run lifecycle.

## C.2.1 Band/Grid decision states (Final)

Band/Grid adds these internal decision states inside QUOTE_COLLECTION:

1) **BAND_EVAL**
   - determine current oracle price
   - determine band boundaries (upper/lower)
   - determine nearest grid levels

2) **LEVEL_SELECTION**
   - choose one level to act on:
     - buy level hit OR sell level hit
   - if no level hit → fail safe (no-op)

3) **INVENTORY_BALANCE_CHECK**
   - ensure inventory limits not exceeded
   - ensure enough USDT for buy / enough token for sell

If any fails:
- FAILED_SAFE

## C.2.2 Band/Grid refusal rules (Final)

Refuse execution if:
- price outside band by too much (volatility spike)
- insufficient inventory
- trade size exceeds cap
- cooldown active
- spread indicates manipulation

---

# Appendix C.3 — PCS↔BiSwap Arbitrage Strategy Execution State Machine (Specific)

This defines the arbitrage-specific decision path inside QUOTE_COLLECTION and GUARDRAIL_CHECK.

## C.3.1 Arbitrage decision states (Final)

1) **SPREAD_SCAN**
   - quote PCS buy vs BiSwap sell
   - quote BiSwap buy vs PCS sell

2) **NET_SPREAD_COMPUTE**
   - compute net spread after:
     - swap fees
     - slippage
     - gas estimate
   - derive netExpectedProfitUSDT

3) **SPREAD_THRESHOLD_GATE**
   - if netExpectedProfitUSDT < minThreshold → fail safe

4) **DIRECTION_LOCK**
   - lock trade direction:
     - PCS → BiSwap OR BiSwap → PCS

## C.3.2 Arbitrage refusal rules (Final)

Refuse execution if:
- spread not strong enough
- liquidity below floor
- oracle deviation high
- route contains disallowed hop
- execution would exceed max trade size
- cooldown active

---

# Appendix C.4 — Failure Escalation Policy (Final)

This appendix defines when repeated failures trigger PAUSED or DEGRADED states.

## C.4.1 Vault-level failure counters (Final)

Track per vault:
- failSafeCountRollingWindow (e.g., 1 hour)
- postVerifyFailureCountRollingWindow
- oracleFailureCountRollingWindow
- executionRevertCountRollingWindow

## C.4.2 Escalation rules (Final)

- If failSafeCount > threshold → vault enters DEGRADED (stricter params)
- If postVerifyFailures > threshold → vault enters PAUSED
- If oracleFailures persist > threshold → global redemption + floor actions pause
- If execution reverts persist > threshold → keeper removed and vault paused

## C.4.3 Recovery from PAUSED (Final)

- unpause requires:
  - timelock delay
  - oracle + DEX sanity restored
  - manual acknowledgment event

No silent unpause.

---

# Appendix C.5 — Required Execution Logs (Final)

Each StrategyRun must emit:

- StrategyRunStarted(vaultId, strategyId, keeper, timestamp)
- StrategyRunFailedSafe(vaultId, strategyId, keeper, reasonCode, timestamp)
- StrategyRunExecuted(vaultId, strategyId, keeper, amountIn, amountOut, timestamp)
- StrategyRunPostVerified(vaultId, strategyId, keeper, verified=true/false, timestamp)
- ProfitDeltaComputed(vaultId, strategyId, realizedProfitUSDT, timestamp)
- ProfitEventCommitted(vaultId, profitEventId, realizedProfitUSDT, feeRate, feeAmount, netProfit, timestamp)
- SettlementCommitted(vaultId, profitEventId, claimUSDT, claimLOOP, compoundedUSDT, timestamp)

---

# Appendix D — Default Parameter Sets (Conservative, Final)

This appendix defines the **exact default parameters** for MVP1.
These values are **final and implementation-binding** unless explicitly changed by a timelocked governance action.

There are two parameter profiles:

- **Profile 1: YieldLoop (Cycle Vaults)** → conservative, but active
- **Profile 2: VaultOS (Goal Vaults)** → stricter than YieldLoop (safety-first)

All parameters must be stored in GuardrailEngine and referenced by:
- vaultType
- token
- strategyId

---

## D.0 Global Defaults and System Constants (Final)

### D.0.1 Supported chain and venues (locked)
- Chain: BNB Chain (BSC)
- DEX venues allowed:
  - PancakeSwap (PCS)
  - BiSwap

### D.0.2 Deposit constraints (locked)
- Deposit asset: USDT only
- Minimum deposit: 100 USDT

### D.0.3 Oracle constraints (global)
- maxOracleStalenessSeconds: **300 seconds (5 minutes)**
- oracleDeviationRejectThresholdDefault:
  - YieldLoop default: **1.25%**
  - VaultOS default: **0.75%**

### D.0.4 DEX quote constraints (global)
- quoteValiditySeconds: **20 seconds**
- swapDeadlineSeconds: **120 seconds** (2 minutes)

### D.0.5 Settlement rounding policy (global)
- All user credits round down
- All fee calculations round up minimally to prevent undercollection
- Dust policy: dust routed to ReserveVault

### D.0.6 Gas refusal rule (global)
- If base fee and gas conditions imply unreliable settlement:
  - refuse trading
- Withdrawals remain available

---

## D.1 Slippage Caps (Final)

Slippage is defined as:
- `slippage = (expectedOut - actualMinOut) / expectedOut`

### D.1.1 Slippage caps — YieldLoop (Cycle vaults)

#### D.1.1.1 Stable routes (USDT ↔ major liquid asset)
- maxSlippageBps: **60 bps (0.60%)**

#### D.1.1.2 Non-stable routes (volatile pairs)
- maxSlippageBps: **90 bps (0.90%)**

#### D.1.1.3 LOOP payout conversion swap (USDT → LOOP)
- maxSlippageBps: **120 bps (1.20%)**
Reason: LOOP liquidity may be thinner than majors.

### D.1.2 Slippage caps — VaultOS (Goal vaults)
VaultOS must be stricter.

#### D.1.2.1 Stable routes
- maxSlippageBps: **40 bps (0.40%)**

#### D.1.2.2 Non-stable routes
- maxSlippageBps: **65 bps (0.65%)**

#### D.1.2.3 LOOP payout conversion swap
- maxSlippageBps: **95 bps (0.95%)**

---

## D.2 Liquidity Floors (Final)

Liquidity floors prevent execution in thin pools.

Liquidity is measured as:
- minimum reserve depth in USDT terms in the active pool
- AND max price impact estimate below cap

### D.2.1 Liquidity floor — YieldLoop
- minimumPoolLiquidityUSDT: **250,000 USDT**
- maxPriceImpactBps: **80 bps (0.80%)**

### D.2.2 Liquidity floor — VaultOS
- minimumPoolLiquidityUSDT: **500,000 USDT**
- maxPriceImpactBps: **50 bps (0.50%)**

---

## D.3 Trade Sizing Caps (Final)

Trade sizing caps limit loss exposure and MEV incentives.

### D.3.1 YieldLoop trade sizing caps

#### D.3.1.1 Max trade as % of vault value
- maxTradePercentOfVaultBps: **1500 bps (15%)**

#### D.3.1.2 Max absolute trade size
- maxTradeSizeUSDT: **2,500 USDT**

#### D.3.1.3 Max daily turnover
(turnover = sum of trade notionals in a 24h window)
- maxDailyTurnoverPercentOfVaultBps: **6000 bps (60%)**

### D.3.2 VaultOS trade sizing caps (stricter)

#### D.3.2.1 Max trade as % of vault value
- maxTradePercentOfVaultBps: **800 bps (8%)**

#### D.3.2.2 Max absolute trade size
- maxTradeSizeUSDT: **1,250 USDT**

#### D.3.2.3 Max daily turnover
- maxDailyTurnoverPercentOfVaultBps: **2500 bps (25%)**

---

## D.4 Cooldowns and Execution Frequency (Final)

Cooldowns reduce predictability and repeated trading attacks.

### D.4.1 YieldLoop cooldowns
- minSecondsBetweenTradesPerVault: **3,600 seconds (1 hour)**
- maxTradesPerVaultPerDay: **6**
- minSecondsBetweenSameStrategyRuns: **10,800 seconds (3 hours)**

### D.4.2 VaultOS cooldowns
- minSecondsBetweenTradesPerVault: **10,800 seconds (3 hours)**
- maxTradesPerVaultPerDay: **2**
- minSecondsBetweenSameStrategyRuns: **21,600 seconds (6 hours)**

---

## D.5 Drawdown Caps and Auto-Pause Thresholds (Final)

Drawdown is computed in USDT terms vs principal + prior settled value.

### D.5.1 YieldLoop drawdown limits
- maxDrawdownBps: **650 bps (6.50%)**
- pauseIfDrawdownBps: **800 bps (8.00%)**
- maxConsecutiveNoProfitRuns: **12**
- maxConsecutiveFailedSafeRuns: **20**

### D.5.2 VaultOS drawdown limits (stricter)
- maxDrawdownBps: **300 bps (3.00%)**
- pauseIfDrawdownBps: **400 bps (4.00%)**
- maxConsecutiveNoProfitRuns: **6**
- maxConsecutiveFailedSafeRuns: **10**

---

## D.6 Arbitrage Threshold Parameters (Final)

Arbitrage must exceed all costs plus safety margin.

### D.6.1 Spread definition
netSpreadUSDT must include:
- router fees
- expected slippage
- gas estimate
- execution uncertainty buffer

### D.6.2 YieldLoop arbitrage thresholds
- minNetProfitThresholdUSDT: **2.50 USDT**
- minNetSpreadBps: **18 bps (0.18%)**
- maxArbTradeSizeUSDT: **1,800 USDT**
- maxArbTradesPerDay: **3**

### D.6.3 VaultOS arbitrage thresholds (stricter)
- minNetProfitThresholdUSDT: **4.00 USDT**
- minNetSpreadBps: **28 bps (0.28%)**
- maxArbTradeSizeUSDT: **900 USDT**
- maxArbTradesPerDay: **1**

---

## D.7 Band/Grid Strategy Parameters (Final)

Band/Grid is conservative: no martingale, no doubling, no revenge trading.

### D.7.1 YieldLoop band/grid defaults
- gridLevels: **7**
- bandWidthBps: **900 bps (9.0%)**
- maxSingleLevelAllocationBps: **300 bps (3.0%) of vault**
- rebalanceCooldownSeconds: **86,400 seconds (24 hours)**

### D.7.2 VaultOS band/grid defaults (stricter)
- gridLevels: **5**
- bandWidthBps: **600 bps (6.0%)**
- maxSingleLevelAllocationBps: **175 bps (1.75%) of vault**
- rebalanceCooldownSeconds: **172,800 seconds (48 hours)**

---

## D.8 Token Allowlist (MVP1 Final)

MVP1 inventory allowlist is intentionally narrow.

### D.8.1 Allowed deposit token
- USDT (BEP-20)

### D.8.2 Allowed inventory tokens
- BTCB (primary)
- BNB (secondary utility)
- ETH (BSC peg)
- XRP (BSC peg)
- SOL (BSC peg)
- LOOP (payout and redemption token)

### D.8.3 Forbidden tokens (explicit)
- any token not listed above
- meme coins
- ultra-low-liquidity assets
- rebasing tokens
- fee-on-transfer tokens
- reflective tokens

Final rule:
- if token is not explicitly allowlisted, it is forbidden.

---

## D.9 Venue Health Rules (Final)

### D.9.1 Venue disable triggers
If any of the following occurs:
- router revert rate exceeds threshold
- pool liquidity collapses
- oracle deviation persists
- widespread slippage spikes

Then:
- venue is auto-disabled (strategy refuses execution)
- admin may pause and disable venue immediately

### D.9.2 Venue disable thresholds
YieldLoop:
- disableVenueAfterRevertsCount: **8 per 1h**
VaultOS:
- disableVenueAfterRevertsCount: **3 per 1h**

---

## D.10 Redemption Throttle Defaults (Final)

These parameters protect ReserveVault from drains.

### D.10.1 Reserve outflow caps
- maxReserveOutflowPer24hBps: **250 bps (2.50%) of reserve**
- maxReserveOutflowPerRequestUSDT: **2,500 USDT**

### D.10.2 User redemption caps
- maxUserRedemptionPer24hUSDT: **1,000 USDT**
- maxUserOpenRedemptionRequests: **3**

### D.10.3 Redemption settle cadence
- keeperSettleIntervalSeconds: **3,600 seconds (1 hour)**

---

## D.11 Floor Policy Defaults (Final)

### D.11.1 Floor adjustment schedule
- floorIsUpOnly: **TRUE**
- floorUpdateMinDelay: **48 hours timelock**
- maxFloorIncreasePer30DaysBps: **250 bps (2.50%)**

### D.11.2 Floor support buyback caps
- maxBuybackPer24hUSDT: **1.25% of reserve**
- buybackDisabledIfOracleStale: **TRUE**
- buybackDisabledIfLiquidityBelowFloor: **TRUE**
- buybackDisabledIfPriceImpactTooHigh: **TRUE**

---

## D.12 Parameter Override Hierarchy (Final)

Parameters are resolved in this order:

1) vaultType-specific parameter override
2) token-specific override
3) strategy-specific override
4) global default

If multiple overrides apply:
- the strictest setting wins

---

## D.13 Parameter Change Governance Rules (Final)

All parameter changes require:
- multisig proposal
- timelock (48h minimum)
- onchain activation event
- UI change log display

Emergency pause is allowed immediately but does NOT change parameters.

---

## D.14 Developer Acceptance Criteria for Appendix D

Implementation must include:
- GuardrailEngine storing all parameters above
- strict enforcement of every cap
- reason-coded failure-safe logs
- separate parameter sets for YieldLoop vs VaultOS
- separate thresholds for strategy types

If dev asks “what should we set X to,” and X is in this appendix, the answer is:
- “Use Appendix D.”

---

# Appendix E — Full Worked Examples (Math, Step-by-Step, Final)

This appendix provides deterministic, step-by-step examples for every critical mechanism.
All examples use explicit numbers so the dev can unit-test correctness.

Conventions used:
- All dollar values are USDT terms.
- Performance fees apply ONLY to verified realized profit events.
- No fee is ever applied to principal.
- Rounding policy:
  - user credits round DOWN (floor)
  - fee amounts round UP minimally (ceiling)
  - dust goes to ReserveVault

---

## E.0 Shared Variables and Definitions

### E.0.1 Profit event settlement variables
- `P` = realizedProfitUSDT for the event (must be > 0 to charge fee)
- `r_base` = base performance fee rate
  - USDT payout base: 0.20
  - LOOP payout base: 0.175
- `d_supporter` = supporter/native discount rate (0.00 to max configured)
- `r_final = r_base * (1 - d_supporter)`
- `F = feeAmountUSDT = ceil(P * r_final)`
- `N = netProfitUSDT = P - F`

### E.0.2 Compounding selection variables (YieldLoop or VaultOS, if enabled)
- `c` = compoundPercent selected by user (0 to 1)
- `C = compoundedUSDT = floor(N * c)`
- `W = withdrawableClaimUSDT = N - C`

### E.0.3 Fee routing split variables (example split used here)
FeeRouter routes fee `F` to:
- Dev/Audit/Ops = 50%
- Marketing/Partnerships = 25%
- ReserveVault = 25%

This split is configurable, but the example assumes:
- `F_ops = ceil(F * 0.50)`
- `F_mkt = floor(F * 0.25)`
- `F_res = F - F_ops - F_mkt` (remainder to reserve)

(Exact rounding behavior is dev-defined, but must be deterministic and documented; remainder-to-reserve is simplest.)

---

## E.1 Profit Event — USDT Payout (No Supporter Discount)

**Scenario**
- User is in YieldLoop cycle.
- A strategy run realizes profit.
- Payout mode: USDT
- Supporter discount: none
- User compound selection: 50%

**Inputs**
- `P = 100.00`
- `r_base = 0.20` (USDT payout)
- `d_supporter = 0.00`
- `c = 0.50`

**Step 1 — Compute final fee rate**
- `r_final = 0.20 * (1 - 0.00) = 0.20`

**Step 2 — Compute fee amount**
- `F = ceil(100.00 * 0.20) = ceil(20.00) = 20.00`

**Step 3 — Compute net profit**
- `N = 100.00 - 20.00 = 80.00`

**Step 4 — Apply compounding selection**
- `C = floor(80.00 * 0.50) = floor(40.00) = 40.00`
- `W = 80.00 - 40.00 = 40.00`

**Step 5 — Route fee**
- `F_ops = ceil(20.00 * 0.50) = ceil(10.00) = 10.00`
- `F_mkt = floor(20.00 * 0.25) = floor(5.00) = 5.00`
- `F_res = 20.00 - 10.00 - 5.00 = 5.00`

**Result**
- Fee charged: `20.00`
- Net profit: `80.00`
- Compounded: `40.00`
- Claimable USDT: `40.00`
- Routed:
  - Ops: `10.00`
  - Marketing: `5.00`
  - Reserve: `5.00`

---

## E.2 Profit Event — LOOP Payout (No Supporter Discount, With Conversion)

**Scenario**
- Realized profit event occurs.
- Payout mode: LOOP (lower base fee)
- Supporter discount: none
- Compound selection: 0% (claim all net profit)
- LOOP conversion happens at settlement time.

**Inputs**
- `P = 80.00`
- `r_base = 0.175` (LOOP payout)
- `d_supporter = 0.00`
- `c = 0.00`
- At conversion time, DEX executes:
  - `W_usdt = claimable USDT pre-conversion`
  - swap results in `LOOP_out = 1600.00` LOOP
  - Effective conversion rate: `1 LOOP = 0.04 USDT` (example only)

**Step 1 — Final fee rate**
- `r_final = 0.175 * (1 - 0.00) = 0.175`

**Step 2 — Fee amount**
- `F = ceil(80.00 * 0.175) = ceil(14.00) = 14.00`

**Step 3 — Net profit**
- `N = 80.00 - 14.00 = 66.00`

**Step 4 — Compounding**
- `C = floor(66.00 * 0.00) = 0.00`
- `W = 66.00 - 0.00 = 66.00`

**Step 5 — Fee routing (same split)**
- `F_ops = ceil(14.00 * 0.50) = ceil(7.00) = 7.00`
- `F_mkt = floor(14.00 * 0.25) = floor(3.50) = 3.00`
- `F_res = 14.00 - 7.00 - 3.00 = 4.00`

**Step 6 — LOOP conversion**
- Convert claimable USDT `W = 66.00` into LOOP via allowlisted DEX route.
- Example executed output:
  - `LOOP_out = 1600.00` LOOP

**Result**
- Fee charged: `14.00`
- Net profit: `66.00`
- Claimable asset: `1600.00 LOOP`
- Routed:
  - Ops: `7.00`
  - Marketing: `3.00`
  - Reserve: `4.00`
- If conversion fails → fallback credits `66.00 USDT` claimable instead.

---

## E.3 Profit Event — LOOP Payout + Supporter Discount

**Scenario**
- Payout mode: LOOP
- Supporter discount: 20% (example)
- Compound selection: 25%

**Inputs**
- `P = 200.00`
- `r_base = 0.175`
- `d_supporter = 0.20`
- `c = 0.25`

**Step 1 — Final fee rate**
- `r_final = 0.175 * (1 - 0.20)`
- `r_final = 0.175 * 0.80 = 0.14`

**Step 2 — Fee amount**
- `F = ceil(200.00 * 0.14) = ceil(28.00) = 28.00`

**Step 3 — Net profit**
- `N = 200.00 - 28.00 = 172.00`

**Step 4 — Compounding**
- `C = floor(172.00 * 0.25) = floor(43.00) = 43.00`
- `W = 172.00 - 43.00 = 129.00`

**Step 5 — Fee routing**
- `F_ops = ceil(28.00 * 0.50) = 14.00`
- `F_mkt = floor(28.00 * 0.25) = 7.00`
- `F_res = 28.00 - 14.00 - 7.00 = 7.00`

**Step 6 — LOOP conversion**
- Convert `W = 129.00 USDT` to LOOP at settlement time.
- Example executed output:
  - `LOOP_out = 3225.00 LOOP` (example)

**Result**
- Effective fee rate: 14.0%
- Fee charged: `28.00`
- Net profit: `172.00`
- Compounded: `43.00 USDT` value
- Claimable: `3225.00 LOOP` (example execution)
- Reserve receives: `7.00 USDT` from this event

---

## E.4 YieldLoop Cycle Settlement Example (30-Day Cycle)

**Scenario**
- User deposited: 1,000 USDT
- Cycle runs 30 days
- During cycle, there are 3 ProfitEvents (already settled as they occur)
- At cycle end, user’s selected cycle-end option is “Compound 50% / Withdraw 50%”
- Payout mode: USDT

**Cycle Inputs**
- Principal at start: `principal = 1000.00`
- ProfitEvent #1: `P1 = 40.00`
- ProfitEvent #2: `P2 = 25.00`
- ProfitEvent #3: `P3 = 60.00`
- Performance fee base: 20%
- Supporter discount: none
- Per-event compounding selection during cycle: 50% (same as cycle-end for simplicity)

**Event Settlement (repeat E.1 logic each event)**

### Event #1
- Fee: `F1 = ceil(40 * 0.20) = 8.00`
- Net: `N1 = 32.00`
- Compounded: `C1 = floor(32 * 0.50) = 16.00`
- Claimable: `W1 = 16.00`

### Event #2
- Fee: `F2 = ceil(25 * 0.20) = 5.00`
- Net: `N2 = 20.00`
- Compounded: `C2 = floor(20 * 0.50) = 10.00`
- Claimable: `W2 = 10.00`

### Event #3
- Fee: `F3 = ceil(60 * 0.20) = 12.00`
- Net: `N3 = 48.00`
- Compounded: `C3 = floor(48 * 0.50) = 24.00`
- Claimable: `W3 = 24.00`

**Cycle Totals**
- Total fees: `F_total = 8 + 5 + 12 = 25.00`
- Total net profit: `N_total = 32 + 20 + 48 = 100.00`
- Total compounded: `C_total = 16 + 10 + 24 = 50.00`
- Total claimable: `W_total = 16 + 10 + 24 = 50.00`

**Cycle End State**
- New principal basis for next cycle (if user continues):  
  `principal_next = 1000.00 + 50.00 = 1050.00`
- User can claim: `50.00 USDT`
- Vault closes (or rolls to next cycle depending on product rule; for MVP1 YieldLoop is cycle-based with end checkpoint and user choice applied).

---

## E.5 VaultOS Maturity Withdrawal Example (Penalty-Free)

**Scenario**
- User creates a VaultOS goal vault called “Xmas Club”
- Deposit: 500 USDT
- Lock term: 360 days
- Vault reaches maturity
- User withdraws at maturity penalty-free
- Performance fees still apply on profit events during lock period (profit-only)

**Inputs**
- Principal: `500.00`
- Over the term, total realized profit events netted:
  - Total realized profits summed: `P_total = 70.00`
- Assume user chose USDT payout throughout, no supporter discount, and compounding 100% (to maximize growth)

**Profit Event Aggregation (simplified)**
- Total performance fees:  
  `F_total = ceil(70.00 * 0.20) = 14.00`
- Total net profit:  
  `N_total = 70.00 - 14.00 = 56.00`
- If compounding was 100%:
  - Compounded total: `56.00`
  - Claimable during lock (if allowed): 0.00
  (VaultOS may allow claimable-only-at-maturity; final policy is enforceable by vault rules.)

**Maturity Withdrawal**
- Penalty at maturity: **0**
- User receives:
  - principal `500.00` + compounded `56.00` = `556.00 USDT`
- Vault closes.

---

## E.6 VaultOS Break-Lock Penalty Example (20% → 5% Decay)

**Scenario**
- User creates a VaultOS vault:
  - Deposit: 1,000 USDT
  - Lock term: 5 years (60 months)
  - Penalty schedule: starts at 20% and decays to 5% over full term
- User breaks lock at month 18

**Penalty schedule definition**
- Term months: `T = 60`
- Start penalty: `p_start = 0.20`
- End penalty: `p_end = 0.05`
- Month index at break: `m = 18`

Linear decay (final baseline rule):
- `p(m) = p_start - ( (p_start - p_end) * (m / T) )`

**Step 1 — Compute penalty percent**
- `p(m) = 0.20 - (0.15 * (18/60))`
- `18/60 = 0.30`
- `0.15 * 0.30 = 0.045`
- `p(m) = 0.20 - 0.045 = 0.155`

Penalty percent at month 18 = **15.5%**

**Assume current vault withdrawal value**
- Principal + compounded value at break time: `V = 1,120.00` (example, already settled)

**Step 2 — Compute penalty amount**
- `Penalty = ceil(V * 0.155) = ceil(173.60) = 174.00`

**Step 3 — Compute user payout**
- `UserPayout = V - Penalty = 1120.00 - 174.00 = 946.00`

**Step 4 — Route penalty (final routing example)**
PenaltyRouter split (final for MVP1):
- 50% to ReserveVault
- 50% to Ops/Dev wallet

- Reserve: `ceil(174.00 * 0.50) = 87.00`
- Ops/Dev: `174.00 - 87.00 = 87.00`

**Result**
- User receives: `946.00 USDT`
- Penalty paid: `174.00 USDT`
- Reserve inflow: `87.00 USDT`
- Ops/Dev inflow: `87.00 USDT`
- Vault closes permanently.

---

## E.7 Redemption Request + Settlement Example (Queued, Throttled)

**Scenario**
- ReserveVault currently holds: `Reserve = 50,000 USDT`
- Outflow cap: 2.5% per 24h (from Appendix D)
  - `cap24h = 50,000 * 0.025 = 1,250 USDT`
- User requests redemption of LOOP for 900 USDT worth
- Another user already redeemed 600 USDT in the last 24h window
- Therefore available capacity left: `1,250 - 600 = 650 USDT`

**Request**
- User requests: `900 USDT worth`
- Request must be queued because it exceeds remaining 650 capacity.

**Step 1 — RedemptionRequested logged**
- status: PENDING
- requestedOutUSDT: 900
- eligibleOutUSDTNow: 650
- queuedRemainder: 250

**Step 2 — Settlement in current window**
- Keeper settles up to 650 in this 24h window.
- User receives: `650 USDT`
- LOOP burned for corresponding amount (per redemption price rules)
- Request remains partially pending: 250

**Step 3 — Next window settlement**
- After window resets, keeper settles remaining 250.
- User receives: `250 USDT`
- LOOP burned for remainder
- Request becomes SETTLED

**Result**
- Total received: 900 USDT across two settlements
- Reserve outflow obeyed cap
- Full event trail emitted

---

## E.8 Reserve Ledger Bookkeeping Example (Fee + Penalty + Redemption)

**Scenario**
Start of day reserve: `ReserveStart = 10,000 USDT`

During the day:
1) FeeRouter routes reserve portion from performance fees: `+120 USDT`
2) PenaltyRouter routes reserve portion from break-lock penalties: `+300 USDT`
3) Redemption settlements pay out: `-500 USDT`

**Step 1 — Apply inflows**
- Reserve after fees: `10,000 + 120 = 10,120`
- Reserve after penalties: `10,120 + 300 = 10,420`

**Step 2 — Apply outflow**
- Reserve after redemptions: `10,420 - 500 = 9,920`

**End of day reserve**
- `ReserveEnd = 9,920 USDT`

**ReserveUpdated events required**
- ReserveUpdated(source="FeeRouter", delta=+120, newBalance=10120)
- ReserveUpdated(source="PenaltyRouter", delta=+300, newBalance=10420)
- ReserveUpdated(source="RedemptionModule", delta=-500, newBalance=9920)

---

## E.9 “No Profit” Event Example (Fee Must Be Zero)

**Scenario**
A strategy run executes but produces negative or zero delta after verification.

**Inputs**
- realizedProfitUSDT `P = -3.20`

**Rules**
- If `P <= 0`:
  - performance fee = 0
  - no claim credit is created
  - settlement logs the no-profit outcome

**Result**
- Fee: 0
- Claimable: 0
- Event emitted: ProfitDeltaComputed(..., -3.20) + NoProfitEventLogged(...)

---

## E.10 LOOP Conversion Failure Fallback Example

**Scenario**
- Settlement computes claimable: `W = 55.00 USDT`
- Payout mode: LOOP
- Attempted swap fails (router revert or minOut not met)

**Rule**
- fallback to USDT claim credit

**Result**
- claimableLOOP: 0
- claimableUSDT: 55.00
- Event emitted: LOOPSwapFailedFallback(vaultId, 55.00, reasonCode)

---

# Appendix F — Disclosure Templates (Final UI Copy)

This appendix contains the exact disclosure text templates that must appear in the DApp.
These templates are designed to:
- force clear user consent
- prevent misleading claims
- reduce regulatory risk
- prevent “I didn’t know” disputes

UI requirements:
- The user must scroll to bottom before acceptance checkbox unlocks
- Acceptance is recorded onchain (or signed message logged and indexed)
- Any material change to configuration requires re-consent

---

## F.0 Global Disclosure Rules (Final)

### F.0.1 Required formatting
- Each disclosure must include:
  - Title
  - Bullet risk list
  - Fees summary
  - Consent checkbox
  - “I decline” option (when applicable)

### F.0.2 Required universal line
Every disclosure must include this sentence verbatim:

> YieldLoop and VaultOS are DeFi protocols. Your funds are at risk. Past performance does not predict future results.

---

## F.1 YieldLoop Deposit Disclosure (Final)

### Title
**YieldLoop Deposit Disclosure (Read Before Depositing)**

### Body
By depositing into a YieldLoop Vault you understand:

- This is a DeFi product.
- There is no guaranteed profit.
- You may lose some or all of your deposit.
- Smart contract bugs are possible even after audit.
- DEX liquidity changes can cause losses.
- Oracle failure can halt trading or settlement.
- Market movements can cause drawdowns.

### Restrictions
- You cannot withdraw during an active 30-day cycle.
- You can change your cycle-end choice before the cycle ends.
- You can claim only after settlement checkpoints.

### Fee Summary
- Performance fee applies only to realized profit events.
- Base performance fee:
  - 20% if paid out in USDT
  - 17.5% if paid out in LOOP
- Supporter/native discounts may reduce the fee.

### Consent Checkbox
[ ] I understand and accept these risks and restrictions.

Buttons:
- **Continue to Deposit**
- **Cancel**

---

## F.2 YieldLoop Strategy Proposal Disclosure (Final)

### Title
**Strategy Proposal Disclosure (Approval Required)**

### Body
Before the vault begins execution you must approve a strategy proposal.

You understand:
- The strategy may change what assets are held during the cycle.
- The strategy may perform:
  - Band/Grid trades
  - PCS ↔ BiSwap arbitrage trades
- Strategy execution is bounded by guardrails, but guardrails do not guarantee profit.
- The system may refuse to trade under unsafe conditions.
- Refusal to trade may reduce profit potential.

### Fee Summary
- Performance fee applies only to realized profit events.
- Fee is deducted before compounding and before claim credits.

### Consent Checkbox
[ ] I understand this strategy is not financial advice and I approve it.

Buttons:
- **Approve Strategy**
- **Decline Strategy (Vault will remain idle)**

---

## F.3 YieldLoop Active Cycle Risk Disclosure (Final)

### Title
**Active Cycle Notice (Cycle Locked)**

### Body
Your vault is currently in an active 30-day cycle.

You understand:
- Withdrawals are locked until the cycle settlement checkpoint.
- The system may trade assets as needed for execution.
- Prices may move against the strategy.
- Arbitrage may fail due to slippage, MEV, or liquidity.
- The system may pause trading for safety.

### Consent Checkbox
[ ] I understand and wish to continue.

Button:
- **Return to Vault**

---

## F.4 YieldLoop Cycle-End Option Disclosure (Final)

### Title
**Cycle-End Choice Disclosure (Must Confirm Before Cycle Ends)**

### Body
At the end of the 30-day cycle you may select:

- Compound all
- Compound 50% / Withdraw 50%
- Withdraw all

You understand:
- Your selection affects how profit is handled.
- Your selection can be changed before the cycle ends.
- Profit is not guaranteed.

### Consent Checkbox
[ ] I confirm my selection.

Button:
- **Confirm Choice**

---

## F.5 VaultOS Deposit + Lock Disclosure (Final)

### Title
**VaultOS Goal Vault Disclosure (Lock-Based Savings Vault)**

### Body
VaultOS is a long-term savings vault. You understand:

- You are depositing USDT into a locked goal vault.
- Your lock term is binding.
- You can exit early only through Break Lock with a penalty.
- The penalty is designed to discourage early exit.
- This is a DeFi product and funds are at risk.

### Lock Rules
- Lock duration: (user-selected)
- Maturity date: (shown)
- Penalty schedule: decreases over time from 20% down to 5%

### Fee Summary
- Performance fee applies only to realized profit events.
- Base performance fee:
  - 20% if paid out in USDT
  - 17.5% if paid out in LOOP
- Supporter/native discounts may reduce the fee.
- Break Lock penalty is NOT discounted.

### Consent Checkbox
[ ] I understand this is a locked goal vault and early exit will trigger a penalty.

Buttons:
- **Continue**
- **Cancel**

---

## F.6 VaultOS Break Lock Disclosure (Final)

### Title
**Break Lock Warning (Penalty Applies)**

### Body
If you break the lock:

- Your vault will permanently close.
- A penalty will be deducted from your withdrawal.
- The penalty amount depends on time remaining.

This is designed to feel inconvenient and expensive because VaultOS is built for long-term outcomes.

### Penalty Disclosure
- Current penalty percent: (shown)
- Current penalty amount: (shown)
- Net payout after penalty: (shown)

### Consent Checkbox
[ ] I understand I am closing my vault early and accepting the penalty.

Buttons:
- **Proceed to Final Confirmation**
- **Cancel**

---

## F.7 VaultOS Recovery Disclosure (Final)

### Title
**Recovery & Inheritance Disclosure (Mandatory Setup)**

### Body
VaultOS requires a recovery system. You understand:

- If you lose wallet access, recovery is the only method to access funds.
- Recovery uses:
  - recovery wallet address
  - guardian threshold approvals
  - optional timelock unlock path
- YieldLoop cannot recover your wallet seed phrase.
- You are responsible for storing your Recovery Kit.

### Consent Checkbox
[ ] I understand recovery is required and I accept responsibility for securing my Recovery Kit.

Buttons:
- **Continue to Setup**
- **Cancel Vault Creation**

---

## F.8 Beneficiary / Inheritance Disclosure (Final)

### Title
**Beneficiary Disclosure (Who Receives Funds If You Disappear)**

### Body
You must assign a beneficiary. You understand:

- The beneficiary can only withdraw after a completed recovery.
- The beneficiary cannot withdraw early without the recovery system completing.
- You can change beneficiary later through a verified update process (timelocked if needed).

### Consent Checkbox
[ ] I understand the beneficiary will receive this vault if recovery completes.

Buttons:
- **Confirm Beneficiary**
- **Cancel**

---

## F.9 LOOP Payout Disclosure (Final)

### Title
**LOOP Payout Option Disclosure (Lower Fee, Different Asset)**

### Body
You may choose to receive profit payouts in LOOP instead of USDT.

You understand:
- LOOP is a volatile token.
- LOOP market price may drop.
- LOOP payout uses a USDT → LOOP conversion swap at settlement time.
- If conversion fails, you will receive USDT instead.
- LOOP payout uses a lower base performance fee.

### Fee Summary
- USDT payout base fee: 20%
- LOOP payout base fee: 17.5%

### Consent Checkbox
[ ] I understand LOOP payouts are volatile and I accept receiving LOOP.

Buttons:
- **Select LOOP Payout**
- **Cancel**

---

## F.10 Redemption Disclosure (Final)

### Title
**LOOP Redemption Disclosure (Throttle + Queue)**

### Body
You may redeem LOOP for USDT from the reserve.

You understand:
- Redemption is throttled.
- Redemption may queue and settle over time.
- Redemption is not instant liquidity during heavy demand.
- Reserve can only pay out within safety caps.

### Consent Checkbox
[ ] I understand redemption may be delayed and is subject to throttling.

Buttons:
- **Request Redemption**
- **Cancel**

---

## F.11 “No Insurance” Disclosure (Final)

### Title
**No Insurance Notice**

### Body
There is no insurance protection in MVP1.

You understand:
- Losses are possible
- exploit events can occur
- price moves can reduce vault value
- redemption may be throttled

### Consent Checkbox
[ ] I understand there is no insurance.

Button:
- **Continue**

---

# Appendix G — Full Glossary (Final)

This glossary defines every term used in YieldLoop and VaultOS.

---

## G.1 Core Terms

### YieldLoop
A 30-day cycle vault system that executes strategies only after user acceptance, then settles deterministically.

### VaultOS
A goal-based locked vault system used for long-term savings, gifting, and generational wealth setups.

### Vault
A user-specific isolated contract instance that holds funds and enforces a vault type’s rules.

### Vault Type
A category of vault with distinct rules:
- YieldLoopVault (cycle rules)
- VaultOSGoalVault (lock rules)

### VaultFactory
Contract module that creates vaults.

### VaultRegistry
Contract module that records vault addresses and ownership mapping.

---

## G.2 Trading / Strategy Terms

### Strategy
A plugin module that produces trades under GuardrailEngine constraints.

### StrategyRun
One execution attempt by a keeper.

### Band/Grid Strategy
A conservative buy/sell system that trades around price levels within a defined band.

### Arbitrage Strategy
A strategy that attempts to buy on one DEX and sell on another to capture spread.

### PCS
PancakeSwap. Allowlisted venue.

### BiSwap
BiSwap DEX. Allowlisted venue.

### MEV
Miner/Maximal Extractable Value. Attacks include sandwiching and backrunning.

---

## G.3 Accounting Terms

### ProfitEvent
A settlement record created when realizedProfitUSDT > 0.

### realizedProfitUSDT
Profit measured in USDT terms and verified post-execution and post-swap.

### Performance Fee
A fee charged only on realized profit events.

### ClaimLedger
Module that stores claimable user balances (USDT and LOOP).

### SettlementLedger
Module that stores ProfitEvents and cycle settlement records.

---

## G.4 Fee and Discount Terms

### FeeRouter
Module that routes collected fees to fixed destinations.

### DiscountEngine
Module that applies supporter/native discount to performance fee.

### Supporter / Native Discount
A discount applied to performance fee based on user credential (NFT or similar).

---

## G.5 LOOP Token Terms

### LOOP
Protocol payout token optionally received instead of USDT.

### ReserveVault
USDT reserve used to support LOOP redemption and floor mechanisms.

### Redemption
The process of exchanging LOOP for USDT from the reserve.

### Throttle
Redemption cap system that limits outflow per time window.

### Floor
A redemption-backed minimum value policy, enforced through reserve rules and redemption mechanics.

---

## G.6 VaultOS Terms

### Lock Term
Duration selected at vault creation. Enforced until maturity unless Break Lock used.

### Maturity
Timestamp at which vault unlocks penalty-free withdrawal.

### Break Lock
Early exit action that permanently closes vault and applies penalty.

### Penalty Schedule
A declining penalty from 20% to 5% over the lock duration.

### Recovery Kit
Printable/downloadable recovery instructions and guardian list for VaultOS.

### Guardian
An address authorized to participate in recovery.

### Beneficiary
Address authorized to receive vault proceeds after recovery completes.

---

## G.7 Security Terms

### Keeper
Allowlisted executor that triggers StrategyRuns.

### Timelock
Delay mechanism that prevents instant admin changes.

### Multisig
Multiple-signature admin wallet required for governance actions.

---

## G.8 UI Terms

### Disclosure
Mandatory text a user must accept before depositing or changing settings.

### Acceptance Gate
Mechanism requiring user signature before strategy execution starts.

### Transparency Screens
UI pages that expose fees, profits, reserve balances, and floor status.

---

# Appendix H — Event Schema (Final)

This appendix defines the **complete required event set** for YieldLoop + VaultOS (VaultOS = Generationz).
Events are not optional — they are required for:
- UI transparency
- indexers
- dispute resolution
- auditability
- regulatory defensibility

Rule:
> If an important action can occur, it must emit an event.

All events must include:
- chain timestamp
- vaultId where applicable
- actor address where applicable

---

## H.0 Event Standards (Final)

### H.0.1 Naming
- Use consistent, explicit names:
  - VaultCreated
  - StrategyProposalGenerated
  - ProfitEventCommitted

### H.0.2 IDs
- vaultId must be a uint256
- profitEventId must be uint256, scoped per vault
- requestId must be uint256 for redemptions

### H.0.3 Reason codes
All failure events must use reasonCode enums, never strings.

### H.0.4 Minimum indexed fields
Each event must index at least:
- vaultId (where applicable)
- user address (where applicable)
- strategyId (where applicable)

---

# H.1 VaultFactory Events (Final)

## VaultCreated
Emitted when any vault is created.

Fields:
- vaultId (indexed)
- vaultAddress
- owner (indexed)
- vaultType (enum)
- depositToken (address)
- depositAmountUSDT
- createdTimestamp

## VaultCreationFailed
Emitted if vault creation is attempted but refused.

Fields:
- attemptedOwner (indexed)
- vaultType
- depositAmountUSDT
- reasonCode
- timestamp

---

# H.2 VaultRegistry Events (Final)

## VaultRegistered
Fields:
- vaultId (indexed)
- vaultAddress
- owner (indexed)
- vaultType
- timestamp

## VaultOwnerUpdated
Allowed only if explicitly supported (final MVP1: NOT supported).

Fields:
- vaultId (indexed)
- oldOwner
- newOwner
- reasonCode
- timestamp

Final note:
- In MVP1, vault owner is immutable. This event should never fire.

---

# H.3 YieldLoopVault Events (Final)

## StrategyProposalGenerated
Fields:
- vaultId (indexed)
- proposalHash (indexed)
- strategyList (array of strategyIds)
- parametersHash
- generatedTimestamp

## StrategyProposalDeclined
Fields:
- vaultId (indexed)
- proposalHash (indexed)
- declinedBy (indexed)
- declinedTimestamp

## StrategyAccepted
Fields:
- vaultId (indexed)
- proposalHash (indexed)
- acceptedBy (indexed)
- acceptanceSignatureHash
- acceptedTimestamp

## CycleStarted
Fields:
- vaultId (indexed)
- cycleStartTimestamp
- cycleEndTimestamp

## CycleEndChoiceUpdated
Fields:
- vaultId (indexed)
- updatedBy (indexed)
- selection (enum: COMPOUND_ALL / HALF_HALF / WITHDRAW_ALL)
- timestamp

## CycleEnded
Fields:
- vaultId (indexed)
- cycleEndTimestamp

## CycleSettlementStarted
Fields:
- vaultId (indexed)
- timestamp

## CycleSettlementCompleted
Fields:
- vaultId (indexed)
- cycleId
- totalProfitUSDT
- totalFeesUSDT
- totalCompoundedUSDT
- totalClaimableUSDT
- totalClaimableLOOP
- timestamp

---

# H.4 VaultOSGoalVault (Generationz) Events (Final)

## GoalVaultCreated
Fields:
- vaultId (indexed)
- vaultAddress
- owner (indexed)
- bucketLabelHash
- depositAmountUSDT
- lockStartTimestamp
- maturityTimestamp
- penaltyStartBps (2000)
- penaltyEndBps (500)

## BeneficiaryUpdated
Fields:
- vaultId (indexed)
- updatedBy (indexed)
- oldBeneficiary
- newBeneficiary
- timestamp

## RecoveryConfigured
Fields:
- vaultId (indexed)
- owner (indexed)
- recoveryWallet
- guardianCount
- threshold
- configuredTimestamp

## BreakLockInitiated
Fields:
- vaultId (indexed)
- initiatedBy (indexed)
- penaltyBpsNow
- penaltyAmountUSDT
- netPayoutUSDT
- timestamp

## BreakLockExecuted
Fields:
- vaultId (indexed)
- executedBy (indexed)
- penaltyAmountUSDT
- netPayoutUSDT
- timestamp

## MaturityReached
Fields:
- vaultId (indexed)
- maturityTimestamp
- timestamp

## MaturityWithdrawalExecuted
Fields:
- vaultId (indexed)
- executedBy (indexed)
- withdrawalAmountUSDT
- withdrawalAmountLOOP
- timestamp

---

# H.5 YieldLoopCore StrategyRun Events (Final)

## StrategyRunStarted
Fields:
- vaultId (indexed)
- strategyId (indexed)
- keeper (indexed)
- attemptId
- startedTimestamp

## StrategyRunFailedSafe
Fields:
- vaultId (indexed)
- strategyId (indexed)
- keeper (indexed)
- attemptId
- reasonCode
- numericContext1 (optional)
- numericContext2 (optional)
- timestamp

## StrategyRunExecuting
Fields:
- vaultId (indexed)
- strategyId (indexed)
- keeper (indexed)
- attemptId
- venue (PCS/BISWAP)
- timestamp

## StrategyRunExecuted
Fields:
- vaultId (indexed)
- strategyId (indexed)
- keeper (indexed)
- attemptId
- tokenIn
- tokenOut
- amountIn
- amountOut
- venue
- timestamp

## StrategyRunPostVerified
Fields:
- vaultId (indexed)
- strategyId (indexed)
- keeper (indexed)
- attemptId
- verified (bool)
- timestamp

## ProfitDeltaComputed
Fields:
- vaultId (indexed)
- strategyId (indexed)
- attemptId
- realizedProfitUSDT
- timestamp

---

# H.6 Settlement and Profit Events (Final)

## ProfitEventCommitted
Fields:
- vaultId (indexed)
- profitEventId (indexed)
- strategyId (indexed)
- realizedProfitUSDT
- payoutMode (USDT/LOOP)
- feeRateBps
- supporterDiscountBps
- feeAmountUSDT
- netProfitUSDT
- compoundedUSDT
- claimableUSDT
- timestamp

## NoProfitEventLogged
Fields:
- vaultId (indexed)
- attemptId
- realizedProfitUSDT (<= 0)
- reasonCode
- timestamp

## ClaimBalanceUpdated
Fields:
- vaultId (indexed)
- user (indexed)
- deltaUSDT
- deltaLOOP
- newClaimableUSDT
- newClaimableLOOP
- timestamp

---

# H.7 FeeRouter / DiscountEngine Events (Final)

## DiscountApplied
Fields:
- vaultId (indexed)
- user (indexed)
- payoutMode
- baseFeeRateBps
- supporterDiscountBps
- effectiveFeeRateBps
- timestamp

## FeeAssessed
Fields:
- vaultId (indexed)
- profitEventId (indexed)
- feeAmountUSDT
- feeRateBps
- timestamp

## FeeRouted
Fields:
- vaultId (indexed)
- profitEventId (indexed)
- destinationType (OPS/MKT/RESERVE)
- destinationAddress
- amountUSDT
- timestamp

---

# H.8 ReserveVault / Redemption Events (Final)

## ReserveUpdated
Fields:
- sourceType (FEES/PENALTY/REDEMPTION_OUT/FLOOR_ACTION)
- sourceVaultId (optional)
- token (address) (final MVP1: USDT only)
- deltaAmount
- newReserveBalance
- timestamp

## RedemptionRequested
Fields:
- requestId (indexed)
- requester (indexed)
- amountLOOP
- expectedOutUSDT
- floorAtRequest
- queued (bool)
- timestamp

## RedemptionCancelled
Fields:
- requestId (indexed)
- requester (indexed)
- cancelledTimestamp

## RedemptionPartiallySettled
Fields:
- requestId (indexed)
- requester (indexed)
- usdtPaidOutThisSettlement
- remainingOutUSDT
- timestamp

## RedemptionSettled
Fields:
- requestId (indexed)
- requester (indexed)
- totalUSDTOut
- loopBurned
- timestamp

## RedemptionRefused
Fields:
- requestId (indexed)
- requester (indexed)
- reasonCode
- timestamp

---

# H.9 LOOP Conversion Events (Final)

## LOOPSwapExecuted
Fields:
- vaultId (indexed)
- profitEventId (indexed)
- amountInUSDT
- amountOutLOOP
- venue
- timestamp

## LOOPSwapFailedFallback
Fields:
- vaultId (indexed)
- profitEventId (indexed)
- amountUSDT
- reasonCode
- timestamp

---

# H.10 Admin / Governance / Timelock Events (Final)

## AdminActionProposed
Fields:
- actionId (indexed)
- proposedBy (indexed)
- actionType (enum)
- payloadHash
- activationTimestamp
- timestamp

## AdminActionActivated
Fields:
- actionId (indexed)
- activatedBy (indexed)
- actionType
- payloadHash
- activatedTimestamp

## EmergencyPauseActivated
Fields:
- scope (GLOBAL / STRATEGY / VENUE / REDEMPTION)
- scopeId (optional)
- activatedBy (indexed)
- timestamp

## EmergencyPauseReleased
Fields:
- scope
- scopeId (optional)
- releasedBy (indexed)
- timestamp

## KeeperAllowlistUpdated
Fields:
- keeper (indexed)
- added (bool)
- updatedBy (indexed)
- timestamp

## StrategyAllowlistUpdated
Fields:
- strategyId (indexed)
- added (bool)
- updatedBy (indexed)
- timestamp

## VenueStatusUpdated
Fields:
- venue (PCS/BISWAP)
- enabled (bool)
- updatedBy (indexed)
- timestamp

## FeeSplitUpdateProposed
Fields:
- splitId
- newSplitHash
- activationTimestamp
- timestamp

## FeeSplitUpdateActivated
Fields:
- splitId
- newSplitHash
- activatedTimestamp
- timestamp

## ReserveParamUpdateProposed
Fields:
- paramSetHash
- activationTimestamp
- timestamp

## ReserveParamUpdateActivated
Fields:
- paramSetHash
- activatedTimestamp
- timestamp

## FloorUpdateProposed
Fields:
- newFloorValue
- activationTimestamp
- timestamp

## FloorUpdateActivated
Fields:
- newFloorValue
- activatedTimestamp
- timestamp

---

# H.11 Required Reason Code Enum List (Final)

All modules must share a single ReasonCode enum.

Minimum enum values required:

- OK
- NOT_ELIGIBLE_STATE
- VAULT_NOT_FOUND
- VAULT_CLOSED
- GLOBAL_PAUSED
- VAULT_PAUSED
- STRATEGY_NOT_ALLOWED
- KEEPER_NOT_ALLOWED
- VENUE_DISABLED
- ORACLE_UNAVAILABLE
- ORACLE_STALE
- ORACLE_DEVIATION_TOO_HIGH
- SLIPPAGE_TOO_HIGH
- LIQUIDITY_TOO_LOW
- PRICE_IMPACT_TOO_HIGH
- COOLDOWN_ACTIVE
- MAX_TRADES_EXCEEDED
- DRAWDOWN_CAP_EXCEEDED
- SPREAD_TOO_LOW
- ROUTE_NOT_ALLOWLISTED
- TOKEN_NOT_ALLOWLISTED
- EXECUTION_REVERTED
- POST_VERIFY_FAILED
- GAS_TOO_HIGH
- RESERVE_INSUFFICIENT
- REDEMPTION_THROTTLED
- REDEMPTION_QUEUE_FULL
- FLOOR_ACTION_DISABLED
- UNAUTHORIZED_CALLER
- INVALID_SIGNATURE
- INVALID_UNLOCK_CODE
- RECOVERY_ALREADY_PENDING
- RECOVERY_NOT_PENDING
- RECOVERY_DELAY_NOT_ELAPSED

---

# Appendix I — Test Plan + Acceptance Criteria (Final)

This appendix defines what must be tested before launch and what “ready” means.

---

## I.1 Unit Test Requirements (Final)

### I.1.1 VaultFactory tests
- rejects deposits < 100 USDT
- rejects non-USDT deposits
- creates YieldLoop vault correctly
- creates VaultOS vault correctly
- emits VaultCreated correctly
- registers vault in registry

### I.1.2 VaultRegistry tests
- owner mappings accurate
- vault list query accurate
- vaultId uniqueness
- no owner mutation (MVP1)

### I.1.3 GuardrailEngine tests
- slippage cap refusal
- liquidity floor refusal
- trade size refusal
- cooldown refusal
- drawdown refusal
- degraded mode override application

### I.1.4 OracleAdapter tests
- staleness refusal
- deviation refusal
- missing feed refusal
- safe fallback behavior

### I.1.5 YieldLoopCore tests
- only allowlisted keeper can execute
- only allowlisted strategy executes
- all refusal paths log failure-safe
- atomic execution enforcement
- execution revert behavior safe
- post verify failure triggers pause

### I.1.6 SettlementLedger + ClaimLedger tests
- ProfitEvent committed correctly
- No-profit event committed correctly
- fee computed correctly
- compounding computed correctly
- claim balances computed correctly
- rounding rules enforced

### I.1.7 FeeRouter tests
- fee split routing correct
- reserve routing correct
- emits FeeRouted correctly
- fee updates timelocked

### I.1.8 VaultOSPenaltyEngine tests
- penalty percent computed correctly across term
- penalty decreases from 20% → 5%
- penalty rounding correct
- penalty routing correct

### I.1.9 RecoveryEngine tests
- recovery wallet initiation valid
- guardian initiation valid
- unlock code path valid
- cancellation valid
- beneficiary ready only after delay
- cannot bypass delay

### I.1.10 Reserve + Redemption tests
- reserve inflows correct
- reserve outflows capped
- queue ordering FIFO
- partial settlement works
- cancellation works
- LOOP burn correct

---

## I.2 Integration Test Requirements (Final)

Integration tests must run against:
- BSC fork environment
- PCS router
- BiSwap router
- real token contracts (fork)
- oracle mock + oracle real adapter

Must test:
- full YieldLoop cycle end-to-end
- full VaultOS lock-to-maturity end-to-end
- break-lock end-to-end
- redemption queue end-to-end
- pause/unpause paths

---

## I.3 Fork Testing Requirements (Final)

Fork tests must simulate:
- high MEV environment
- liquidity collapse
- oracle staleness
- DEX outage
- repeated failure-safe events
- repeated execution reverts

Success means:
- funds remain safe
- no ledger corruption
- refusal paths hold

---

## I.4 Launch Readiness Acceptance Criteria (Final)

Launch is permitted only when all are true:

1) 100% unit test pass
2) integration tests pass
3) fork tests pass
4) external audit completed (SourceHat)
5) all audit fixes applied
6) contract sources verified
7) event schema complete and indexer validated
8) UI transparency screens functional
9) emergency pause tested
10) reserve + redemption throttles tested
11) no admin custody function exists
12) disclosure templates implemented
13) recovery kit export works
14) beneficiary recovery flow works

---

# Appendix J — Incident Response Runbooks (Final)

These runbooks define exact playbooks for protocol survival.

---

## J.1 Keeper Down

**Symptoms**
- strategy runs stop
- no settlement progresses
- vaults show “delayed”

**Actions**
1) Verify keeper health dashboard
2) Enable backup keeper address
3) Increase settle interval (timelocked if needed)
4) Notify users via in-app + email

**Requirement**
- withdrawals remain governed by vault rules
- redemptions may delay but remain queued

---

## J.2 Oracle Outage / Staleness

**Symptoms**
- ORACLE_STALE failures
- strategies refuse trade
- floor actions disabled
- redemption paused

**Actions**
1) Pause trading execution immediately
2) Leave withdrawals enabled
3) Pause redemption settlement (keep queue open)
4) Replace oracle adapter feed (timelock required if structural)
5) Notify users

---

## J.3 DEX Outage (PCS or BiSwap)

**Symptoms**
- VENUE_DISABLED
- repeated execution reverts
- quotes fail

**Actions**
1) Disable affected venue
2) Force strategy to use remaining venue only (if safe)
3) Pause arbitrage strategy if both venues unstable
4) Notify users

---

## J.4 MEV Exploitation Detection

**Symptoms**
- repeated post-verify anomalies
- slippage spikes
- unexpected spread collapses

**Actions**
1) Pause strategy module
2) Tighten slippage caps (timelock unless emergency param set exists)
3) Reduce trade size caps
4) Switch keepers to private relay submission if supported
5) Notify users

---

## J.5 Admin Compromise Suspected

**Symptoms**
- unusual admin proposals
- unknown keeper additions
- strategy allowlist anomalies

**Actions**
1) Emergency pause GLOBAL
2) Rotate multisig keys
3) Freeze timelock execution if supported
4) Public disclosure notice to users
5) Audit last 7 days event logs
6) Re-enable after review

---

## J.6 Emergency Pause Procedure

**Goal**
Stop trading without harming user withdrawal rights.

**Actions**
1) Activate EmergencyPause GLOBAL
2) Disable all strategies
3) Disable floor buybacks
4) Leave maturity + break-lock exits enabled
5) Leave YieldLoop cycle settlement enabled if safe
6) Notify users in-app + email

---

## J.7 Reserve Drain / Redemption Bank Run

**Symptoms**
- high redemption queue volume
- outflow approaching cap
- LOOP price under heavy pressure

**Actions**
1) Ensure throttles enforced
2) Pause floor buybacks (do not waste reserve)
3) Maintain redemption queue FIFO
4) Publish reserve transparency report
5) Notify users that redemption may delay

---

## J.8 Contract Exploit Found

**Symptoms**
- abnormal outflows
- missing funds
- inconsistent ledger

**Actions**
1) Emergency pause GLOBAL
2) Disable redemption settlement
3) Disable all strategies
4) Preserve logs
5) Engage audit team immediately
6) Deploy patched version if upgradeable
7) If not upgradeable: migrate plan required (separate doc)

---

## J.9 Communications Protocol (Mandatory)

All incident notices must include:
- what happened (facts only)
- what systems paused
- what actions users can/cannot take
- expected next checkpoint time (if known)
- links to transparency dashboard

No spin. No hype.
