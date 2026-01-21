# YieldLoop — Builder Document (Dev-Ready / Zero Ambiguity)

## Title Page (LOCKED)

**Document Title:** YieldLoop — Complete Builder Document (Dev-Ready / No Ambiguity)  
**Project/Product Name:** YieldLoop  
**Token:** LOOP (Redeem Token)  
**Document Type:** Engineering Build Specification (Primary Source of Truth)  
**Status:** FINAL BUILD DOC — Developer must not decide missing logic  
**Chain:** BNB Chain (BSC / EVM)  
**DEX Venues (Locked v1):** PancakeSwap (PCS) + BiSwap  
**Permitted Strategies (Locked v1):** Spot Trading + PCS↔BiSwap Arbitrage  
**Deposit Asset (Locked v1):** USDT (BEP-20)  
**Minimum Deposit:** 250 USDT  
**Author:** Todd Koletsky  
**Version:** 1.0.0-BUILD  
**Date:** January 20, 2026  

---

## Simple Table of Contents (LOCKED)

0. High-Level Summary (What to Build, What Not to Build)  
1. System Requirements (Hard Rules / Non-Negotiables)  
2. User Vault Model (Positions, Allocation, Compounding, Rewards)  
3. Strategy Layer (Spot + Arbitrage)  
4. Profit Accounting + Settlement (No fake profit)  
5. Fee System (USDT vs LOOP claim rates + routing splits)  
6. LOOP Token System (Minting rules + claim system)  
7. System Reserve (SR) + Collateralization Ratio (CR)  
8. Floor Price Ratchet (Monotonic floor)  
9. Redemption Engine (Base + bonus tiers + queue)  
10. VaultOS Locking Engine (30 days → 25 years; cap boost)  
11. Safety Guardrails + Failure Handling + Pauses  
12. Engine List (Inputs/Outputs/Triggers/Fallbacks per Engine)  
13. Contract Architecture (Contracts, storage, roles, upgrade policy)  
14. Off-Chain Services (keepers/bots, quote engine, monitoring, email)  
15. UI/UX Requirements (All screens + required values + user prompts)  
16. Parameter Defaults (All defaults locked)  
17. Test Plan + Simulation Plan (must pass before launch)  
18. Deployment Plan (testnet→mainnet; staging; rollback)  
19. Legal/Disclosure Requirements (what must be shown to users)  
20. Appendices (State machines, formulas, event schemas)

---

## High-Level Summary (LOCKED)

### A) What YieldLoop IS
YieldLoop is a **DeFi spot trading + arbitrage product** on **BNB Chain** where users:
1) deposit **USDT (BEP-20)** (minimum 250 USDT)
2) choose allocation weights between **BTCB / ETH / BNB / XRP / SOL**
3) choose a compounding mode:
   - 100% compound
   - 50/50 compound/collect
   - 100% collect
4) choose reward currency:
   - claim rewards in **USDT** (20% performance fee on distributed profits)
   - claim rewards in **LOOP** (17.5% performance fee on distributed profits)
5) optionally lock their Position (VaultOS) from **30 days up to 25 years** for improved redemption bonus cap
6) at any time may:
   - deposit more
   - request close
   - withdraw all funds not currently in trade
   - withdraw remaining funds as trades close

YieldLoop runs:
- **spot trading** within allocation constraints
- **opportunistic arbitrage between PCS and BiSwap** when thresholds are met

Profit handling is **strictly deterministic**:
- Only closed trade profit counts
- Fees are only taken on distributed profit
- Rewards are only issued from verified net profit events

LOOP is:
- minted only from verified net profit events
- minted at a monotonic, non-decreasing FloorPrice
- redeemable to USDT with a reserve-gated bonus tier system

System Reserve (SR):
- funded from performance fee split
- backs redemption and floor logic
- forces redemption bonus to collapse to 1.000 when reserve health is weak

---

### B) What YieldLoop is NOT (Hard Exclusions)
YieldLoop v1 MUST NOT include:
- LP / farming / liquidity provision (any form)
- lending / borrowing
- leverage
- derivatives/perps/options
- flash loans
- external automated routers as a dependency (no Krystal dependency)
- any “yield smoothing” or fake reward emission
- any minting of LOOP not tied to verified net profit events

---

### C) Key Outputs (What the Dev Must Deliver)
The developer must deliver a complete production system including:

**Smart Contracts**
- Vault/Position accounting
- Fee routing pools
- LOOP minting and supply controls
- System Reserve accounting
- FloorPrice engine
- Redemption engine (bonus tiers + queue)
- VaultOS lock enforcement
- Event emission for every critical state change
- Admin roles + timelock + emergency pause

**Off-Chain Automation**
- keeper/bot execution engine for spot + arbitrage
- pricing/quote ingestion for PCS + BiSwap
- monitoring + alerting
- monthly email notification engine

**Frontend DApp**
- deposit / configure / confirm / acknowledge risks
- live vault states and balances
- allocation display and weights
- compounding mode selector
- reward currency selector + fee disclosure
- claim USDT / claim LOOP
- redeem LOOP → USDT
- lock selection interface
- close request flow
- transaction history + logs

---

### D) Design Philosophy (Non-Optional)
This build must follow these principles:
- **No ambiguity**
- **No silent defaults**
- **No dev interpretation**
- **No hidden risk**
- **Everything measurable**
- **Everything logged**
- **Everything state-machine based**
- **Everything reversible only through defined rules**

---

# 1. System Requirements (Hard Rules / Non-Negotiables)

This section is the absolute rulebook. Anything conflicting with this section is invalid. The developer MUST implement these as explicit checks, state rules, and/or contract guards.

---

## 1.1 Chain / Network Requirements (Locked v1)

### 1.1.1 Blockchain
- YieldLoop v1 MUST deploy on **BNB Chain (BSC / EVM)**.
- All contracts MUST verify correct chain ID at runtime where applicable.

### 1.1.2 Token Standard
- All assets MUST be BEP-20 / ERC-20 compatible tokens.

### 1.1.3 Stablecoin Deposit Asset (Locked)
- Deposit token MUST be:
  - **USDT (BEP-20)** ONLY
- Deposits of any other asset MUST revert.
- No native BNB deposits in v1.

---

## 1.2 DEX Venue Requirements (Locked v1)

### 1.2.1 Allowed DEX Venues
YieldLoop v1 venue list is locked to:
- PancakeSwap (PCS)
- BiSwap

### 1.2.2 Venue Use Policy
- Spot trading MAY execute on PCS and/or BiSwap.
- Arbitrage MAY execute ONLY between PCS ↔ BiSwap.
- No other DEX venues permitted in v1.

### 1.2.3 Venue Governance
- Adding a new DEX venue is NOT permitted in v1.
- A future version may add venues ONLY via:
  - new audited deployment
  - timelock change window
  - documented upgrade announcement
  - version increment

---

## 1.3 Allowed Assets & Allocation Rules (Locked v1)

### 1.3.1 Allocation Assets
The only permitted allocation assets are:
- BTCB
- ETH
- BNB
- XRP
- SOL

### 1.3.2 Allocation Weights Encoding (Hard)
- All allocation weights MUST be stored in **basis points** (BPS)
- Total allocation MUST equal exactly:
  - **10000 BPS**
- Any config where sum != 10000 MUST revert.

### 1.3.3 Default Allocation (Locked)
Default allocation weights MUST be:
- BTCB: 3000 BPS (30%)
- ETH:  2500 BPS (25%)
- BNB:  2500 BPS (25%)
- XRP:  1000 BPS (10%)
- SOL:  1000 BPS (10%)

### 1.3.4 Allocation Enforcement
Execution MUST respect allocation policy:
- spot trading and arb execution MUST NOT create a long-term allocation drift beyond:
  - **MaxDriftBPS = 250 BPS per asset (2.5%)** default
- if drift exceeds MaxDriftBPS:
  - system MUST prioritize rebalancing back within drift limits
- drift rules apply only to “free holdings”
  - active execution may temporarily drift until settlement completes

---

## 1.4 Strategy Requirements (Locked v1)

### 1.4.1 Strategy Classes (Only Two)
YieldLoop v1 MUST support ONLY:

**Strategy A — Spot Trading**
- swaps within approved asset list + USDT

**Strategy B — Cross-DEX Arbitrage**
- PCS ↔ BiSwap only

### 1.4.2 Hard Exclusions (Must Be Enforced)
YieldLoop v1 MUST NOT include:
- liquidity provision (LP), farming, yield farming
- lending, borrowing, credit loops
- leverage/margin
- perps/options/derivatives
- flash loans
- external strategy manager dependency (e.g., Krystal)
- “profit smoothing” / “APR targeting” / artificial emissions

These exclusions MUST be enforced as:
- no code paths
- no UI toggles
- no config switches enabling them

---

## 1.5 Profit Definition + No Fake Accounting (Hard)

### 1.5.1 No Unrealized Profit
The system MUST NOT:
- credit rewards from unrealized gains
- mint LOOP from unrealized gains
- claim performance before close

### 1.5.2 Profit Event Definition (Hard)
A Profit Event exists ONLY when:
- a trade/arb execution is CLOSED, AND
- NetProfitUSDT > 0

### 1.5.3 Net Profit Formula (Hard)
For every execution close:

NetProfitUSDT =
  EndValueUSDT
- StartValueUSDT
- GasCostUSDT
- DEXFeeUSDT
- SlippageUSDT
- MEVLossUSDT

If any term cannot be computed deterministically:
- assume worst case
- if still uncertain, classify as **Non-Profit Event**
- do not mint LOOP
- do not charge fee
- do not credit rewards

---

## 1.6 Performance Fee Requirements (Locked)

### 1.6.1 Fee Applies ONLY to Distributed Profit
Fee MUST be applied ONLY to:
- **DistributableProfitUSDT**

Fee MUST NOT be applied to:
- principal deposits
- compounded profit
- unrealized profit
- losses

### 1.6.2 Fee Rates (Locked)
- If user chooses reward currency = USDT:
  - FeeRate = **20%**
- If user chooses reward currency = LOOP:
  - FeeRate = **17.5%**

Fee changes are NOT permitted in v1.

### 1.6.3 Fee Split (Locked Defaults)
Fee split defaults (total 100%):
- Dev/Ops: 25%
- Marketing/Partnerships/Onboarding: 25%
- System Reserve (SR): 25%
- LoopLabs: 25%

Fee split MUST be stored as:
- 4 integer BPS values (sum = 10000)

### 1.6.4 Fee Split Changes (Allowed With Limits)
Fee split MAY be changed ONLY if:
- performed by authorized governance/admin roles
- timelock delay has completed
- new split still sums to exactly 10000 BPS

Fee split change MUST emit an event:
- FeeSplitUpdated(oldSplit, newSplit, effectiveTimestamp)

---

## 1.7 LOOP Token Requirements (Hard)

### 1.7.1 LOOP is Not Governance
- LOOP grants NO governance rights.
- Governance is not part of LOOP token.

### 1.7.2 Minting Constraints (Hard)
LOOP MUST be minted ONLY when:
- Profit Event = TRUE
- reward currency = LOOP

LOOP MUST NOT be minted for:
- deposits
- marketing
- referrals
- losses
- admin distributions
- emissions

### 1.7.3 Mint Price Constraint (Hard)
MintPrice MUST equal:
- MintPrice = FloorPrice

No exception.
No discounts.
No floating mint prices.

### 1.7.4 Mint Formula (Hard)
LoopMinted = UserNetProfitUSDT / FloorPrice

---

## 1.8 System Reserve Requirements (Hard)

### 1.8.1 SR Funding Rule
SR MUST be funded ONLY from:
- performance fees routed by fee split

ReserveSplit default:
- 25% of FeeUSDT

### 1.8.2 SR Asset Type (Locked v1)
SR MUST hold:
- USDT only

SR MUST NOT be invested in v1.

### 1.8.3 SR Usage (Locked v1)
SR may be used ONLY for:
- LOOP redemption payouts

No other usage is permitted in v1.

---

## 1.9 Floor Price Requirements (Hard)

### 1.9.1 Monotonic Floor Rule
FloorPrice MUST be monotonic:
- FloorPrice_next >= FloorPrice_current

FloorPrice MUST never decrease.

### 1.9.2 Floor Update Epoch (Locked)
Floor updates occur:
- monthly
- at first settlement epoch after **00:00 UTC on the 1st of each month**

### 1.9.3 Floor Calculation Rules
Define:
- SR_USDT
- LOOP_Supply
- TargetCR

Locked default:
- TargetCR = 1.05

CandidateFloor =
  SR_USDT / (LOOP_Supply * TargetCR)

FloorPrice_next =
  max(FloorPrice_current, CandidateFloor)

---

## 1.10 Redemption Requirements (Hard)

### 1.10.1 Base Redemption
All LOOP redemptions MUST pay at least:
- Base multiplier = 1.000

RedeemUSDT_base = LOOP_Redeemed * FloorPrice

### 1.10.2 Bonus System (Unlocked Users)
Unlocked cap:
- MaxBonusUnlocked = 1.025

BonusMultiplier MUST be:
- tiered steps of 0.0075 (0.75%)

Unlocked allowed values:
- 1.0000
- 1.0075
- 1.0150
- 1.0225
- 1.0250

### 1.10.3 Reserve Gating (Hard)
If a redemption would push reserve health below safety:
- bonus MUST collapse to 1.000
- redemption MUST queue if still unsafe

Minimum safety collateralization:
- MinCR = 1.00 (Locked)

---

## 1.11 VaultOS Lock Requirements (Hard)

### 1.11.1 Lock Durations (Locked Menu)
Allowed durations:
- 30 days
- 90 days
- 180 days
- 1 year
- 2 years
- 3 years
- 5 years
- 10 years
- 25 years

### 1.11.2 Lock Behavior
- Lock increases redemption bonus cap only
- Lock does not affect FloorPrice
- Lock cannot be shortened
- Early exit is NOT allowed in v1

### 1.11.3 Lock Cap Schedule (Locked)
Unlocked:
- cap = 1.0250

Locked caps:
- 30d  -> 1.0250
- 90d  -> 1.0275
- 180d -> 1.0300
- 1y   -> 1.0350
- 2y   -> 1.0400
- 3y   -> 1.0450
- 5y+  -> 1.0500
- 10y  -> 1.0500
- 25y  -> 1.0500

Hard max:
- 1.0500

---

## 1.12 Withdraw / Close Requirements (Hard)

### 1.12.1 Withdraw Availability Rule
User may withdraw:
- all funds not currently in active execution/trade immediately

### 1.12.2 Close Request Rule
When user requests close:
- system MUST stop opening new trades for that Position
- system MUST unwind as trades close
- once all trades closed:
  - full remaining balance becomes withdrawable

### 1.12.3 No Forced Lock of Principal
YieldLoop MUST NOT lock principal except:
- while it is actively in a trade execution window
- OR user explicitly uses VaultOS lock option (which affects redemption cap, not principal withdrawal)

---

## 1.13 Notification Requirements (Hard)

### 1.13.1 Monthly Contact Emails (Required)
System MUST send monthly email summary:
- vault value
- allocation weights
- lock status and remaining time
- claimable balances (USDT/LOOP)
- redemption floor and current bonus tier
- security reminders

### 1.13.2 Email Reliability Requirements
- Send attempts must retry
- Failures must log
- User can opt out ONLY of marketing content, not security notices

---

## 1.14 “Dev Must Not Ask Founder” Requirement (Hard)
This builder document is the source of truth.
Developer must implement every decision exactly as written.

If a missing decision exists, developer MUST:
- flag it as a blocker
- stop implementation of that module until a written patch is added to this doc

There MUST NOT be:
- silent assumptions
- “reasonable defaults”
- “we’ll decide later”
- “dev preference”

---

# 2. User Vault Model (Positions, Allocation, Compounding, Rewards)

This section defines the complete vault/position behavior and required state machines.
The developer MUST implement exactly these vault objects, states, permissions, and user flows.

---

## 2.1 Core Design (Hard)
YieldLoop uses:
- per-user Vaults
- per-deposit Positions
- deterministic accounting per Position

Key rule:
- Positions are isolated accounting units.
- Positions MUST NOT cross-subsidize each other.
- Vault may aggregate views for UX, but state and accounting are Position-based.

---

## 2.2 Vault Object (Per Wallet)

### 2.2.1 Vault Identifier
- VaultID = keccak256(userAddress)

### 2.2.2 Vault Properties (Stored)
Each Vault MUST store:
- userAddress
- createdTimestamp
- positionCount
- totalPrincipalUSDT (sum of all Position principals, excluding claim balances)
- totalClaimableUSDT (aggregate of all positions)
- totalClaimableLOOP (aggregate of all positions)
- vaultStatus

### 2.2.3 Vault Status (Enum)
VaultStatus MUST be:
- ACTIVE
- FROZEN (admin emergency)
- SUNSET (protocol sunset state)

Default:
- ACTIVE

---

## 2.3 Position Object (Per Deposit)

### 2.3.1 Position Identifier
- PositionID = sequential integer index within Vault

### 2.3.2 Position Required Properties (Stored)
Each Position MUST store:

**Identity**
- VaultID
- PositionID
- createdTimestamp
- lastUpdatedTimestamp

**Deposit**
- principalUSDT (original principal)
- currentValueUSDT (computed view; not authoritative unless settlement snapshot)
- depositTxHash

**User Strategy Config**
- allocationBPS[5]  (BTCB, ETH, BNB, XRP, SOL; must sum 10000)
- compoundingMode   (A/B/C)
- rewardCurrency    (USDT or LOOP)

**Execution / Trade State**
- positionState (enum)
- openExecutionCount
- inTradeUSDT (USDT currently deployed in active execution)
- freeUSDT (USDT not in active execution)
- freeTokenBalances[5] (token balances held idle per asset)
- lastExecutionTimestamp

**Claim Accounting**
- claimableUSDT
- claimableLOOP
- lifetimeNetProfitUSDT
- lifetimeFeesUSDT

**Close Controls**
- closeRequested (bool)
- closeRequestTimestamp

**Lock Config**
- lockEnabled (bool)
- lockDurationSeconds
- lockStartTimestamp
- lockEndTimestamp
- lockBonusCapMultiplier (scaled int e.g. 10250 = 1.0250)

**Risk / Compliance**
- riskDisclosureAckTimestamp
- feeDisclosureAckTimestamp
- strategyDisclosureAckTimestamp
- lastConfigChangeTimestamp

---

## 2.4 Position State Machine (Hard)

### 2.4.1 PositionState Enum (Locked)
PositionState MUST be one of:

0) CREATED  
1) FUNDED  
2) CONFIGURED  
3) READY  
4) EXECUTING_SPOT  
5) EXECUTING_ARB  
6) SETTLING  
7) PAUSED  
8) CLOSE_REQUESTED  
9) CLOSING  
10) CLOSED  

### 2.4.2 State Transition Rules (Hard)

**CREATED → FUNDED**
- occurs immediately when deposit succeeds

**FUNDED → CONFIGURED**
- occurs when user completes required configuration:
  - allocation weights
  - compounding mode
  - reward currency
  - disclosures acknowledged

**CONFIGURED → READY**
- occurs immediately after config validity checks pass

**READY → EXECUTING_SPOT**
- only if spot execution thresholds pass
- only if closeRequested = FALSE
- only if not paused

**READY → EXECUTING_ARB**
- only if arbitrage thresholds pass
- only if closeRequested = FALSE
- only if not paused

**EXECUTING_* → SETTLING**
- occurs after execution closes

**SETTLING → READY**
- if closeRequested = FALSE and position still active

**SETTLING → CLOSE_REQUESTED**
- if closeRequested = TRUE

**READY → CLOSE_REQUESTED**
- if user requests close

**CLOSE_REQUESTED → CLOSING**
- when unwinding begins or execution completes

**CLOSING → CLOSED**
- once no active executions remain and balances are fully free

**ANY → PAUSED**
- if safety engine pauses the position

**PAUSED → READY**
- only when unpause conditions are met AND closeRequested=FALSE

**PAUSED → CLOSE_REQUESTED**
- if user requests close while paused

**CLOSE_REQUESTED / CLOSING cannot return to READY**
- once close requested, no new trades may start

---

## 2.5 Deposit Flow (Hard)

### 2.5.1 Deposit Requirements
- deposit asset: USDT only
- minimum: 250 USDT
- user must approve ERC20 allowance
- contract must transferFrom user to Vault contract

### 2.5.2 Deposit Behavior
On deposit:
- create Position
- set freeUSDT += depositAmount
- set principalUSDT += depositAmount
- state transitions CREATED→FUNDED

### 2.5.3 Deposit UI Requirements
UI MUST display before deposit confirmation:
- minimum deposit requirement
- that YieldLoop is spot trading + arbitrage only
- that rewards are not guaranteed
- that losing wallet seed loses funds

---

## 2.6 Configuration Flow (Hard)

### 2.6.1 Required Config Inputs
User must set:

1) Allocation weights:
- input either sliders or numeric BPS fields
- sum must equal 100%

2) Compounding mode:
- Mode A: 100% compound
- Mode B: 50/50
- Mode C: 100% collect

3) Reward currency:
- USDT (20% fee on distributed profit)
- LOOP (17.5% fee on distributed profit)

4) Optional lock selection:
- OFF by default
- else choose from locked menu

### 2.6.2 Mandatory Disclosures (Hard)
User MUST acknowledge:
- strategy disclosure
- fee disclosure
- risk disclosure

Acks must be stored on-chain as timestamps per Position:
- riskDisclosureAckTimestamp
- feeDisclosureAckTimestamp
- strategyDisclosureAckTimestamp

Without acks:
- execution cannot begin

### 2.6.3 Default Config (Hard)
If user does not change defaults, defaults MUST apply:

Allocation (BPS):
- BTCB 3000
- ETH  2500
- BNB  2500
- XRP  1000
- SOL  1000

Compounding:
- Mode B (50/50)

Reward currency:
- LOOP

Lock:
- OFF

---

## 2.7 Config Change Rules (Hard)

### 2.7.1 Config Change Allowed Window
User may change config ONLY if:
- openExecutionCount = 0
- positionState is READY or PAUSED
- closeRequested = FALSE

### 2.7.2 Config Change Restrictions
User may update:
- allocation weights
- compounding mode
- reward currency

User may NOT:
- shorten or remove lock once enabled
- change lock duration after lockStartTimestamp

### 2.7.3 Config Change Compliance Gate
Any config change MUST require:
- reconfirm disclosures (“refresh acknowledgment”)
- store lastConfigChangeTimestamp

Reason:
- compliance control and explicit consent

---

## 2.8 Add Deposit Flow (Hard)

User may add funds at any time with two options:

### 2.8.1 Option 1 — Add to Existing Position (Default = OFF)
This option is NOT allowed in v1 to reduce complexity.
Locked v1 rule:
- Add deposit always creates a NEW Position.

### 2.8.2 Option 2 — Create New Position (Locked v1 Default)
- every deposit creates a new PositionID
- new Position inherits default config (Section 2.6.3)
- user may configure independently

This prevents:
- weird blended accounting
- strategy drift
- lock intermixing
- execution entanglement

---

## 2.9 Withdraw Flow (Hard)

### 2.9.1 Withdrawable Balance Rules
User can withdraw ONLY:
- freeUSDT
- freeTokenBalances (if system holds them idle and user requests)
Default:
- system SHOULD unwind idle tokens to USDT before withdrawal unless user requests otherwise

Locked v1 rule:
- withdrawals are in USDT only

### 2.9.2 Withdrawal Limits
- user may withdraw freeUSDT at any time
- if user tries to withdraw funds inTradeUSDT:
  - revert
  - UI must show message:
    - “Funds currently in trade. Request close to unwind.”

### 2.9.3 Withdrawal Fees
YieldLoop v1 has:
- NO withdrawal fee
- NO principal fee

Only performance fee exists on distributed profit.

---

## 2.10 Close Position Flow (Hard)

### 2.10.1 Close Request
User may request close at any time.

Effect:
- closeRequested = TRUE
- closeRequestTimestamp stored
- PositionState transitions to CLOSE_REQUESTED (or remains if already closing)

### 2.10.2 Close Execution Rules
Once closeRequested = TRUE:
- no new trades may start
- open executions must finish normally
- once execution closes, system transitions to CLOSING
- system unwinds all token holdings back to USDT (unless prevented by guardrails)

### 2.10.3 Closing Completion
Position becomes CLOSED only when:
- openExecutionCount = 0
- inTradeUSDT = 0
- all token balances unwound to USDT
- freeUSDT is 100% of remaining value

### 2.10.4 Withdraw at Close
Upon CLOSED:
- user may withdraw 100% remaining balance immediately

---

## 2.11 Claim Flows (Hard)

Claim actions are separate from withdrawal of principal.

### 2.11.1 Claim USDT
User can claim:
- claimableUSDT at any time
Claim reduces claimableUSDT and transfers USDT to wallet.

### 2.11.2 Claim LOOP
User can claim:
- claimableLOOP at any time
Claim reduces claimableLOOP and transfers LOOP to wallet.

### 2.11.3 Claim Balance Safety
Claim balances MUST be protected from:
- execution engine spending
- allocation routing
They are “out of strategy” and belong to user.

---

## 2.12 Redemption Flow (Hard)

Redemption converts LOOP to USDT under Redemption Engine rules.

### 2.12.1 Redeem Trigger
User calls:
- redeem(LOOP_amount)

### 2.12.2 Redeem Output (Locked)
Redeem output MUST always be:
- USDT only

### 2.12.3 Redeem Constraints
Redeem is governed by:
- FloorPrice
- BonusMultiplier tiers
- lock cap (if locked Position)
- reserve gating
- queue fallback

---

## 2.13 UI/UX Mandatory Screens (User Vault Layer)

UI MUST include these screens at minimum:

1) Landing / Connect Wallet
2) Deposit screen
3) Strategy config screen
4) Disclosures + acknowledgments screen
5) Vault dashboard (all positions)
6) Position details screen
7) Claim screen
8) Redeem screen
9) Lock selection screen (VaultOS)
10) Close position screen
11) Tx history / event log screen
12) Settings / disclosures re-read screen

---

## 2.14 Mandatory Events (On-Chain Logging)

Contracts MUST emit events for:

- VaultCreated(user, vaultId)
- PositionCreated(vaultId, positionId, amountUSDT)
- PositionConfigured(vaultId, positionId, allocationBPS, compoundingMode, rewardCurrency, lockEnabled, lockDuration)
- DisclosureAcknowledged(vaultId, positionId, disclosureType, timestamp)
- ConfigUpdated(vaultId, positionId, newConfig)
- CloseRequested(vaultId, positionId, timestamp)
- PositionStateChanged(vaultId, positionId, oldState, newState)
- ClaimUSDT(vaultId, positionId, amount)
- ClaimLOOP(vaultId, positionId, amount)
- RedeemLOOP(vaultId, positionId, loopAmount, usdtOut, bonusMultiplier)

---

# 3. Strategy Layer (Spot + PCS↔BiSwap Arbitrage)

This section defines exactly how the trading layer behaves. The developer MUST implement these strategies, triggers, thresholds, and fallback rules exactly as written.
No other strategies are permitted in v1.

---

## 3.1 Strategy Scope (Locked v1)

YieldLoop v1 supports ONLY:
1) Spot Trading Strategy
2) PCS↔BiSwap Cross-DEX Arbitrage Strategy

Hard exclusions:
- no LP
- no yield farming
- no lending
- no flash loans
- no triangular arbitrage (excluded v1)
- no non-PCS / non-BiSwap venues

---

## 3.2 Execution Model Overview

### 3.2.1 Keeper-Based Execution (Hard)
YieldLoop uses an off-chain keeper/bot to evaluate opportunities and submit on-chain execution calls.

Constraints:
- keeper/bot MUST be permissioned via approved executor role
- keeper/bot actions MUST be bounded by on-chain guardrails
- keeper/bot MUST NOT be able to withdraw funds
- keeper/bot MUST NOT be able to change user configs

### 3.2.2 Execution Windows
Execution windows are discrete.
Each execution window MUST:
- define start snapshot (StartValueUSDT)
- execute one strategy (spot OR arb, not both in same window)
- close with settlement snapshot (EndValueUSDT)
- output Profit Event or Non-Profit Event

### 3.2.3 Single-Strategy per Execution (Hard)
Each execution window MUST run:
- either Spot Trading execution
- OR Arbitrage execution

Not both.

---

## 3.3 Shared Constraints (Apply to ALL Trading)

These constraints apply to spot and arbitrage equally.

### 3.3.1 Approved Assets (Hard)
The execution engine may only trade:
- USDT
- BTCB
- ETH
- BNB
- XRP
- SOL

If any call attempts a different token:
- revert

### 3.3.2 Max Slippage (Locked Default)
MaxSlippagePerLegBPS = 30 BPS (0.30%)

Rule:
- If simulated execution slippage > 30 BPS:
  - execution MUST NOT occur

### 3.3.3 Max Gas Cost (Locked Default)
MaxGasPerExecutionUSDT = 1.25 USDT

Rule:
- If estimated gas cost > 1.25 USDT:
  - execution MUST NOT occur

### 3.3.4 Minimum Liquidity Depth (Locked Default)
MinimumLiquidityDepthUSDT = 50,000 USDT per pool

Rule:
- If pool liquidity depth does not meet threshold for proposed size:
  - proposed size MUST be reduced until threshold satisfied
  - if cannot satisfy -> abort execution

### 3.3.5 Execution Size Limits
Per Position execution limits:

- MaxExecutionSizePercent = 10% of free position value (Locked)
- MaxExecutionSizeUSDT = 750 USDT (Locked)

ActualExecutionSizeUSDT =
  min(freeValueUSDT * 0.10, 750)

The system MUST NOT exceed these limits.

### 3.3.6 CloseRequested Block (Hard)
If Position.closeRequested = TRUE:
- no new executions may start (spot or arb)
- only unwinds/closing actions may occur

### 3.3.7 Pause Block (Hard)
If PositionState = PAUSED:
- execution may not start
- only user withdraw/close allowed

---

## 3.4 Spot Trading Strategy (Strategy A)

### 3.4.1 Purpose
Spot strategy exists to:
- keep allocations near target weights
- capture basic profitable movements when opportunity exists
- never sacrifice gas efficiency

Spot trading is not “high frequency”.
It is conservative execution.

### 3.4.2 Spot Trade Trigger (Hard Rule)
Spot trade may execute ONLY if:

ExpectedNetProfitUSDT >= MinExpectedProfitSpotUSDT

Locked default:
- MinExpectedProfitSpotUSDT = 0.50 USDT

ExpectedNetProfitUSDT calculation MUST include:
- DEX fees
- gas cost
- slippage buffer
- MEV buffer

ExpectedNetProfitUSDT =
  ExpectedGrossProfitUSDT
- TotalDEXFeesUSDT
- GasCostUSDT
- SlippageBufferUSDT
- MEVBufferUSDT

### 3.4.3 Spot Venue Selection Rule
Spot trades MAY use:
- PCS
- BiSwap

Venue selection priority rule (Locked):
1) Highest expected net profit after all costs
2) Lowest slippage
3) Lowest gas route
4) Highest liquidity depth

### 3.4.4 Allocation Drift Rule
Spot trading MUST enforce allocation drift constraints:

Define:
- TargetAllocationBPS[5]
- CurrentAllocationBPS[5]
- MaxDriftBPS = 250

CurrentAllocationBPS is computed from:
- free token balances + free USDT
- plus in-trade amounts marked at entry values (conservative)

If any asset drift exceeds MaxDriftBPS:
- spot strategy MUST prioritize rebalancing that asset toward target

### 3.4.5 Spot Execution Pattern (Locked)
Spot strategy MUST execute as one of:

A) Rebalance trade:
- swap USDT -> underweight token
OR
- swap overweight token -> USDT

B) Profit capture trade:
- swap token -> USDT or USDT -> token
- only if expected net profit threshold is met

Spot strategy MUST NOT execute:
- speculative random trades
- trades violating drift constraints
- trades below profit threshold

---

## 3.5 Cross-DEX Arbitrage (Strategy B)

### 3.5.1 Purpose
Arbitrage exists to capture pricing inefficiencies between PCS and BiSwap.

### 3.5.2 Arbitrage Assets Allowed
Arbitrage may only involve:
- USDT and 1 approved token at a time

Meaning:
- USDT ↔ TOKEN on PCS, then TOKEN ↔ USDT on BiSwap
OR reverse

No multi-asset cycles.
No triangular paths.

### 3.5.3 Arbitrage Trigger Condition (Hard Rule)
Arbitrage executes ONLY if:

ExpectedNetProfitUSDT >= MinExpectedProfitArbUSDT

Locked default:
- MinExpectedProfitArbUSDT = 1.50 USDT

ExpectedNetProfitUSDT MUST include:
- all swap fees
- gas cost
- slippage buffer
- MEV buffer

Locked default buffers:
- SlippageBufferUSDT = 0.25 USDT
- MEVBufferUSDT = 0.50 USDT

### 3.5.4 Arbitrage Size Limits (Locked)
MaxArbSizePercent = 7.5% of free position value
MaxArbSizeUSDT = 500 USDT

ActualArbSizeUSDT =
  min(freeValueUSDT * 0.075, 500)

### 3.5.5 Arbitrage Venue Direction Rule (Locked)
The engine MUST:
- compute buy price and sell price from both DEXs
- choose direction with highest expected net profit

Direction options:
- Buy on PCS, sell on BiSwap
- Buy on BiSwap, sell on PCS

### 3.5.6 Arbitrage Execution Pattern (Locked)
Two-leg arbitrage MUST execute:
1) Swap USDT -> TOKEN on Venue A
2) Swap TOKEN -> USDT on Venue B

If both legs cannot execute safely:
- execution MUST abort

### 3.5.7 Mid-Execution Spread Collapse Handling (Hard)
If during execution:
- leg 1 executes but leg 2 quote deteriorates below profitability

Then:
- the system MUST attempt to execute leg 2 only if:
  - it does not violate slippage cap
  - it does not exceed gas cap
  - it prevents leaving the vault with an undesired exposure

If executing leg 2 violates constraints:
- do NOT execute leg 2
- mark exposure as temporary
- schedule controlled unwind into USDT ASAP

Unwind rules:
- unwind MUST obey slippage and gas ceilings
- if unwind cannot meet ceilings:
  - hold TOKEN until unwind becomes safe
  - block further executions for that Position until unwind completes

---

## 3.6 Pricing / Quote Engine Requirements (Hard)

### 3.6.1 Quote Sources
Quotes MUST be sourced from:
- PCS router quoting
- BiSwap router quoting

No third-party price oracle is required for v1 execution, but:
- sanity-check oracle MAY be used for protection (Section 11)

### 3.6.2 Quote Freshness Window
Quotes MUST be treated as stale after:
- QuoteTTLSeconds = 15 seconds (Locked)

If quote age > 15 seconds:
- execution MUST NOT occur

### 3.6.3 Quote Safety Margin
All quotes MUST incorporate safety margins:
- slippage tolerance = MaxSlippagePerLegBPS
- MEV buffer = MEVBufferUSDT (arb) / 0.25 USDT (spot)

---

## 3.7 MEV Handling Requirements (Hard)

### 3.7.1 Anti-MEV Routing (Required if available)
If a MEV-protected transaction relay is available:
- keeper MUST use it for arbitrage by default

If not available:
- MEVBufferUSDT MUST remain active and conservative

### 3.7.2 No Sandwich-Unsafe Trades
If a trade is identified as sandwich-risk-high:
- arbitrage MUST NOT execute
- spot trades MUST only execute if rebalancing is required

Definition:
- sandwich-risk-high if:
  - pool liquidity shallow OR
  - swap size large relative to pool OR
  - mempool conditions indicate heavy MEV activity

(Implementation: must be conservative; false negatives not allowed.)

---

## 3.8 Execution Failure Handling (Hard)

### 3.8.1 Execution Attempt Limit
For each Position:
- MaxFailedExecutionsPer10Min = 3 (Locked)

If exceeded:
- PositionState -> PAUSED automatically
- pauseReason stored

### 3.8.2 DEX Failure / Quote Failure
If PCS or BiSwap quotes fail:
- retry for up to 5 minutes
- if failure persists:
  - position enters PAUSED
  - notify user

### 3.8.3 Gas Spike Failure
If estimated gas > MaxGasPerExecutionUSDT for > 10 minutes:
- Position enters PAUSED
- bot continues to monitor but cannot trade

---

## 3.9 Required Event Logging (Trading Layer)

Contracts MUST emit:

- ExecutionStarted(vaultId, positionId, strategyType, venueA, venueB, inputAmountUSDT)
- ExecutionLeg(vaultId, positionId, legIndex, venue, tokenIn, tokenOut, amountIn, amountOut)
- ExecutionClosed(vaultId, positionId, strategyType, startValueUSDT, endValueUSDT, netProfitUSDT)
- ExecutionFailed(vaultId, positionId, strategyType, reasonCode)
- PositionPaused(vaultId, positionId, reasonCode, timestamp)
- PositionUnpaused(vaultId, positionId, timestamp)

Reason codes MUST be enumerated constants.
No free-text reason strings in contracts.

---

## 3.10 Strategy Summary (Dev Checklist)

Developer MUST implement:

✅ Spot Strategy:
- rebalancing enforcement
- profit threshold gating

✅ Arbitrage Strategy:
- direction selection
- profit threshold gating
- two-leg execution only
- unwind fallback behavior

✅ Shared Guardrails:
- slippage cap
- gas cap
- liquidity depth
- size caps
- pause conditions
- closeRequested block
- quote TTL

Anything outside these rules is forbidden in v0.8.0

---

# 4. Profit Accounting + Settlement (No Fake Profit)

This section defines the ONLY allowed accounting method and the ONLY allowed settlement sequence.
The system MUST behave deterministically and must be fully auditable from on-chain logs.

---

## 4.1 Core Accounting Rule (Hard)
YieldLoop ONLY recognizes profit when an execution closes.

The system MUST NOT:
- assume profit
- estimate profit for rewards
- charge fees before close
- “smooth” returns
- fake APR payouts

---

## 4.2 Execution Snapshot Requirements (Hard)

Every execution window MUST create 2 snapshots:

### 4.2.1 Start Snapshot
Start snapshot MUST include:
- StartTimestamp
- Position free balances at start:
  - freeUSDT
  - freeTokenBalances[BTCB,ETH,BNB,XRP,SOL]
- Position trade allocation for this execution:
  - inputAmountUSDT
  - strategyType (SPOT or ARB)
  - venues used (PCS/BiSwap)
- StartValueUSDT (computed)

### 4.2.2 End Snapshot
End snapshot MUST include:
- EndTimestamp
- free balances after execution
- execution outputs per leg (token in/out, amount)
- EndValueUSDT (computed)

---

## 4.3 Value Computation Standard (Hard)

### 4.3.1 Value Basis
USDT-equivalent value MUST be computed using:
- actual swap outputs from router receipts
- plus current on-chain quote value for remaining idle token balances (conservative)
- NOT from external APIs

### 4.3.2 Conservative Valuation Rule
If token valuation is uncertain:
- apply worst-case quote within slippage cap
- if still uncertain -> treat value as lower

This protects from false profit classification.

---

## 4.4 Cost Accounting (Hard)

Every execution MUST compute:

- GasCostUSDT
- DEXFeeUSDT
- SlippageUSDT
- MEVLossUSDT

### 4.4.1 GasCostUSDT
GasCostUSDT MUST be:
- actual gas used * gas price
- converted to USDT using on-chain BNB/USDT price quote
- stored per execution

### 4.4.2 DEXFeeUSDT
DEXFeeUSDT MUST be computed from:
- router fee parameters OR
- implied fee from input/output delta
- must be stored

### 4.4.3 SlippageUSDT
SlippageUSDT MUST be computed as:
ExpectedOut (quote at execution start) - ActualOut

Converted to USDT basis and stored.

### 4.4.4 MEVLossUSDT
MEVLossUSDT is:
- any detectable loss between pre-signed expected execution and outcome due to frontrunning/sandwich
- if not detectable -> set 0
- do NOT guess

---

## 4.5 Net Profit Formula (Hard)

NetProfitUSDT =
  EndValueUSDT
- StartValueUSDT
- GasCostUSDT
- DEXFeeUSDT
- SlippageUSDT
- MEVLossUSDT

Rules:
- If NetProfitUSDT <= 0:
  - ProfitEvent = FALSE
- If NetProfitUSDT > 0:
  - ProfitEvent = TRUE

---

## 4.6 Settlement Trigger (Hard)

Settlement occurs ONLY when:
- execution closes
- NetProfitUSDT has been computed

Settlement MUST execute immediately after execution close.

Settlement MUST NOT be delayed or batched across executions.

---

## 4.7 Settlement Sequence (Hard)

For each Profit Event:

1) Calculate DistributableProfitUSDT
2) Calculate FeeUSDT
3) Route FeeUSDT into fee pools
4) Credit user claim (USDT or LOOP)
5) Apply compound reinvest amount into free balances
6) Update lifetime counters
7) Emit Settlement events

If any step fails:
- entire settlement MUST revert
- execution is considered failed
- position MUST pause after max failure threshold

---

## 4.8 Distributable Profit Rules (Hard)

Let:
- NP = NetProfitUSDT

### Mode A — 100% Compound
- DistributableProfitUSDT = 0
- CompoundProfitUSDT = NP

### Mode B — 50% Compound / 50% Collect
- DistributableProfitUSDT = NP * 0.50
- CompoundProfitUSDT = NP * 0.50

### Mode C — 100% Collect
- DistributableProfitUSDT = NP
- CompoundProfitUSDT = 0

---

## 4.9 Fee Rules (Hard)

Fee applies ONLY to distributable profit.

Let:
- DP = DistributableProfitUSDT

### Reward Currency = USDT
- FeeRate = 0.20
- FeeUSDT = DP * 0.20

### Reward Currency = LOOP
- FeeRate = 0.175
- FeeUSDT = DP * 0.175

If DP = 0:
- FeeUSDT = 0

---

## 4.10 Fee Routing (Hard)

Let:
- F = FeeUSDT

Default splits:
- Dev/Ops:      25% (2500 BPS)
- Marketing:    25% (2500 BPS)
- SystemReserve:25% (2500 BPS)
- LoopLabs:     25% (2500 BPS)

Routing outputs:
- DevOpsUSDT    = F * 0.25
- MarketingUSDT = F * 0.25
- ReserveUSDT   = F * 0.25
- LoopLabsUSDT  = F * 0.25

Routing MUST:
- update pool balances
- emit FeeRouted event

---

## 4.11 User Reward Credit (Hard)

Let:
- UserNetProfitUSDT = DP - F

If reward currency = USDT:
- Position.claimableUSDT += UserNetProfitUSDT

If reward currency = LOOP:
- LoopMinted = UserNetProfitUSDT / FloorPrice
- Position.claimableLOOP += LoopMinted

---

## 4.12 Compound Reinvestment Handling (Hard)

CompoundProfitUSDT MUST remain inside the Position and become available to future executions.

Implementation requirement:
- compound profit must increase freeUSDT OR rebalance holdings per allocation router

Locked v1 rule:
- compound profit is held as USDT until next allocation rebalance execution

So:
- freeUSDT += CompoundProfitUSDT

No forced conversion during settlement.

---

## 4.13 Loss Event Handling (Hard)

If NetProfitUSDT <= 0:
- no fee is charged
- no LOOP minted
- no claim balances increase
- compound does not increase
- lifetime loss may be tracked for analytics

The system MUST emit:
- ExecutionClosed with netProfitUSDT <= 0
- SettlementSkipped(reasonCode = NON_PROFIT_EVENT)

---

## 4.14 Lifetime Accounting (Hard)

Each Position MUST track:
- lifetimeNetProfitUSDT (sum of NetProfitUSDT where >0)
- lifetimeFeesUSDT (sum of FeeUSDT)
- totalExecutions
- totalProfitEvents
- totalLossEvents

These MUST update on every execution close.

---

## 4.15 Settlement Events (Mandatory)

Contracts MUST emit:

- ProfitEventDetected(vaultId, positionId, netProfitUSDT)
- SettlementExecuted(
    vaultId,
    positionId,
    compoundingMode,
    rewardCurrency,
    distributableProfitUSDT,
    compoundProfitUSDT,
    feeRate,
    feeUSDT,
    userNetProfitUSDT,
    loopMinted,
    floorPrice
  )
- FeeRouted(vaultId, positionId, devOpsUSDT, marketingUSDT, reserveUSDT, loopLabsUSDT)
- SettlementSkipped(vaultId, positionId, reasonCode)

Reason codes MUST be enums:
- NON_PROFIT_EVENT
- MISSING_COST_DATA
- SNAPSHOT_INVALID
- EXECUTION_ABORTED
- OTHER_GUARDRAIL_FAILURE

---

## 4.16 Settlement Integrity Requirements (Hard)

### 4.16.1 No Partial Settlement
Settlement MUST be atomic.
Either:
- all settlement steps succeed
or
- entire settlement reverts

### 4.16.2 No Silent Rounding Loss
All math MUST use:
- integer arithmetic
- consistent scaling factors
- explicit rounding direction

Locked rounding rules:
- Fees round DOWN (favor user)
- LOOP mint rounds DOWN (favor system reserve safety)
- Redemption payout rounds DOWN (favor reserve safety)

---

## 4.17 Dev Checklist (Settlement Layer)

Developer MUST implement:

✅ Execution start/end snapshots  
✅ Deterministic cost accounting  
✅ Hard profit formula  
✅ Settlement only on close  
✅ Fee applied only to distributable profit  
✅ Fee routing to pools  
✅ Claim balance credits  
✅ Loss events do not mint/charge  
✅ Full event schema emission  
✅ Atomic settlement (no partial writes)

---

# 5. Fee System (USDT vs LOOP Claim Rates + Routing Splits)

This section defines all fee behavior and pool handling. There are no other fee types in v1.
The developer MUST implement fees exactly as written.

---

## 5.1 Fee Types (Locked v1)

YieldLoop v1 has ONLY:
- Performance Fee on distributed profit

YieldLoop v1 MUST NOT include:
- deposit fee
- withdrawal fee
- management fee
- subscription fee
- hidden swap spread fees
- “gas pool fee”
- any off-ledger extraction

---

## 5.2 Performance Fee Trigger (Hard)

Performance fee is charged ONLY when:
- Settlement occurs on Profit Event
- DistributableProfitUSDT > 0

If:
- DistributableProfitUSDT = 0
Then:
- FeeUSDT = 0

---

## 5.3 Performance Fee Rates (Locked)

Reward currency determines fee rate:

- If user selects **USDT rewards**:
  - FeeRate = 20% (0.20)

- If user selects **LOOP rewards**:
  - FeeRate = 17.5% (0.175)

Fee rate MUST be:
- stored as constant
- not admin-changeable in v1

---

## 5.4 Fee Calculation (Hard)

Let:
- DP = DistributableProfitUSDT

FeeUSDT = DP * FeeRate

UserNetProfitUSDT = DP - FeeUSDT

No other deductions are allowed.

---

## 5.5 Fee Splits (Locked Default + Governance Change Rules)

### 5.5.1 Split Buckets (Exactly Four)
Fee split MUST always route into exactly these four buckets:

1) Dev/Ops Pool  
2) Marketing/Partnerships/Onboarding Pool  
3) System Reserve (SR)  
4) LoopLabs Pool  

No additional fee buckets may be introduced in v1.

### 5.5.2 Default Split (Locked)
Default split is 25/25/25/25:

- Dev/Ops: 2500 BPS
- Marketing: 2500 BPS
- System Reserve: 2500 BPS
- LoopLabs: 2500 BPS

Split MUST sum to exactly 10000 BPS.

### 5.5.3 Split Governance Change (Allowed With Limits)
Fee split MAY be changed ONLY if:

- authorized governance/admin role initiates change
- timelock completes
- new split still sums to 10000 BPS
- change emits FeeSplitUpdated event

Fee split changes MUST be stored as:
- devOpsSplitBPS
- marketingSplitBPS
- reserveSplitBPS
- loopLabsSplitBPS

---

## 5.6 Fee Routing Execution (Hard)

Let:
- F = FeeUSDT

Compute:

DevOpsUSDT    = floor(F * devOpsSplitBPS / 10000)
MarketingUSDT  = floor(F * marketingSplitBPS / 10000)
ReserveUSDT    = floor(F * reserveSplitBPS / 10000)
LoopLabsUSDT   = F - (DevOpsUSDT + MarketingUSDT + ReserveUSDT)

Rounding rules:
- floors applied to first three buckets
- remainder assigned to LoopLabs bucket
(this prevents under-routing and ensures full conservation of F)

Routing must:
- update balances
- emit event with routed values

---

## 5.7 Fee Pool Custody Requirements (Hard)

### 5.7.1 Custody Model (Locked)
All fee pools MUST exist as contract-controlled balances.

No pool funds may be held by keeper bots.

### 5.7.2 Pool Denomination (Locked v1)
All pools are denominated in:
- USDT only

The system MUST NOT swap fee pool USDT into other assets in v1.

---

## 5.8 Fee Pool Withdrawal Rules (Hard)

### 5.8.1 Dev/Ops Pool Withdrawal
- may be withdrawn only by DevOps role (multi-sig)
- withdrawal emits DevOpsWithdrawn(amount, to)

### 5.8.2 Marketing Pool Withdrawal
- may be withdrawn only by Marketing role (multi-sig)
- withdrawal emits MarketingWithdrawn(amount, to)

### 5.8.3 LoopLabs Pool Withdrawal
- may be withdrawn only by LoopLabs role (multi-sig)
- withdrawal emits LoopLabsWithdrawn(amount, to)

### 5.8.4 System Reserve Withdrawal
System Reserve MUST NOT be withdrawn for discretionary spending in v1.
System Reserve is used ONLY by Redemption Engine payouts.

The only allowed SR outflow is:
- redemption payouts to users
- plus queue payouts (if queue used)

Any attempt to withdraw SR to admin addresses MUST revert.

---

## 5.9 Fee Disclosure Requirements (Hard)

UI MUST display, before user final confirmation:

If reward currency = USDT:
- “Performance fee: 20% on distributed profits”

If reward currency = LOOP:
- “Performance fee: 17.5% on distributed profits”

Also display:
- split breakdown 25/25/25/25 (or current split if changed)
- “No fees on principal deposit”
- “No withdrawal fees”

---

## 5.10 Events (Mandatory)

Contracts MUST emit:

- FeeSplitUpdated(oldDevBPS, oldMktBPS, oldResBPS, oldLabsBPS, newDevBPS, newMktBPS, newResBPS, newLabsBPS, effectiveTimestamp)
- FeeCharged(vaultId, positionId, feeRate, feeUSDT)
- FeeRouted(vaultId, positionId, devOpsUSDT, marketingUSDT, reserveUSDT, loopLabsUSDT)
- PoolBalanceUpdated(poolType, newBalanceUSDT)

PoolType enum MUST include:
- DEVOPS
- MARKETING
- RESERVE
- LOOPLABS

---

# 6. LOOP Token System (Minting, Claims, Supply Constraints)

This section defines LOOP’s full behavior and its integration with YieldLoop.
LOOP MUST function as a redeem token, minted only from verified profits.

---

## 6.1 LOOP Token Properties (Hard)

- Name: LOOP
- Symbol: LOOP
- Decimals: 18
- Standard: ERC-20/BEP-20 compatible

LOOP MUST be minted only by LOOP Mint Engine / authorized contract role.

---

## 6.2 LOOP Supply Rules (Hard)

### 6.2.1 No Fixed Supply
LOOP has no fixed max supply in v1.

### 6.2.2 Minting Constraints
LOOP MUST be minted ONLY when:
- ProfitEventDetected = TRUE
- reward currency = LOOP
- settlement executes successfully

No other mint paths exist.

### 6.2.3 Burning
LOOP burning:
- NOT required in v1
- if implemented, it MUST be user-initiated burn only
- system MUST NOT burn user LOOP without explicit action

---

## 6.3 LOOP Minting Engine (Hard)

### 6.3.1 Mint Trigger
Mint occurs only during Settlement Engine execution.

### 6.3.2 Mint Price
MintPrice MUST equal:
- FloorPrice (current)

### 6.3.3 Mint Amount Formula
LoopMinted = floor(UserNetProfitUSDT * 1e18 / FloorPrice)

FloorPrice must be expressed in USDT terms scaled to 1e18:
- FloorPriceUSDT_1e18

Rounding:
- LOOP minted rounds DOWN (reserve safety)

### 6.3.4 Mint Output
Minted LOOP is NOT auto-sent to wallet.
It is credited to claim system:

- Position.claimableLOOP += LoopMinted

This prevents accidental immediate redemption loops and creates clean accounting.

---

## 6.4 Claim System (Hard)

Claim balances exist to separate:
- principal + trading funds
from
- user-owned profit rewards

### 6.4.1 ClaimableLOOP Handling
- claimableLOOP belongs to user
- it MUST NOT be used by trading engine
- it MUST be withdrawable/claimable at any time

### 6.4.2 Claim LOOP Flow
User action:
- claimLoop(positionId, amount)

Rules:
- amount <= claimableLOOP
- decrease claimableLOOP
- transfer LOOP token to user wallet

### 6.4.3 Claim USDT Flow
User action:
- claimUSDT(positionId, amount)

Rules:
- amount <= claimableUSDT
- decrease claimableUSDT
- transfer USDT to user wallet

---

## 6.5 LOOP Transferability (Locked)

LOOP MUST be transferable like standard ERC-20.

There are no transfer restrictions in v1, except:
- blacklist/sanction compliance may be added later (not in v1 unless required by law)

---

## 6.6 LOOP Redemption Integration (Hard)

LOOP redemption to USDT is defined in Section 9.

LOOP contract MUST allow Redemption Engine to:
- transferFrom user wallet LOOP into redemption module
- burn or retain redeemed LOOP (implementation choice locked below)

Locked v1 rule:
- redeemed LOOP MUST be burned
(to prevent redemption reuse and to keep supply honest)

Thus redemption process:
- receive LOOP
- burn LOOP
- pay USDT from System Reserve

---

## 6.7 LOOP Events (Mandatory)

LOOP token contract MUST emit:
- Transfer (ERC-20 standard)
- Approval (ERC-20 standard)

YieldLoop system MUST emit:
- LoopMinted(vaultId, positionId, loopAmount, floorPrice)
- LoopClaimed(vaultId, positionId, loopAmount)
- LoopRedeemed(user, loopAmount, usdtOut, bonusMultiplier)

---

## 6.8 Dev Checklist (Fee + LOOP Layer)

Developer MUST deliver:

✅ only one fee type (performance on distributed profit)  
✅ fee rate locked by reward currency (20% vs 17.5%)  
✅ exact fee routing to 4 pools  
✅ SR cannot be withdrawn for spending  
✅ LOOP minted only from verified profits  
✅ LOOP mint price = FloorPrice always  
✅ claim balances fully segregated  
✅ redeemed LOOP is burned  
✅ full event emission for fees + LOOP

---

# 7. System Reserve (SR) + Collateralization Ratio (CR)

This section defines the reserve system that backs LOOP redemption and supports floor/bonus mechanics.
The developer MUST implement SR exactly as written. SR is the backbone of credibility.

---

## 7.1 System Reserve Definition (Hard)

### 7.1.1 Purpose
System Reserve (SR) exists ONLY to:
- pay LOOP redemption payouts (USDT out)
- maintain reserve health
- enable monotonic floor ratchet
- enable conditional redemption bonus tiers

SR is NOT:
- a treasury for discretionary spending
- an investment pool
- a dev fund
- a marketing budget

### 7.1.2 Denomination (Locked v1)
SR MUST be held in:
- USDT only

SR MUST remain liquid at all times.
SR MUST NOT be deployed into any strategies.

---

## 7.2 SR Funding (Hard)

SR is funded ONLY through fee routing:

ReserveInflowUSDT = FeeUSDT * reserveSplitBPS / 10000

Default:
- reserveSplitBPS = 2500 (25%)

SR balance update:
SR_USDT_next = SR_USDT_current + ReserveInflowUSDT

---

## 7.3 SR Outflows (Hard)

The ONLY permitted SR outflows are:
- redemption payouts (RedeemUSDT)
- redemption queue payouts (if queue enabled)

No other SR transfers are allowed.
Admin cannot withdraw SR.
Any SR withdrawal attempt outside redemption MUST revert.

---

## 7.4 Collateralization Ratio (CR) (Hard)

CR is the reserve health metric.

Definitions:
- SR_USDT = system reserve balance in USDT
- LOOP_Supply = circulating LOOP supply (total minted - total burned)
- FloorPrice = current floor price (USDT per LOOP)

CR formula:
CR = SR_USDT / (LOOP_Supply * FloorPrice)

Units:
- numerator in USDT
- denominator in USDT

---

## 7.5 TargetCR + MinCR (Locked)

### 7.5.1 TargetCR (Locked default)
TargetCR controls the floor ratchet aggressiveness.

Locked default:
- TargetCR = 1.05

Meaning:
- floor can only rise if reserve surplus supports 105% collateral buffer.

### 7.5.2 MinCR (Locked default)
MinCR defines redemption safety threshold.

Locked default:
- MinCR = 1.00

Rules:
- If CR would fall below MinCR after redemption:
  - redemption MUST NOT execute normally
  - redemption MUST apply queue logic or bonus collapse (Section 9/10)

---

## 7.6 SR Storage + Accounting Requirements

SR must be stored in a dedicated contract module:
- ReserveVault

ReserveVault MUST store:
- SR_USDT balance
- lifetimeReserveInflowUSDT
- lifetimeReserveOutflowUSDT

ReserveVault MUST expose read methods:
- getReserveBalance()
- getCollateralizationRatio()
- getMinCR()
- getTargetCR()

---

## 7.7 SR Events (Mandatory)

ReserveVault MUST emit:

- ReserveInflow(amountUSDT, sourcePositionId, timestamp)
- ReserveOutflow(amountUSDT, toUser, loopBurned, timestamp)
- ReserveBalanceUpdated(oldBalanceUSDT, newBalanceUSDT)

---

## 7.8 SR Integrity Rules (Hard)

### 7.8.1 SR Must Match Actual Balance
ReserveVault contract MUST reconcile:
- internal accounting SR_USDT
with
- actual USDT token balance

If mismatch is detected:
- system enters global PAUSE
- emits ReserveMismatchDetected event
- redemptions halt until fixed

### 7.8.2 No SR Token Leakage
ReserveVault must disallow:
- approvals to external addresses
- arbitrary transfers
- delegatecalls to unknown modules

---

# 8. Floor Price Ratchet Engine (Monotonic Floor)

This section defines the floor price mechanism that "goes up and locks" over time.
The developer MUST implement monotonic floor behavior with no exceptions.

---

## 8.1 FloorPrice Definition

FloorPrice is:
- mint price for LOOP rewards
- base redemption price for LOOP redemptions

FloorPrice is expressed as:
- USDT per LOOP

FloorPrice MUST be stored as:
- FloorPriceUSDT_1e18 (scaled integer)

Example:
- FloorPrice = 1.250000 USDT
- stored as 1.25 * 1e18

---

## 8.2 Floor Engine Epoch Schedule (Locked v1)

Floor updates occur:
- once per month

Timing rule:
- Floor update happens at the first successful settlement after 00:00 UTC on the 1st of each month.

If no settlement occurs for long periods:
- keeper MUST submit a floor update transaction at month start.

Floor updates MUST be explicit events.

---

## 8.3 Candidate Floor Calculation (Hard)

Definitions:
- SR_USDT = reserve balance
- LOOP_Supply = current LOOP supply
- TargetCR = 1.05

CandidateFloor =
  SR_USDT / (LOOP_Supply * TargetCR)

CandidateFloor MUST be computed using fixed-point arithmetic.

Edge cases:

### 8.3.1 If LOOP_Supply = 0
If LOOP_Supply == 0:
- FloorPrice MUST remain unchanged
- CandidateFloor is undefined
- emit FloorUpdateSkipped(reason=ZERO_SUPPLY)

This prevents division by zero.

---

## 8.4 Monotonic Floor Rule (Hard)

FloorPrice_next = max(FloorPrice_current, CandidateFloor)

FloorPrice MUST NEVER decrease.

Even if:
- reserve drops
- markets crash
- AUM declines
- volume falls

Floor remains constant until reserve supports higher floor.

---

## 8.5 Floor Initialization (Locked)

FloorPrice MUST initialize at:

- InitialFloorPrice = 1.000000 USDT

Stored as:
- 1.0 * 1e18

This ensures early LOOP is minted at 1:1 basis until reserve surplus builds.

---

## 8.6 Floor Update Authority (Locked)

Floor updates may be executed by:
- keeper/executor role
- OR Settlement Engine after Profit Event

Floor updates MUST obey:
- epoch schedule
- monotonic rule

No admin override to lower floor exists.

Admin MAY trigger emergency floor freeze:
- floor stops updating upward temporarily
- floor never decreases
- requires multi-sig approval
- emits FloorFrozen event

Default:
- Floor freeze OFF

---

## 8.7 Floor Update Events (Mandatory)

Floor Engine MUST emit:

- FloorUpdateAttempt(timestamp, currentFloor, candidateFloor)
- FloorUpdated(oldFloor, newFloor, timestamp)
- FloorUpdateSkipped(reasonCode, timestamp)

Reason codes:
- ZERO_SUPPLY
- EPOCH_NOT_REACHED
- RESERVE_MISMATCH
- SYSTEM_PAUSED

---

## 8.8 Floor Integration Rules (Hard)

### 8.8.1 Mint Uses Current Floor
Settlement Engine MUST use the current FloorPrice at time of mint:
- MintPrice = FloorPrice

### 8.8.2 Redemption Uses Current Floor
Redemption Engine MUST use the current FloorPrice at time of redemption:
- RedeemBase = LOOP_amount * FloorPrice

No stale floor allowed.

---

## 8.9 Dev Checklist (Reserve + Floor)

Developer MUST implement:

✅ SR is USDT-only  
✅ SR only funds redemption outflows  
✅ CR computed correctly from SR, LOOP supply, FloorPrice  
✅ TargetCR=1.05, MinCR=1.00  
✅ Floor updates monthly  
✅ Floor init = 1.00  
✅ Floor monotonic max(current, candidate)  
✅ Full event emission + mismatch detection halt

---

# 9. Redemption Engine (Base + Bonus Tiers + Queue)

This section defines LOOP redemption to USDT. Redemption is ALWAYS reserve-gated.
The developer MUST implement redemption exactly as written.

---

## 9.1 Redemption Purpose (Hard)
Redemption provides a deterministic way for users to convert LOOP back into USDT at:
- a guaranteed base rate (FloorPrice)
- plus a conditional reserve-funded bonus tier

Redemption is NOT:
- guaranteed profit
- guaranteed bonus
- guaranteed instant liquidity under stress

---

## 9.2 Redemption Inputs / Outputs (Locked)

### 9.2.1 Input
- LOOP amount (user-specified)

### 9.2.2 Output (Locked v1)
- USDT only

YieldLoop v1 MUST NOT support:
- redemption to other tokens
- redemption to stable baskets
- redemption to BNB

---

## 9.3 Redemption Preconditions (Hard)

Redemption may execute only if:
- system not globally paused OR redemption allowed while paused (default: disallowed)
- reserve mismatch flag is FALSE
- LOOP amount > 0
- user has LOOP balance >= LOOP amount
- redemption does not violate MinCR

If any fail:
- redemption MUST revert OR queue (as defined below)

---

## 9.4 Redemption Base Rate (Hard)

Base redemption always uses:
- FloorPrice (current)

BaseUSDTOut =
  LOOP_amount * FloorPrice

No exceptions.

---

## 9.5 Bonus System (Tiered, Reserve-Gated)

### 9.5.1 Bonus Overview
BonusMultiplier exists to reward usage and locking, but must never endanger reserve.

Bonus multiplier always satisfies:
- 1.0000 <= BonusMultiplier <= CapMultiplier

Unlocked users:
- CapMultiplier = 1.0250

Locked users:
- CapMultiplier = per lock schedule (Section 10)

### 9.5.2 Tier Step (Hard)
BonusMultiplier MUST move in fixed steps:

TierStep = 0.0075

BonusMultiplier permitted values (unlocked):
- 1.0000
- 1.0075
- 1.0150
- 1.0225
- 1.0250

Locked users may have additional caps above 1.0250 but bonus still moves in TierStep increments.

---

## 9.6 CR-Based Bonus Tier Mapping (Hard)

Redemption bonus is determined from CR bands.

Definitions:
- CR = SR_USDT / (LOOP_Supply * FloorPrice)
- CapMultiplier = unlocked cap or lock cap

### 9.6.1 CR Bands (Locked)
The system MUST use these CR bands:

Band 0: CR < 1.0000  
Band 1: 1.0000 <= CR < 1.0125  
Band 2: 1.0125 <= CR < 1.0250  
Band 3: 1.0250 <= CR < 1.0375  
Band 4: CR >= 1.0375  

These bands were chosen to align with 0.0075 tier increments and conservative reserve gating.

### 9.6.2 TierIndex Assignment (Locked)
TierIndex MUST be:

- Band 0 -> TierIndex = 0
- Band 1 -> TierIndex = 1
- Band 2 -> TierIndex = 2
- Band 3 -> TierIndex = 3
- Band 4 -> TierIndex = 4

### 9.6.3 BonusMultiplier Formula (Hard)
BonusMultiplierRaw =
  1.0000 + TierStep * TierIndex

Then apply cap:
BonusMultiplier =
  min(BonusMultiplierRaw, CapMultiplier)

---

## 9.7 Redemption Output Formula (Hard)

USDTOut =
  floor(LOOP_amount * FloorPrice * BonusMultiplier)

All values are fixed-point scaled.

Rounding rule (Locked):
- redemption payout rounds DOWN (reserve safety)

---

## 9.8 MinCR Enforcement (Hard)

Redemption MUST NOT execute if it would cause:
- CR_post < MinCR

MinCR (Locked):
- 1.0000

CR_post is computed conservatively as:

SR_post = SR_USDT - USDTOut

CR_post =
  SR_post / ((LOOP_Supply - LOOP_amount) * FloorPrice)

Edge case:
- if LOOP_Supply - LOOP_amount == 0:
  - CR_post is not needed
  - allow redemption if SR_post >= 0

---

## 9.9 Redemption Queue System (Mandatory Fallback)

YieldLoop MUST implement a redemption queue.

Reason:
- prevents bank-run death spirals
- keeps system deterministic
- ensures fair processing under stress

Queue type:
- FIFO (First In First Out)

### 9.9.1 When to Queue
Redemption request MUST be queued if:
- normal redemption would violate MinCR
OR
- system is paused but queue acceptance allowed (default OFF)

Locked v1 rule:
- queue acceptance allowed ONLY if system status is ACTIVE
- if PAUSED: redemption and queue submission both disabled

### 9.9.2 Queue Entry Schema (Hard)
Each queued request stores:
- queueId
- userAddress
- timestamp
- loopAmountRequested
- capMultiplierAtSubmit (derived from lock state)
- status (QUEUED / PARTIAL / FILLED / CANCELLED)

### 9.9.3 Cancel Queue Request
User may cancel queued redemption ONLY if:
- status == QUEUED
- not partially filled

If cancelled:
- mark CANCELLED
- emit RedemptionQueueCancelled event

---

## 9.10 Queue Processing Rules (Hard)

### 9.10.1 Processing Frequency (Locked)
Queue processing occurs:
- once per day at 00:00 UTC
- executed by keeper

### 9.10.2 Daily Redemption Budget (Locked)
To preserve SR health, daily redemptions are limited:

DailyRedemptionMaxPercentSR = 5.0% (Locked)
DailyRedemptionMaxUSDT =
  SR_USDT * 0.05

Meaning:
- no more than 5% of SR can be redeemed per day

### 9.10.3 Partial Fills
If a queued redemption request exceeds remaining daily budget:
- request is partially filled
- remaining LOOP stays queued

Partial fill MUST:
- burn proportional LOOP
- pay proportional USDTOut
- update remaining loopAmountRequested

---

## 9.11 Redeemed LOOP Handling (Locked)

Redeemed LOOP MUST be burned.

Steps:
1) transferFrom user LOOP into redemption contract
2) burn LOOP
3) transfer USDTOut from SR to user

This prevents:
- double redemption
- circular re-use
- supply inflation exploits

---

## 9.12 Redemption Events (Mandatory)

Contracts MUST emit:

- RedemptionRequested(user, loopAmount, timestamp)
- RedemptionExecuted(user, loopBurned, usdtOut, floorPrice, bonusMultiplier, capMultiplier, timestamp)
- RedemptionQueued(queueId, user, loopAmount, capMultiplier, timestamp)
- RedemptionQueueFilled(queueId, user, loopBurned, usdtOut, remainingLoopQueued, timestamp)
- RedemptionQueueCancelled(queueId, user, timestamp)

---

# 10. VaultOS Lock Engine (30 Days → 25 Years)

Locking exists solely to increase the redemption bonus CAP.
Locking does NOT change FloorPrice.
Locking does NOT lock principal trading access unless explicitly chosen.

---

## 10.1 Locking Type (Locked v1)

Locked v1 model:
- locking applies to the Position
- it increases redemption cap (CapMultiplier)

Lock does not apply separately to:
- claim balances
- wallet-held LOOP

Meaning:
- locked advantage applies when redeeming LOOP associated with that Position’s claim/redemption pathway

Implementation requirement:
- redemption calls MUST include PositionID context if lock is enabled
- system must validate user ownership of PositionID

---

## 10.2 Lock Options Menu (Locked v1)

User may choose:
- OFF (default)
- 30 days
- 90 days
- 180 days
- 1 year
- 2 years
- 3 years
- 5 years
- 10 years
- 25 years

No other durations allowed in v1.

---

## 10.3 Lock Cap Schedule (Locked v1)

CapMultiplier values are locked as:

Unlocked:
- 1.0250

Locked:
- 30d  -> 1.0250
- 90d  -> 1.0275
- 180d -> 1.0300
- 1y   -> 1.0350
- 2y   -> 1.0400
- 3y   -> 1.0450
- 5y   -> 1.0500
- 10y  -> 1.0500
- 25y  -> 1.0500

Hard maximum:
- 1.0500

No caps above 1.05 exist in v1.

---

## 10.4 Lock Activation Rules (Hard)

### 10.4.1 Lock Activation Moment
LockStartTimestamp is set when:
- user confirms lock during Position configuration OR
- user adds lock later (only allowed when no open execution)

LockEndTimestamp:
- LockStartTimestamp + LockDurationSeconds

### 10.4.2 Lock Can Be Added Later (Allowed With Constraints)
User may add a lock later ONLY if:
- openExecutionCount = 0
- closeRequested = FALSE
- PositionState is READY
- user re-acknowledges disclosures

---

## 10.5 Lock Immutability (Hard)

Once lockEnabled = TRUE:
- lock cannot be removed
- lock cannot be shortened
- lock cannot be extended in v1
(no “extend to upgrade bonus” feature in v1)

Early exit:
- NOT allowed

---

## 10.6 Lock + Redemption Interaction (Hard)

CapMultiplier used during redemption MUST be:

If lockEnabled = FALSE:
- CapMultiplier = 1.0250

If lockEnabled = TRUE:
- CapMultiplier = lockBonusCapMultiplier (per schedule)

BonusMultiplier is computed from CR bands (Section 9),
then capped:

BonusMultiplier = min(BonusMultiplierRaw, CapMultiplier)

---

## 10.7 Lock Maturity Behavior (Locked v1)

When lock ends:
- lockEnabled remains TRUE (historical)
- but lock cap advantage expires

Locked rule:
- After maturity, CapMultiplier reverts to unlocked cap:
  - 1.0250

This prevents permanent privilege without ongoing lock.

---

## 10.8 Lock UI Requirements (Hard)

UI MUST disclose:
- lock is irreversible
- early exit not allowed
- lock increases redemption bonus cap only
- bonus is conditional and reserve-gated
- floor never decreases but bonus can drop

UI MUST require:
- explicit checkbox:
  - “I understand this lock is irreversible and cannot be exited early.”

---

## 10.9 Lock Events (Mandatory)

Contracts MUST emit:

- LockEnabled(vaultId, positionId, durationSeconds, capMultiplier, startTimestamp, endTimestamp)
- LockExpired(vaultId, positionId, timestamp)
- LockCapApplied(vaultId, positionId, capMultiplier, timestamp)

---

# 11. Safety Guardrails + Failure Handling + Pauses

This section defines system-wide safety constraints.
The developer MUST implement these guardrails as enforceable logic (contract checks + keeper constraints).
No silent failure is allowed. All failure paths must emit reason-coded events.

---

## 11.1 Guardrail Philosophy (Hard)
YieldLoop is NOT a casino bot.
YieldLoop MUST prioritize:
- capital preservation
- deterministic accounting
- gas efficiency
- clean failure handling

If the system cannot execute safely, it MUST do nothing.

---

## 11.2 Trading Guardrails (Locked Defaults)

These values are locked defaults for v1 and MUST exist as on-chain configurable parameters,
but MUST initialize to the values below.

### 11.2.1 Slippage Guardrails
- MaxSlippagePerLegBPS = 30 (0.30%)

Rule:
- If expected slippage > MaxSlippagePerLegBPS -> abort execution

### 11.2.2 Gas Guardrails
- MaxGasPerExecutionUSDT = 1.25 USDT

Rule:
- If estimated gas > MaxGasPerExecutionUSDT -> abort execution

### 11.2.3 Liquidity Depth Guardrails
- MinimumLiquidityDepthUSDT = 50,000 USDT per pool

Rule:
- if insufficient liquidity -> reduce trade size
- if size cannot be reduced enough -> abort execution

### 11.2.4 Execution Size Guardrails
Per Position:
- MaxExecutionSizePercent = 10%
- MaxExecutionSizeUSDT = 750

Arbitrage-specific:
- MaxArbSizePercent = 7.5%
- MaxArbSizeUSDT = 500

Rule:
- never exceed either cap

### 11.2.5 Quote Freshness Guardrails
- QuoteTTLSeconds = 15 seconds

Rule:
- if quote age > 15 seconds -> abort execution

---

## 11.3 Profit Threshold Guardrails (Locked Defaults)

### 11.3.1 Spot Profit Threshold
- MinExpectedProfitSpotUSDT = 0.50

### 11.3.2 Arbitrage Profit Threshold
- MinExpectedProfitArbUSDT = 1.50

Rule:
- if ExpectedNetProfitUSDT < threshold -> abort execution

---

## 11.4 Risk Drift Guardrails (Allocation Discipline)

### 11.4.1 Drift Definition
Drift is deviation between:
- actual held allocation
and
- user target allocation

### 11.4.2 Drift Threshold
- MaxDriftBPS = 250 (2.5%) per asset

Rule:
- if drift exceeds threshold:
  - spot rebalancing becomes priority
  - speculative spot trades are blocked until drift returns within limits

---

## 11.5 Failure Handling (Hard)

### 11.5.1 No Partial Trades Policy
If a multi-leg plan cannot complete safely:
- engine MUST abort before starting
- if already started -> controlled unwind

### 11.5.2 Controlled Unwind Policy
If the system is left holding TOKEN exposure (unplanned):
- system MUST schedule unwind to USDT ASAP
- unwind must obey:
  - slippage cap
  - gas cap
  - liquidity depth
- if unwind cannot obey caps:
  - hold token and pause Position
  - notify user

---

## 11.6 Pause System (Position-Level + Global)

### 11.6.1 Position Pause Conditions (Locked)
Position MUST auto-pause if any condition occurs:

A) MaxFailedExecutionsPer10Min exceeded
- MaxFailedExecutionsPer10Min = 3

B) Quote failure persists
- QuoteFailureWindow = 5 minutes

C) Gas spike persists
- GasSpikeWindow = 10 minutes

D) Reserve mismatch detected
- immediate pause (global)

E) Execution accounting mismatch
- e.g. snapshot invalid / cost data missing

F) Arbitrage exposure cannot unwind within constraints

### 11.6.2 Position Pause Effects
When PAUSED:
- no new executions allowed
- user may:
  - withdraw freeUSDT
  - request close
  - claim balances
  - redeem (only if not globally paused)

### 11.6.3 Position Unpause Rule
Position may unpause ONLY if:
- pause condition cleared
- openExecutionCount = 0
- closeRequested = FALSE
- keeper submits unpause tx

---

## 11.7 Global Pause / Kill Switch (Hard)

### 11.7.1 Global Pause Conditions
Global pause MUST trigger if:
- Reserve mismatch detected
- Critical exploit detected (manual)
- Oracle sanity check failure persists (if oracle sanity module active)

### 11.7.2 Global Pause Effects
When globally paused:
- no new trades or arbitrage
- no config changes
- deposits disabled
- withdrawals of freeUSDT allowed
- claim balances allowed
- redemption disabled (Locked v1 default)

---

## 11.8 Oracle Sanity Check (Locked v1 Setting)

Oracle sanity module is OPTIONAL but default ON for safety.

### 11.8.1 Oracle Purpose
Oracle sanity is not used for trading decisions.
It is used ONLY to block obviously corrupted pool prices.

### 11.8.2 Oracle Inputs
Oracle must provide:
- reference price for BTCB/USDT, ETH/USDT, BNB/USDT, XRP/USDT, SOL/USDT

### 11.8.3 Sanity Threshold
If DEX implied price deviates from oracle by:
- OracleDeviationMaxBPS = 500 (5.0%)

Then:
- block execution
- increment failure counter

---

## 11.9 Events (Mandatory)

Contracts MUST emit:

- GuardrailTriggered(vaultId, positionId, guardrailType, value, threshold, timestamp)
- PositionPaused(vaultId, positionId, reasonCode, timestamp)
- PositionUnpaused(vaultId, positionId, timestamp)
- GlobalPaused(reasonCode, timestamp)
- GlobalUnpaused(timestamp)

guardrailType enum MUST include:
- SLIPPAGE
- GAS
- LIQUIDITY
- QUOTE_TTL
- PROFIT_THRESHOLD
- ORACLE_SANITY
- SIZE_CAP
- DRIFT

reasonCode enum MUST include:
- FAILED_EXECUTIONS
- QUOTE_FAILURE
- GAS_SPIKE
- RESERVE_MISMATCH
- SNAPSHOT_INVALID
- EXPOSURE_UNWIND_FAILURE
- ORACLE_DEVIATION
- ADMIN_EMERGENCY

---

# 12. Engine List (Per-Engine Inputs / Outputs / Triggers / Fallbacks)

This section is the full engine inventory.
Each engine MUST be implemented as a contract module, service module, or explicit component.
No engine may be merged away if it removes auditability.

---

## Engine 1 — Vault Manager (Contract)
### Purpose
- create Vaults and Positions
- store configs and state
- manage withdrawals, closes, claims

### Inputs
- depositUSDT()
- configurePosition()
- updateConfig()
- requestClose()
- withdrawFreeUSDT()
- claimUSDT()
- claimLOOP()

### Outputs
- state updates
- events for every action

### Fallback
- revert invalid actions

---

## Engine 2 — Allocation Router (Off-chain + On-chain validation)
### Purpose
- compute target allocations
- propose trades that move holdings toward target weights

### Inputs
- Position free balances
- allocationBPS target
- drift metrics

### Outputs
- trade plan (spot)

### Guardrails
- drift <= MaxDriftBPS
- tokens limited to approved list

### Fallback
- hold USDT if plan unsafe

---

## Engine 3 — Spot Execution Engine (Keeper service + contract executor)
### Purpose
- execute spot trades on PCS/BiSwap conservatively

### Trigger
- ExpectedNetProfitSpotUSDT >= 0.50
- guardrails satisfied

### Outputs
- Execution logs
- Snapshot data

### Fallback
- abort execution cleanly

---

## Engine 4 — Arbitrage Engine (Keeper service + contract executor)
### Purpose
- execute PCS↔BiSwap arbitrage when spreads exist

### Trigger
- ExpectedNetProfitArbUSDT >= 1.50
- guardrails satisfied

### Outputs
- two-leg arb execution
- logs

### Fallback
- controlled unwind / pause

---

## Engine 5 — Quote Engine (Keeper service)
### Purpose
- pull PCS and BiSwap router quotes
- calculate slippage impact and quote TTL

### Inputs
- pools/routes
- current gas price

### Outputs
- expectedOut values
- quote timestamp

### Fallback
- if quote fails -> increment failure counter

---

## Engine 6 — Profit Detector / Auditor (Contract module)
### Purpose
- compute StartValueUSDT / EndValueUSDT
- compute NetProfitUSDT deterministically

### Outputs
- ProfitEventDetected OR SettlementSkipped

### Fallback
- classify as NON_PROFIT_EVENT if uncertain

---

## Engine 7 — Settlement Engine (Contract module)
### Purpose
- profit split (compound/collect)
- fee charge
- fee routing
- claim credit (USDT or LOOP)
- lifetime counters

### Guardrails
- atomic settlement only
- no partial writes

### Fallback
- revert and pause after repeated failure

---

## Engine 8 — Fee Router + Pool Vaults (Contract module)
### Purpose
- route FeeUSDT to:
  - DevOps
  - Marketing
  - Reserve
  - LoopLabs

### Fallback
- revert if fee split invalid

---

## Engine 9 — LOOP Mint Engine (Contract module)
### Purpose
- mint LOOP only from profit
- mint at FloorPrice only

### Fallback
- no mint if profit invalid

---

## Engine 10 — Reserve Vault (Contract module)
### Purpose
- hold SR_USDT
- enforce reserve-only-outflow via redemption
- provide CR

### Fallback
- global pause on mismatch

---

## Engine 11 — Floor Ratchet Engine (Contract module)
### Purpose
- monthly FloorPrice update
- monotonic max(current, candidate)

### Fallback
- skip update with reason codes

---

## Engine 12 — Redemption Engine (Contract module)
### Purpose
- redeem LOOP to USDT
- compute bonus tiers
- enforce MinCR
- burn redeemed LOOP
- queue requests when needed

### Fallback
- queue rather than fail under CR constraints

---

## Engine 13 — Redemption Queue Processor (Keeper service + contract function)
### Purpose
- process queued redemptions daily
- enforce daily redemption budget (5% SR/day)

### Fallback
- pause queue processing if reserve mismatch

---

## Engine 14 — Lock Engine (VaultOS) (Contract module)
### Purpose
- apply lock cap schedule
- enforce immutability
- expire cap at maturity

### Fallback
- revert invalid lock changes

---

## Engine 15 — Safety Monitor (Off-chain)
### Purpose
- monitor:
  - failed executions
  - pause triggers
  - reserve balance health
  - gas spikes
  - oracle sanity failures

### Outputs
- alerts to admin
- keeper stop signals

---

## Engine 16 — Notification Engine (Off-chain)
### Purpose
- monthly emails with:
  - vault summary
  - lock status
  - claim balances
  - floor/bonus tier
  - safety notices

### Fallback
- retries + log failures

---

## Engine 17 — Admin / Governance Engine (Multi-sig + Timelock)
### Purpose
- control parameter changes:
  - fee split
  - guardrails
  - TargetCR
- control global pause/unpause
- cannot change immutable constraints

### Hard limits
- cannot lower FloorPrice
- cannot mint LOOP outside profit events
- cannot withdraw reserve

---

## 12.1 Engine Implementation Checklist (Hard)
Developer MUST deliver all engines above with:
- documented interfaces
- event schemas
- integration tests

No engine may be omitted without explicit patch approval.

---

# 13. Contract Architecture (Contracts, Storage, Roles, Upgrade Policy)

This section defines exactly what contracts must exist, what they store, and who can do what.
The developer MUST follow this architecture so the system is maintainable, auditable, and not dependent on “tribal knowledge”.

---

## 13.1 Required Contracts (Locked v1)

YieldLoop v1 MUST deploy the following contracts:

1) **YieldLoopVault** (main vault / positions contract)
2) **ExecutionCoordinator** (keeper entrypoint + execution gating)
3) **SettlementEngine** (profit settlement + fee charge + reward credit)
4) **FeeRouter** (fee split routing + pool accounting)
5) **PoolVault_DevOps** (USDT pool)
6) **PoolVault_Marketing** (USDT pool)
7) **ReserveVault (SR)** (USDT pool, redemption-only outflow)
8) **PoolVault_LoopLabs** (USDT pool)
9) **LoopToken (LOOP)** (ERC20/BEP20 mintable/burnable token)
10) **FloorEngine** (monotonic ratchet logic)
11) **RedemptionEngine** (redeem LOOP→USDT)
12) **RedemptionQueue** (FIFO queue state)
13) **LockEngine** (VaultOS lock module)
14) **TimelockController** (governance/admin change delay)
15) **RoleManager / AccessControl** (RBAC; can be merged into each contract using OpenZeppelin)

Contracts may be merged ONLY if:
- no loss of separation of concerns
- no loss of event visibility
- no loss of upgrade safety
- no increased blast radius

Default recommendation:
- keep modular

---

## 13.2 Core Storage Model (Hard)

### 13.2.1 YieldLoopVault Storage
YieldLoopVault MUST store:

**Vault Storage**
- mapping(address => Vault)
- mapping(bytes32 VaultID => Vault)

**Position Storage**
- mapping(VaultID => mapping(PositionID => Position))

**Accounting**
- totalDepositedUSDT
- totalClaimableUSDT
- totalClaimableLOOP
- totalExecutions

**State Flags**
- globalPaused (bool)
- reserveMismatchFlag (bool)

### 13.2.2 Position Storage Schema (Hard)
Position struct MUST contain exactly these groups:

A) Identity
- vaultId
- positionId
- createdTimestamp

B) Config
- allocationBPS[5]
- compoundingMode
- rewardCurrency

C) Lock
- lockEnabled
- lockDurationSeconds
- lockStart
- lockEnd
- lockCapMultiplier

D) Execution
- positionState
- openExecutionCount
- inTradeUSDT
- freeUSDT
- freeTokenBalances[5]
- lastExecutionTimestamp
- failureCounterRolling10Min
- lastFailureTimestamp

E) Claim
- claimableUSDT
- claimableLOOP
- lifetimeNetProfitUSDT
- lifetimeFeesUSDT

F) Close
- closeRequested
- closeRequestTimestamp

G) Disclosures
- strategyDisclosureAckTimestamp
- feeDisclosureAckTimestamp
- riskDisclosureAckTimestamp
- lastConfigChangeTimestamp

---

## 13.3 Role-Based Access Control (RBAC) (Hard)

### 13.3.1 Roles (Locked)
System MUST define these roles:

- DEFAULT_ADMIN_ROLE (multi-sig only)
- TIMELOCK_ADMIN_ROLE (timelock management)
- EXECUTOR_ROLE (keeper/bot execution)
- PAUSER_ROLE (emergency pause authority)
- UNPAUSER_ROLE (unpause authority via timelock unless emergency cleared)
- FEE_SPLIT_SETTER_ROLE (timelocked)
- PARAMETER_SETTER_ROLE (timelocked)
- DEVOPS_WITHDRAW_ROLE
- MARKETING_WITHDRAW_ROLE
- LOOPLABS_WITHDRAW_ROLE
- FLOOR_UPDATER_ROLE (keeper + settlement engine)
- QUEUE_PROCESSOR_ROLE (keeper)

No role may be held by an EOA except:
- keeper EOAs MAY hold EXECUTOR_ROLE, FLOOR_UPDATER_ROLE, QUEUE_PROCESSOR_ROLE
All admin roles MUST be multi-sig contracts.

### 13.3.2 Role Safety Requirements
- role grants MUST be timelocked (except EXECUTOR_ROLE)
- role revocation MUST be immediate (admin action)
- all role changes MUST emit events

---

## 13.4 Upgrade Policy (Locked v1)

### 13.4.1 Upgradeability Choice (Locked)
YieldLoop v1 MUST use one of two models:

**Model A (Preferred): Non-upgradeable immutable contracts**
- redeploy for upgrades
- safest
- simplest for audit

**Model B: Proxy upgrade with strict timelock**
- UUPS proxy
- timelock enforced upgrades only

Locked decision for v1:
✅ **Model A — Non-upgradeable immutable contracts**

Reason:
- avoids proxy complexity and attack surface
- improves audit success
- simplifies credibility narrative

### 13.4.2 Parameter Changes Allowed
Even though contracts are immutable, parameters MAY be changeable via timelock:

- fee split BPS
- slippage caps
- gas caps
- thresholds
- TargetCR
- DailyRedemptionMaxPercentSR

These parameters MUST be stored on-chain and changed only by timelock.

---

## 13.5 Timelock Requirements (Hard)

### 13.5.1 Timelock Delay (Locked)
- TimelockDelaySeconds = 172800 seconds (48 hours)

All parameter changes MUST wait 48 hours.

### 13.5.2 Emergency Exceptions
Emergency pause MAY be immediate without timelock.

Emergency unpause:
- requires multi-sig
- and if pause lasted >24h, unpause action MUST be accompanied by public reason note in event logs

---

## 13.6 Contract Interaction Flow (Hard)

### Execution flow:
ExecutionCoordinator:
- validates position state READY
- validates guardrails inputs from keeper
- calls PCS/BiSwap routers
- emits execution leg events
- calls ProfitDetector
- calls SettlementEngine
- SettlementEngine calls:
  - FeeRouter
  - LoopToken (mint if applicable)
  - ReserveVault (inflow accounting)
  - FloorEngine (if epoch)

Redemption flow:
RedemptionEngine:
- pulls FloorPrice
- computes CR + tier bonus
- applies lock cap
- checks MinCR
- if safe:
  - transfers LOOP from user
  - burns LOOP
  - pays USDT from ReserveVault
- else:
  - adds RedemptionQueue entry

Queue processing:
QueueProcessor:
- reads SR budget
- executes FIFO fills
- burns LOOP
- pays USDT

---

## 13.7 Required Interfaces (Hard)

All cross-contract calls MUST use defined interfaces:

- IYieldLoopVault
- IExecutionCoordinator
- ISettlementEngine
- IFeeRouter
- IReserveVault
- ILoopToken
- IFloorEngine
- IRedemptionEngine
- ILockEngine
- IRedemptionQueue

These interfaces MUST be published in repo for audit.

---

## 13.8 Mandatory Contract Events (Architecture Layer)

Contracts MUST emit:

- RoleGranted(role, account, sender)
- RoleRevoked(role, account, sender)
- TimelockScheduled(actionId, target, data, executeAfter)
- TimelockExecuted(actionId, target)
- ParameterUpdated(paramKey, oldValue, newValue, effectiveTimestamp)

---

# 14. Fiat Onboarding + Offboarding Engine (Transak / MoonPay) — “Normie Rail” (LOCKED)

This section defines the complete fiat rail subsystem for YieldLoop.
It MUST be written into the builder document so the dev can implement it without asking the founder.
Even if fiat rails are activated post-launch, the architecture MUST exist from day 1.

This section is split into:
- 14A Fiat Onboarding (Buy USDT with card/bank)
- 14B Fiat Offboarding (Sell USDT to bank)
- 14C Compliance Boundaries + Risk Disclosures
- 14D Failure Handling + Support Paths
- 14E Data Model + Events + UI Requirements
- 14F Implementation Rules (No Custody, No Ambiguity)

---

## 14A. Fiat Onboarding Engine — Buy USDT (LOCKED)

### 14A.1 Purpose
Allow users to fund their YieldLoop positions using fiat payment methods in a way that feels like:
- Coinbase / CashApp / Robinhood style onboarding
- “Buy USDT → Deposit into YieldLoop”

Fiat onboarding MUST:
- be provider-handled KYC/AML
- avoid YieldLoop custody of fiat
- avoid YieldLoop custody of user identity documents
- route final USDT into the user’s self-custody wallet

---

### 14A.2 Supported Providers (Locked v1+)
Providers MUST be implemented in this order:

**Provider #1 (Primary): Transak**  
**Provider #2 (Secondary fallback): MoonPay**

If one provider fails/blocks the user by region/compliance:
- UI MUST automatically offer the other provider

No additional providers may be added without:
- new security review
- updated disclosures
- version increment to builder doc

---

### 14A.3 Supported Fiat Payment Methods
Support depends on provider availability by region.
UI MUST present payment options dynamically based on provider response.

Supported categories:
- Debit card
- Credit card
- Apple Pay / Google Pay (if available)
- Bank transfer / ACH (if available)

YieldLoop MUST NOT store payment method info.

---

### 14A.4 Supported Purchase Asset (Locked)
Users MUST be able to purchase ONLY:

- **USDT on BNB Chain (BEP-20)**

No other asset purchases allowed through YieldLoop UI in v1+.

Hard rule:
- If provider returns a “buy USDT on Ethereum / Polygon / Tron” option, UI MUST NOT allow it.

---

### 14A.5 Onboarding Flow (Normie Flow) — Exact Steps
The flow MUST be deterministic:

1) User clicks **“Add Funds”**
2) UI offers:
   - **“Buy USDT with card/bank”**
   - OR “I already have USDT”
3) If user selects buy:
   - show providers:
     - Transak (default)
     - MoonPay (fallback)
4) User selects amount:
   - in USD (fiat)
   - UI shows estimated USDT received
5) User completes provider checkout in embedded widget or redirect:
   - provider performs KYC/AML if required
   - provider charges payment method
6) Provider sends **USDT (BEP-20)** to:
   - the user’s connected wallet address
7) YieldLoop UI detects deposit arrival:
   - show “Funds received”
8) UI offers immediate continuation:
   - “Deposit into YieldLoop”
9) User proceeds to YieldLoop deposit:
   - Position created
   - Configure Position (Section 2)
   - Disclosures (Section 15)

---

### 14A.6 Wallet Address Handling (Hard)
Provider payout address MUST always be:

- the currently connected wallet address

YieldLoop MUST:
- display the destination address clearly
- require user confirmation:
  - “Yes, send to this wallet address”

If no wallet connected:
- user cannot start provider checkout

---

### 14A.7 Amount Handling + Minimums
Rules:

- Fiat buy minimum MUST be:
  - $260 USD equivalent
Reason:
- ensures user receives enough USDT to satisfy the YieldLoop minimum deposit after provider fees/spread

UI MUST disclose:
- provider fees and spread are separate from YieldLoop fees

---

### 14A.8 Post-Purchase Detection (Hard)
YieldLoop MUST detect fiat purchase completion by:

Primary detection:
- on-chain event monitoring for USDT transfer to user wallet

Fallback detection:
- provider webhook/order-status polling

YieldLoop MUST display a live state:
- Pending
- Processing
- Delivered (USDT arrived)
- Failed (rejected/refunded)

---

### 14A.9 Onboarding Engine Outputs
This engine must produce:

- fiatOrderId
- providerName
- createdTimestamp
- status
- expectedUSDT
- receivedUSDT
- txHash (once delivered)

---

### 14A.10 Onboarding Engine Failure Paths (Mandatory)
Fiat Onboarding MUST support these failure categories:

A) KYC rejected  
B) Payment rejected  
C) Provider unavailable (API down)  
D) Purchase completed but wrong chain sent  
E) Purchase completed but wrong token sent  
F) Purchase pending > 24 hours  
G) User abandons checkout  
H) Funds delivered but UI doesn’t detect (indexing issue)

For each failure category:
- UI MUST show a human-friendly message
- UI MUST show next action:
  - “Try MoonPay”
  - “Contact support”
  - “Open provider order page”
- system MUST log failure event

---

## 14B. Fiat Offboarding Engine — Sell USDT to Bank (LOCKED)

### 14B.1 Purpose
Let users withdraw profits to their bank in a familiar way:

- “Withdraw → Sell USDT → Receive USD in bank”

This MUST feel like a traditional fintech cash-out.

---

### 14B.2 Supported Providers (Locked)
Same providers as onboarding:

- Transak
- MoonPay

Offboarding MUST:
- default to the provider last used successfully (per wallet)

---

### 14B.3 Supported Sell Asset (Locked)
Offboarding supports ONLY:
- USDT (BEP-20)

Users must first:
- withdraw USDT from YieldLoop contracts to wallet
THEN sell via provider.

YieldLoop MUST NOT sell from inside vault.

---

### 14B.4 Offboarding Flow — Exact Steps
1) User clicks **Withdraw**
2) UI shows:
   - withdraw free USDT
   - request close (if funds are in trade)
3) Once USDT is in wallet:
   - UI shows **“Cash Out to Bank”**
4) User selects provider
5) Provider KYC if needed
6) User completes sell order
7) Provider sends fiat to:
   - bank account / card as selected

---

### 14B.5 Offboarding Detection
YieldLoop may not be able to detect bank settlement reliably.
Therefore:

YieldLoop MUST:
- track provider order IDs
- show status via provider API polling/webhook:
  - initiated
  - pending
  - paid-out
  - failed

---

### 14B.6 Offboarding Failure Paths (Mandatory)
Must support:

A) Sell rejected (compliance)  
B) Bank payout failed  
C) Provider API down  
D) User sends wrong token/chain to provider deposit address  
E) Payout pending > 7 days  

UI MUST display:
- provider contact link
- YieldLoop support email
- what YieldLoop can and cannot control

---

## 14C. Compliance Boundaries + Custody Model (LOCKED)

### 14C.1 No Fiat Custody (Hard)
YieldLoop MUST NEVER:
- accept fiat
- hold fiat
- route fiat
- store banking credentials
- store payment details

YieldLoop integrates providers ONLY as:
- third-party checkout rails

---

### 14C.2 No Identity Storage (Hard)
YieldLoop MUST NOT store:
- passports
- driver licenses
- SSN
- selfies
- KYC outcome docs

If KYC status must be displayed:
- store only a boolean + timestamp:
  - providerKYCVerified = TRUE/FALSE
  - providerName
  - verifiedAt

---

### 14C.3 Disclosure Rules (Hard)
Fiat rail screens MUST include a disclosure:

- “Fiat purchase/sale is provided by Transak/MoonPay.”
- “YieldLoop does not custody fiat.”
- “KYC/AML may be required.”
- “Provider may reject transactions.”
- “Provider fees/spread apply.”

User MUST click:
- “I understand” checkbox before checkout.

---

## 14D. Failure Handling + Support Paths (LOCKED)

### 14D.1 Support Responsibility Boundary
YieldLoop support covers:
- wallet connection issues
- chain selection issues
- deposit workflow
- vault operations
- contract behavior
- transaction status

Provider support covers:
- KYC rejection
- payment rejection
- bank payout failure
- refunds

UI MUST clearly label:
- “Provider issue” vs “YieldLoop issue”

---

### 14D.2 Wrong Chain Protection (Hard Requirement)
YieldLoop MUST implement chain guardrails:

- If connected chain != BNB:
  - disable fiat checkout button
  - show “Switch to BNB Chain”

During provider checkout:
- chain must be locked to BNB only
- destination token must be locked to USDT only

---

## 14E. Data Model + Events (Required)

### 14E.1 FiatOrder Object (Off-chain storage required)
Each fiat event creates a FiatOrder record:

Fields:
- userAddress
- providerName (TRANSAK/MOONPAY)
- direction (BUY/SELL)
- fiatAmount
- fiatCurrency
- usdtAmountExpected
- usdtAmountDelivered
- chainId
- tokenAddressUSDT
- status (INITIATED/PENDING/DELIVERED/FAILED/REFUNDED)
- providerOrderId
- txHash (if buy delivered)
- createdTimestamp
- updatedTimestamp

---

### 14E.2 Required Events (App Layer)
App MUST log events (off-chain analytics/log DB):
- FiatBuyStarted
- FiatBuyCompleted
- FiatBuyFailed
- FiatSellStarted
- FiatSellCompleted
- FiatSellFailed

---

## 14F. Implementation Rules (No Ambiguity)

### 14F.1 Integration Mode
Locked integration mode:
- embedded provider widget where possible
- otherwise provider redirect with return URL

### 14F.2 Security Rules
- never store provider API keys in frontend
- keys in backend only
- webhook signature verification required
- rate limit provider status polling

### 14F.3 Activation Switch
Fiat rails MUST have a single on/off feature flag:
- FiatRailsEnabled

Default:
- OFF at launch unless explicitly enabled

When OFF:
- UI shows “Coming Soon”
- no dead buttons

---

# 15. Normie UX Mode (Familiar Experience Mode) (LOCKED)

This section defines how YieldLoop must present itself to normal users.
It is mandatory. If not built, YieldLoop will remain a DeFi toy for nerds.

This section defines:
- 15A UX modes (Normie vs Advanced)
- 15B Default path (simple)
- 15C Required language replacements
- 15D Required UI structure
- 15E User education + guardrails
- 15F Monthly reporting behavior

---

## 15A. UX Modes (Hard)

YieldLoop MUST provide two distinct UI modes:

1) **Normie Mode (default)**
2) **Advanced Mode (toggle)**

Rules:
- Normie Mode is default for every new wallet
- Advanced mode must be manually enabled
- mode setting stored per wallet address (off-chain + optional on-chain mapping)

---

## 15B. Normie Mode — Default User Flow (Hard)

Normie flow must be:

1) Connect Wallet
2) “Add Funds”
   - buy USDT (Transak/MoonPay) OR deposit existing USDT
3) “Start YieldLoop”
   - shows a simple configuration
4) Confirm
5) Strategy runs
6) Dashboard shows earnings
7) Withdraw / cash-out

---

## 15C. Normie Mode — Mandatory Simplifications

### 15C.1 Hide DeFi Jargon
In Normie mode, UI MUST NOT show:
- “vault”
- “router”
- “DEX”
- “slippage”
- “LP”
- “MEV”
- “BPS”
- “arb”

Those concepts may exist internally but are hidden.

---

### 15C.2 Required Language Replacements
Normie mode wording MUST map as:

- Vault → **Account**
- Position → **Plan**
- Allocation Weights → **Portfolio Mix**
- Compounding Mode → **Reinvest Settings**
- Claim → **Withdraw Profits**
- Redeem LOOP → **Convert LOOP to USDT**
- FloorPrice → **Backed Value**
- BonusMultiplier → **Redemption Boost**
- Close Position → **Stop Plan**
- In trade → **Active**
- Free balance → **Available Now**

---

## 15D. Normie Mode — Configuration Options (Locked)

Normie mode must only expose:

1) Portfolio Mix:
   - Conservative (default)
   - Balanced
   - Aggressive

These map to fixed allocation presets:

**Conservative**
- BTCB 35%
- ETH 25%
- BNB 25%
- XRP 10%
- SOL 5%

**Balanced**
- BTCB 30%
- ETH 25%
- BNB 25%
- XRP 10%
- SOL 10%
(this equals default)

**Aggressive**
- BTCB 20%
- ETH 25%
- BNB 25%
- XRP 15%
- SOL 15%

2) Reinvest Settings:
- Reinvest all (100% compound)
- Half and half (50/50 default)
- Withdraw all profits (100% collect)

3) Rewards:
- USDT (20% fee)
- LOOP (17.5% fee) default

4) Lock Boost (optional):
- OFF default
- 1 year (cap 1.035)
- 3 year (cap 1.045)
- 5 year (cap 1.050)
Normie mode must not show full 30d–25y menu.

---

## 15E. Advanced Mode (Full Control)

Advanced mode MUST expose:
- exact allocation BPS sliders
- exact lock durations menu (30d to 25y)
- full analytics (CR, tier bands)
- execution logs
- slippage/gas guardrail readouts
- redemption queue details

But:
- even advanced mode must still enforce guardrails
- advanced mode does NOT unlock additional strategies in v1

---

## 15F. Normie Mode — Default Screens

Normie mode screens must be:

1) Home / Account
2) Add Funds
3) Start Plan
4) Earnings
5) Withdraw / Cash Out
6) Settings (disclosures, support, advanced toggle)

No additional complexity.

---

## 15G. Normie Mode — Dashboard Requirements

Dashboard MUST show:

- Account Balance (USDT)
- Earnings (USDT)
- Earnings (LOOP) if selected
- “Available now” vs “Active”
- Backed Value (FloorPrice)
- Redemption Boost (effective multiplier)

Also show:
- “Stop Plan” button
- “Withdraw Profits” button

---

## 15H. User Education (Hard)

YieldLoop MUST implement:
- contextual tooltips
- “Learn more” modals
- warning banners during stress conditions (queue, pause)

Mandatory Education cards:
- What is LOOP?
- Why is LOOP redeemable?
- What does Backed Value mean?
- What is Redemption Boost?
- Why can payouts queue?

---

## 15I. Mandatory Safety UX (Hard)

Normie users MUST be shown these warnings:

- before first deposit:
  - “This is crypto. You can lose money.”
- before enabling lock:
  - “Lock cannot be reversed.”
- before redemption:
  - “Bonus can drop if reserve is stressed.”
- during queue:
  - “Your redemption will process daily in order.”

---

## 15J. Monthly Reporting UX (Hard)

Normie dashboard MUST include:
- monthly statement view

Statement must show:
- start balance
- end balance
- net profit
- total fees
- total LOOP minted
- redemptions
- lock status

---

## 15K. Required Product Behavior (Non-Negotiable)
In Normie mode:
- default settings must work immediately
- user must not need to understand DeFi
- deposit/buy/withdraw must feel like fintech
- advanced mode exists but is not pushed

---

# 17. Test Plan + Simulation Plan (Must-Pass Before Launch)

This section defines the complete, mandatory test requirements.
The dev MUST not ship without passing every item in this section.
No “we’ll test later”. No partial.

---

## 17.1 Testing Philosophy (Hard)
YieldLoop is money software.
Testing MUST prove:

- no loss of funds from logic errors
- no incorrect minting of LOOP
- no incorrect settlement math
- floor never decreases
- reserve can never be drained outside redemption
- guardrails prevent unsafe trades
- queue protects against reserve stress

---

## 17.2 Required Test Environments

### 17.2.1 Local Dev
- Hardhat/Foundry local chain
- forked BSC mainnet testing mandatory (see 17.2.3)

### 17.2.2 Testnet Deployment
- BSC testnet deployed contracts
- full keeper running against testnet

### 17.2.3 Fork Testing (Mandatory)
- BSC mainnet fork
- PCS + BiSwap router calls tested on fork
- realistic pool liquidity and slippage

---

## 17.3 Unit Tests (Contracts) — Mandatory Coverage

### 17.3.1 Vault + Position
Must test:
- create vault
- deposit minimum enforced (>=250)
- deposit wrong token reverts
- multi-position per vault correct accounting
- withdrawals only freeUSDT
- closeRequested blocks new executions
- state transitions correct
- config changes blocked during execution
- disclosures required before READY

### 17.3.2 Strategy Guardrails
Must test:
- slippage cap abort
- gas cap abort
- quote TTL abort
- liquidity depth scaling/abort
- execution size caps enforced
- drift caps enforced

### 17.3.3 Settlement Engine
Must test:
- profit event computes correctly
- loss event does not mint / no fee
- compounding modes:
  - Mode A: DP=0 fee=0 compound=100%
  - Mode B: DP=50 fee charged only on DP
  - Mode C: DP=100 fee charged on all DP
- reward currency:
  - USDT credits claimableUSDT
  - LOOP credits claimableLOOP
- rounding rules obeyed

### 17.3.4 Fee Router
Must test:
- fee split sums to 10000 or revert
- routing sums conserve full FeeUSDT
- rounding remainder allocated correctly
- pool withdrawals only by correct roles
- SR cannot be withdrawn except redemption

### 17.3.5 LOOP Token Minting
Must test:
- mint only on ProfitEvent settlement
- mint price = FloorPrice always
- mint rounds down
- no admin mint function exists

### 17.3.6 Reserve + CR
Must test:
- CR formula correct
- reserve mismatch triggers global pause
- SR inflow updates properly
- SR outflow only via redemption

### 17.3.7 Floor Engine
Must test:
- initial floor = 1.0
- candidate calculation correct
- floor monotonic:
  - never decreases even if SR drops
- floor update only monthly epoch

### 17.3.8 Redemption Engine
Must test:
- base redemption uses floor
- bonus tier mapping by CR bands correct
- cap applies (unlocked 1.025, locks higher)
- MinCR enforcement blocks unsafe redemption
- burns redeemed LOOP
- outputs USDT only
- queues unsafe redemption

### 17.3.9 Redemption Queue
Must test:
- FIFO ordering
- daily budget 5% SR enforced
- partial fill works
- cancellation rules obeyed
- no queue processing when reserve mismatch

---

## 17.4 Integration Tests — Mandatory

### 17.4.1 Spot Execution Integration
Simulate:
- keeper proposes spot trade
- contract validates guardrails
- router swap executes
- settlement runs
- claim balances updated

Must pass for:
- PCS only
- BiSwap only
- venue selection switching

### 17.4.2 Arbitrage Integration
Simulate:
- buy PCS sell BiSwap
- buy BiSwap sell PCS
- spread collapses mid-flight:
  - ensure unwind behavior triggers
  - ensure pause when unsafe

### 17.4.3 Close While Executing
Simulate:
- execution started
- user requests close
- ensure no new trades
- ensure closure after execution
- withdraw full free balance at end

### 17.4.4 Reward Currency Switching
Simulate:
- user config to USDT
- profit event credits USDT
- user changes to LOOP (only when safe)
- next profits mint LOOP

---

## 17.5 Adversarial / Attack Simulations (Mandatory)

### 17.5.1 Reentrancy
- all external calls protected
- tests for reentrancy across:
  - claims
  - withdrawals
  - redemption

### 17.5.2 Price Manipulation (Pool Abuse)
Simulate:
- attacker manipulates low liquidity pool
- oracle sanity blocks trade
- if oracle disabled, slippage + liquidity depth stops trade

### 17.5.3 Sandwich Attacks
Simulate:
- swap executed with sandwich conditions
- ensure MEV buffer + profit threshold blocks marginal trades

### 17.5.4 Keeper Abuse
Simulate:
- keeper attempts:
  - withdraw funds (must fail)
  - bypass slippage/gas caps (must fail)
  - trade unsupported tokens (must fail)

---

## 17.6 Load / Scale Tests

### 17.6.1 Position Count
Test:
- 1,000 positions
- 10,000 positions

Ensure:
- UI loads
- keeper loop scales
- contract calls remain reasonable

### 17.6.2 Redemption Queue Load
Test:
- 10,000 redemption requests queued
- daily processing stable
- FIFO preserved

---

## 17.7 Must-Pass Acceptance Criteria (Launch Gate)

Launch is blocked unless:

- all unit tests pass
- all integration tests pass
- all adversarial tests pass
- coverage >= 95% critical modules:
  - settlement
  - redemption
  - reserve/floor
  - role controls

---

# 18. Deployment Plan (Testnet → Mainnet, Rollback)

This defines exactly how the dev must deploy and operate the system.

---

## 18.1 Deployment Stages (Locked)

Stage 0: Local + fork testing complete  
Stage 1: Testnet deployment + live keeper  
Stage 2: Internal alpha (limited wallets)  
Stage 3: Public beta (caps)  
Stage 4: Full mainnet launch  

---

## 18.2 Testnet Deployment Requirements

Deploy contracts to BSC testnet:
- all contracts from Section 13
- full role configuration
- timelock live

Run keeper continuously for 14 days.

---

## 18.3 Internal Alpha (Locked)

Alpha constraints:
- allowlisted wallets only
- maximum positions:
  - 50 positions
- max per position:
  - 500 USDT

Alpha must run 30 days with:
- at least 100 executions
- 20 redemptions
- 5 queued redemptions

---

## 18.4 Public Beta (Locked)

Beta constraints:
- open access
- caps:
  - max 1,000 active positions
  - max 2,500 USDT per position
  - max 250,000 USDT total TVL

Beta duration:
- 60 days minimum

---

## 18.5 Mainnet Launch

Only after:
- audit completed
- alpha + beta complete
- incident-free 30 days

---

## 18.6 Emergency Rollback Plan (Hard)

Non-upgradeable contracts means:
- rollback = pause + redeploy + migrate

System MUST support:
- GLOBAL PAUSE
- SUNSET state (no new deposits)
- allow withdrawals
- allow claims
- redemption disabled by default during emergency until SR verified

---

## 18.7 Admin Operational Runbook (Required Deliverable)

Dev MUST provide a runbook including:
- how to pause/unpause
- how to rotate keeper keys
- how to update parameters via timelock
- how to process queue manually if keeper fails
- incident checklist

---

# 19. Legal + Disclosures Requirements (Exact Text Blocks)

These disclosures MUST exist in UI and must be acknowledged.

---

## 19.1 Required Disclosure Blocks (Hard)

### 19.1.1 Core Disclosure
- “YieldLoop is a self-custody DeFi product.”
- “No insurance is provided.”
- “You can lose money.”
- “Smart contract risk exists.”
- “No guaranteed returns.”

### 19.1.2 Strategy Disclosure
- “YieldLoop uses spot trading and opportunistic PCS↔BiSwap arbitrage only.”
- “No lending, no leverage, no LP farming.”

### 19.1.3 Fee Disclosure
- “Performance fee is charged only on distributed profits.”
- “20% fee when claiming in USDT.”
- “17.5% fee when claiming in LOOP.”
- “No deposit fees. No withdrawal fees.”

### 19.1.4 LOOP Token Disclosure
- “LOOP is minted only from verified profit events.”
- “LOOP is redeemable to USDT according to the Redemption Engine rules.”
- “Bonus redemption is not guaranteed and may be reduced or queued.”

### 19.1.5 Seed Phrase Disclosure
- “If you lose your seed phrase, YieldLoop cannot recover your funds.”
- “YieldLoop does not custody your keys.”

---

## 19.2 Fiat Rail Disclosures (If Enabled)

If Transak/MoonPay enabled:
- “Fiat purchase/sale services are provided by Transak/MoonPay.”
- “YieldLoop does not custody fiat.”
- “Provider fees, KYC, and restrictions apply.”

---

## 19.3 Disclosure Enforcement (Hard)
Execution cannot begin unless:
- all 3 acknowledgements stored on-chain:
  - strategyAck
  - feeAck
  - riskAck

Config changes require re-ack.

---

# 20. Appendices (State Machines + Formulas + Reason Codes)

This section provides exact reference tables for dev implementation.

---

## 20A. Position State Machine (Recap)

CREATED → FUNDED → CONFIGURED → READY  
READY → EXECUTING_SPOT → SETTLING → READY  
READY → EXECUTING_ARB → SETTLING → READY  
READY → CLOSE_REQUESTED → CLOSING → CLOSED  
ANY → PAUSED  

---

## 20B. Key Formulas (Recap)

NetProfitUSDT =
  EndValueUSDT
- StartValueUSDT
- GasCostUSDT
- DEXFeeUSDT
- SlippageUSDT
- MEVLossUSDT

DistributableProfitUSDT (DP):
- Mode A: 0
- Mode B: NP * 0.50
- Mode C: NP

FeeUSDT:
- DP * 0.20 (USDT rewards)
- DP * 0.175 (LOOP rewards)

UserNetProfitUSDT:
- DP - FeeUSDT

LOOP minted:
- floor(UserNetProfitUSDT / FloorPrice)

CR:
- SR_USDT / (LOOP_Supply * FloorPrice)

CandidateFloor:
- SR_USDT / (LOOP_Supply * TargetCR)

Floor_next:
- max(Floor_current, CandidateFloor)

BonusMultiplier:
- 1 + TierStep * TierIndex
- capped by lock

---

## 20C. Reason Codes (Locked)

### Execution Failure Reason Codes
- QUOTE_STALE
- SLIPPAGE_TOO_HIGH
- GAS_TOO_HIGH
- LIQUIDITY_TOO_LOW
- PROFIT_TOO_LOW
- ORACLE_DEVIATION
- ROUTER_FAILURE

### Pause Reason Codes
- FAILED_EXECUTIONS
- GAS_SPIKE
- QUOTE_FAILURE
- SNAPSHOT_INVALID
- RESERVE_MISMATCH
- EXPOSURE_UNWIND_FAILURE
- ADMIN_EMERGENCY

### Floor Update Skip Reasons
- ZERO_SUPPLY
- EPOCH_NOT_REACHED
- SYSTEM_PAUSED
- RESERVE_MISMATCH

---

## 20D. Required Deliverables Checklist (Dev Sign-Off)

Developer MUST deliver:

Smart contracts:
- all contracts Section 13
- full test suite Section 17
- deployment scripts
- event indexer schema

Off-chain:
- keeper bot
- queue bot
- monitoring/alerts
- monthly email engine
- immutable logs

Frontend:
- Normie mode default
- advanced mode toggle
- fiat rails (feature-flagged)
- all screens Section 15

