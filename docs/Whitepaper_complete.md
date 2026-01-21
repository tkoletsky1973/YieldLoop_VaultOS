# YieldLoop — Builder Document

## Title Page 

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
- cap = 1.0150

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

# 2. User Vault Model (Accounts/Vaults, Positions, Labels, Allocation, Compounding, Rewards)

This section is the definitive build spec for how user funds are organized.
It includes the new requirement: **multiple labeled accounts/baskets** (Christmas fund, car, college, etc.).
The developer MUST implement this exactly. No alternate data model is allowed in v1.

---

## 2.1 Core Structure (Locked)

YieldLoop storage model is:

- **Wallet (User)**
  - can create **multiple Accounts** (also called Vaults internally)
  - each Account contains **multiple Positions**
  - each deposit creates a new Position

Definitions:

- **Account** (Normie Mode UX term)
- **Vault** (Advanced Mode / internal term)
- **Position** = single deposit container with its own strategy config and accounting

Hard rule:
- **Deposits NEVER merge.** Every deposit creates a new Position, attached to a selected Account/Vault.

---

## 2.2 Objects and Identifiers (Hard)

### 2.2.1 Wallet
A Wallet is the user’s connected address:
- userAddress

YieldLoop MUST treat the wallet as the sole identity.

No username/password account system exists for custody.

---

### 2.2.2 AccountVault (New Required Object)
Each wallet can create multiple AccountVault objects.

AccountVault must have:

- accountVaultId (bytes32)
- ownerAddress (address)
- createdTimestamp (uint64)
- status (enum: ACTIVE / FROZEN / SUNSET)
- positionCount (uint32)

Accounting totals:
- totalDepositedUSDT
- totalWithdrawnUSDT
- totalClaimedUSDT
- totalClaimedLOOP
- totalLifetimeNetProfitUSDT
- totalLifetimeFeesUSDT

AccountVault accounting MUST be additive sums of contained Positions.

---

### 2.2.3 Position Object (Per Deposit)
Each Position belongs to exactly one AccountVault.

Key:
- (accountVaultId, positionId)

Position must have:
- positionId (uint64, incremental per AccountVault)
- createdTimestamp
- config values
- execution state
- balances (free vs in-trade)
- claim balances
- close state
- lock state
- disclosure timestamps

---

## 2.3 Account Labels (User Purpose Buckets)

### 2.3.1 UX Purpose (Hard)
Accounts/Vaults MUST support labels to represent real-world buckets, e.g.:
- Christmas Fund
- New Car
- College
- House
- Vacation
- Emergency Fund

This is NOT optional. It is core to “feels like fintech”.

---

### 2.3.2 Label Storage Policy (Locked)
Locked decision:
✅ Account labels MUST be stored **off-chain** only.

Reason:
- avoid on-chain personal intent data
- reduce gas
- avoid string storage complexity

On-chain stores accountVaultId and accounting only.

---

### 2.3.3 Backend Label Schema (Hard)
Backend MUST store:

AccountLabelRecord:
- ownerAddress
- accountVaultId
- labelText (string, max 32 chars)
- createdTimestamp
- lastUpdatedTimestamp

Label rules:
- min length: 2
- max length: 32
- allowed: [A-Z a-z 0-9 space - _]
- no emojis
- no URL strings

If invalid:
- frontend blocks submission

---

### 2.3.4 Label Mutation Rules (Hard)
User MAY:
- rename label any time (off-chain only)
- create unlimited accounts

User MAY delete an Account label record only if:
- AccountVault status = SUNSET
- and contains zero Positions OR all Positions CLOSED

On-chain AccountVault must never be deleted.

---

## 2.4 Account Creation Rules (Hard)

### 2.4.1 Create Account
User action: Create Account

Creates:
- AccountVault (on-chain)
- Label record (off-chain)

New AccountVault default:
- status = ACTIVE
- positionCount = 0
- all accounting totals = 0

---

### 2.4.2 Preset Labels (Normie Mode Required)
When creating an Account, UI MUST show presets:

- Christmas Fund
- New Car
- College
- House
- Vacation
- Emergency Fund
- Other (custom text)

User can select preset or custom.

---

## 2.5 Deposit Rules (Updated)

### 2.5.1 Deposit Asset (Locked)
Deposit asset MUST be:
- USDT (BEP-20) only

Any other token deposit MUST revert.

---

### 2.5.2 Minimum Deposit (Locked)
MinDepositUSDT = 250

Deposit less than 250 USDT MUST revert.

---

### 2.5.3 Deposit Routing (Hard)
Deposit flow MUST include:

1) user selects an AccountVaultId
   OR creates a new Account
2) depositUSDT(accountVaultId, amount)

Deposit results:
- a NEW Position is created under that AccountVault
- positionState becomes FUNDED
- freeUSDT = amount
- positionId increments

Deposits MUST NOT:
- alter any existing Position
- merge balances into older deposits

---

## 2.6 Position Configuration Requirements (Hard)

Every Position MUST be configured before execution begins.

Config includes:

A) Allocation mix across:
- BTCB
- ETH
- BNB
- XRP
- SOL

B) Compounding mode:
- MODE_A: 100% Compound
- MODE_B: 50/50 Compound + Collect
- MODE_C: 100% Collect

C) Reward currency:
- USDT
- LOOP

D) Optional lock:
- OFF or VaultOS lock duration

---

### 2.6.1 Allocation Encoding (Hard)
allocationBPS[5] where index maps to:

0 BTCB  
1 ETH  
2 BNB  
3 XRP  
4 SOL  

Rules:
- sum(allocationBPS) = 10000 OR revert
- each entry must be <= 10000
- negative values impossible by type

---

### 2.6.2 Default Config (Locked)
Default settings for new Positions:

Allocation:
- BTCB 3000
- ETH  2500
- BNB  2500
- XRP  1000
- SOL  1000

Compounding:
- MODE_B (50/50)

Reward currency:
- LOOP (default)

Lock:
- OFF (default)

---

## 2.7 Execution Eligibility Rules (Hard)

A Position may enter READY state only if:

- deposit complete
- config saved
- disclosures acknowledged on-chain:
  - strategyAckTimestamp
  - feeAckTimestamp
  - riskAckTimestamp

If any disclosure missing:
- Position MUST remain CONFIGURED
- keeper MUST ignore it

---

## 2.8 Balance Partition Rules (Hard)

Positions MUST keep balances partitioned as:

- freeUSDT (withdrawable now)
- inTradeUSDT (not withdrawable)
- idleTokenBalances[5] (optional holding while between trades, not claimable)
- claimableUSDT
- claimableLOOP

Hard rule:
- claimable balances MUST NEVER be used for trading

---

## 2.9 Withdraw Rules (Hard)

### 2.9.1 Withdraw Free Funds Anytime
User may withdraw at any time:

withdrawFreeUSDT(accountVaultId, positionId, amount)

Rules:
- amount <= freeUSDT
- transfers USDT to wallet
- updates accounting

---

### 2.9.2 No Forced Withdrawal Blocking
The system must never prevent:
- withdrawal of freeUSDT
Even if:
- global pause active
- keeper down
- strategy paused

---

## 2.10 Close Rules (Hard)

### 2.10.1 Close Request
User may request close anytime:

requestClose(accountVaultId, positionId)

Effects:
- closeRequested = TRUE
- new executions prohibited
- keeper must stop initiating trades
- position transitions to CLOSE_REQUESTED

---

### 2.10.2 Close Completion
Position becomes CLOSED only when:
- openExecutionCount = 0
- inTradeUSDT = 0
- idleTokenBalances = zeroed OR swapped back to USDT

Then:
- all funds become freeUSDT
- user can withdraw

---

## 2.11 Position Updates and Reconfiguration (Hard)

User MAY update configuration ONLY if:

- positionState = READY or PAUSED
- openExecutionCount = 0
- closeRequested = FALSE

If config updated:
- disclosures MUST be re-acknowledged
- lastConfigChangeTimestamp updated

---

## 2.12 Claim Rules (Hard)

Claim flows exist for profits:

- claimUSDT(accountVaultId, positionId, amount)
- claimLOOP(accountVaultId, positionId, amount)

Rules:
- cannot claim more than claimable balance
- claimable balances decrease
- transfer to wallet occurs

---

## 2.13 Account-Level Reporting (Hard)

UI MUST support:

- wallet-level totals
- account-level totals (by label)
- position-level details

Must display:
- “Christmas Fund total balance”
- “Vacation Fund earnings”
etc.

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

# 9. Redemption Engine (Base + Dynamic Bonus + Caps + Queue) — UPDATED (LOCKED)

This section defines the LOOP → USDT redemption system with **NO ambiguity**.
This version incorporates the new rule:

✅ **Unlocked accounts have a dynamic bonus range of 1.0000 → 1.0150 ONLY**  
✅ Bonus applies ONLY when system reserve health is strong  
✅ Locked accounts can still reach up to 1.0500 cap (VaultOS)

Redemption is reserve-gated and designed to prevent bank-run behavior.

---

## 9.1 Redemption Purpose (Hard)

Redemption exists to:
- give LOOP a deterministic backed value path to USDT
- enforce reserve discipline
- provide conditional bonus payout when system health allows
- queue redemptions under stress instead of breaking the system

Redemption does NOT guarantee:
- instant payout at all times
- bonus payout
- profit

---

## 9.2 Redemption Inputs / Outputs (Locked)

### 9.2.1 Inputs
- LOOP_amount (user specified)

### 9.2.2 Output (Locked v1)
- USDT only (BEP-20)

YieldLoop MUST NOT redeem to:
- BNB
- any other stablecoin
- any basket

---

## 9.3 Redemption Preconditions (Hard)

A redemption request MAY execute immediately only if ALL are true:
- SystemStatus == ACTIVE
- reserveMismatchFlag == FALSE
- LOOP_amount > 0
- userBalanceLOOP >= LOOP_amount
- redemption does not violate MinCR (Section 9.10)

If any fail:
- redemption MUST be queued OR reverted (rules below)

Locked v1 decision:
✅ If it fails MinCR: QUEUE  
✅ If system paused or reserve mismatch: REVERT

---

## 9.4 Redemption Base Payout (Hard)

Base redemption uses FloorPrice only:

BaseUSDTOut =
  LOOP_amount * FloorPrice

This is ALWAYS true.

---

## 9.5 Bonus System Overview (UPDATED)

Redemption payout may include a bonus multiplier:

USDTOut =
  floor(LOOP_amount * FloorPrice * EffectiveMultiplier)

Where:

EffectiveMultiplier =
  min( BonusMultiplier, CapMultiplier )

BonusMultiplier depends only on:
- reserve health (CR bands)

CapMultiplier depends on:
- lock status

Hard truths:
- bonus can drop to 1.0000
- cap cannot be exceeded even if CR is higher
- payout always rounds DOWN

---

## 9.6 Collateralization Ratio (CR) Input (Hard)

CR definition:

CR =
  SR_USDT / (LOOP_Supply * FloorPrice)

Where:
- SR_USDT = ReserveVault USDT balance
- LOOP_Supply = total circulating LOOP supply
- FloorPrice = current monotonic floor

CR MUST be computed on-chain using current storage values.

---

## 9.7 BonusMultiplier (CR-Based Tiers) (Hard)

### 9.7.1 TierStep (Locked)
Bonus uses a fixed step size:

TierStep = 0.0075

This ensures:
- predictable bonus steps
- conservative reserve release

---

## 9.8 Bonus Tiers Table (UPDATED)

This is the global tier mapping for BonusMultiplierRaw.

CR Bands (Locked):

- Band 0: CR < 1.0000
- Band 1: 1.0000 ≤ CR < 1.0125
- Band 2: 1.0125 ≤ CR < 1.0250
- Band 3: 1.0250 ≤ CR < 1.0375
- Band 4: CR ≥ 1.0375

TierIndex assignment (Locked):
- Band 0 → TierIndex = 0
- Band 1 → TierIndex = 1
- Band 2 → TierIndex = 2
- Band 3 → TierIndex = 3
- Band 4 → TierIndex = 4

BonusMultiplierRaw:
- Tier 0: 1.0000
- Tier 1: 1.0075
- Tier 2: 1.0150
- Tier 3: 1.0225
- Tier 4: 1.0250

Formula form:
BonusMultiplierRaw =
  1.0000 + TierStep * TierIndex

---

## 9.9 CapMultiplier (UPDATED — Critical Change)

CapMultiplier controls the max bonus a user can receive.

### 9.9.1 Unlocked Cap (UPDATED / LOCKED)
Unlocked accounts (no VaultOS lock active) MUST have:

UnlockedCapMultiplier = 1.0150

Meaning:
- bonus is dynamic from 1.0000 → 1.0150
- tiers above 1.0150 are ignored for unlocked users

This creates the exact rule you requested:
✅ unlocked accounts only get bonus if system is doing well  
✅ max boost 1.015  
✅ otherwise 1.000 or 1.0075

---

### 9.9.2 Locked Caps (VaultOS)
If VaultOS lock is active and not expired, cap comes from LockEngine:

Cap schedule (unchanged):
- Unlocked: 1.0150
- 30d:  1.0250
- 90d:  1.0275
- 180d: 1.0300
- 1y:   1.0350
- 2y:   1.0400
- 3y:   1.0450
- 5y+:  1.0500
- 10y:  1.0500
- 25y:  1.0500

Hard max:
- 1.0500

---

### 9.9.3 Lock Expiration Rule (Hard)
When lock expires:
- CapMultiplier becomes UnlockedCapMultiplier (1.0150)
- lock history remains recorded but does not grant cap benefit

---

## 9.10 EffectiveMultiplier Computation (Hard)

EffectiveMultiplier =
  min(BonusMultiplierRaw, CapMultiplier)

Examples:

Unlocked user, strong CR:
- BonusMultiplierRaw = 1.0250
- CapMultiplier = 1.0150
- EffectiveMultiplier = 1.0150

Locked 3-year user, medium CR:
- BonusMultiplierRaw = 1.0225
- CapMultiplier = 1.0450
- EffectiveMultiplier = 1.0225

Stressed system:
- BonusMultiplierRaw = 1.0000
- any cap
- EffectiveMultiplier = 1.0000

---

## 9.11 Final Payout Formula (Hard)

USDTOut =
  floor( LOOP_amount * FloorPrice * EffectiveMultiplier )

Rounding rule (Locked):
✅ round DOWN always

---

## 9.12 MinCR Enforcement (Hard)

Redemption must NOT execute if it would cause CR to drop below MinCR.

Locked:
- MinCR = 1.0000

Compute post-redemption reserve:

SR_post = SR_USDT - USDTOut

Compute CR_post:

CR_post =
  SR_post / ((LOOP_Supply - LOOP_amount) * FloorPrice)

Rule:
- If CR_post < MinCR → queue redemption

Edge case:
- if LOOP_Supply - LOOP_amount == 0:
  - allow redemption if SR_post >= 0

---

## 9.13 Redemption Queue System (Mandatory Fallback)

### 9.13.1 Queue Type
- FIFO (First In First Out)

### 9.13.2 Queue When
Redemption MUST be queued if:
- immediate redemption violates MinCR

Redemption MUST REVERT if:
- system paused
- reserve mismatch
- user has insufficient LOOP
- LOOP amount = 0

---

## 9.14 Queue Entry Object (Hard)

QueuedRedemption:
- queueId
- userAddress
- submittedTimestamp
- loopAmountRequested
- capMultiplierAtSubmit
- status: QUEUED / PARTIAL / FILLED / CANCELLED

---

## 9.15 Queue Processing (Hard)

### 9.15.1 Processing Frequency
- once daily at 00:00 UTC
- keeper executes processing

### 9.15.2 Daily Budget (Locked)
Daily maximum redemption payout:

DailyRedemptionMaxPercentSR = 5%

Budget formula:
DailyBudgetUSDT = SR_USDT * 0.05

No more than this budget may be redeemed per day.

---

### 9.15.3 Partial Fill Rules
If a queued request is larger than remaining budget:
- partially fill
- burn proportional LOOP
- pay proportional USDTOut
- remaining LOOP stays queued

---

## 9.16 Redeemed LOOP Handling (Hard)

Redeemed LOOP MUST be burned.

Redemption steps:
1) transferFrom(user → RedemptionEngine, LOOP_amount)
2) burn LOOP_amount
3) transfer USDTOut (ReserveVault → user)

---

## 9.17 Redemption Event Requirements (Mandatory)

Contracts MUST emit:

- RedemptionRequested(user, loopAmount, timestamp)
- RedemptionExecuted(user, loopBurned, usdtOut, floorPrice, bonusMultiplierRaw, capMultiplier, effectiveMultiplier, timestamp)
- RedemptionQueued(queueId, user, loopAmount, capMultiplierAtSubmit, timestamp)
- RedemptionQueueFilled(queueId, user, loopBurned, usdtOut, remainingLoopQueued, timestamp)
- RedemptionQueueCancelled(queueId, user, timestamp)

---

## 9.18 Developer Checklist (Hard)

Developer MUST implement:

✅ UnlockedCapMultiplier = 1.0150 (dynamic 1.0000 → 1.0150 only)  
✅ Bonus tiers still computed globally using CR bands  
✅ EffectiveMultiplier = min(BonusMultiplierRaw, CapMultiplier)  
✅ MinCR enforcement queues unsafe redemptions  
✅ FIFO queue with daily SR budget cap  
✅ Burn LOOP on redemption  
✅ Full event coverage  

This section is final.

---

# 10. VaultOS Lock Engine (30 Days → 25 Years, Bonus Cap Control) — UPDATED (LOCKED)

This section defines YieldLoop’s lock mechanism (“VaultOS”).
VaultOS is NOT staking. It is NOT lending. It does NOT change strategy execution.
It ONLY affects the user’s **maximum redemption bonus cap**.

This section is updated to align with:
✅ Multi-Account/Vault model (Section 2)  
✅ Updated unlocked bonus cap: **dynamic 1.0000 → 1.0150** (Section 9)  
✅ Lock applies at the **Position level** (each deposit/plan can be locked differently)

---

## 10.1 Purpose of Locking (Hard)

VaultOS locking exists to:
- discourage fast exits during stress
- reward long-term capital behavior
- give YieldLoop a reserve stability tool
- create a clean, simple incentive:
  - “Lock longer → higher redemption cap”

Locking does NOT:
- guarantee profit
- prevent trading losses
- increase trading returns
- change fee rates
- grant governance

---

## 10.2 Lock Scope (Hard)

### 10.2.1 Lock Applies to Positions (Locked)
Lock is configured **per Position**, not per Account.

Reason:
- users may have multiple purposes under one Account
- each deposit may have different time horizon

Therefore:
- (accountVaultId, positionId) has its own lock state.

---

## 10.3 Lock State Variables (Hard)

Each Position MUST store:

- lockEnabled (bool)
- lockDurationSeconds (uint64)
- lockStartTimestamp (uint64)
- lockEndTimestamp (uint64)
- lockCapMultiplier (uint256 fixed-point 1e18)
- lockStatus (enum: OFF / ACTIVE / EXPIRED)

Hard rules:
- lockCapMultiplier is immutable after lock activation
- lockEndTimestamp = lockStartTimestamp + lockDurationSeconds

---

## 10.4 Allowed Lock Durations (Locked Menu)

Lock duration must be chosen ONLY from this list:

- 30 days
- 90 days
- 180 days
- 1 year
- 2 years
- 3 years
- 5 years
- 10 years
- 25 years

No other duration values are permitted.
If duration not in list → revert.

---

## 10.5 Lock Cap Multipliers (UPDATED / LOCKED)

### 10.5.1 Unlocked Cap (UPDATED)
If lock is OFF (or expired):

- CapMultiplier = **1.0150**

Meaning:
- unlocked users receive dynamic bonus tiers
- but never above 1.0150
- if system weak: 1.0000

This is the new default base behavior.

---

### 10.5.2 Locked Cap Schedule (LOCKED)
If lock ACTIVE:

- 30 days  → cap = 1.0250
- 90 days  → cap = 1.0275
- 180 days → cap = 1.0300
- 1 year   → cap = 1.0350
- 2 years  → cap = 1.0400
- 3 years  → cap = 1.0450
- 5 years  → cap = 1.0500
- 10 years → cap = 1.0500
- 25 years → cap = 1.0500

Hard maximum:
- CapMultiplier MUST NEVER exceed 1.0500

---

## 10.6 Lock Activation Rules (Hard)

### 10.6.1 When Lock Can Be Added
User may activate a lock ONLY if:

- Position is not closing:
  - closeRequested = FALSE
- Position is not executing:
  - openExecutionCount = 0
- Position state is READY or PAUSED
- lockStatus = OFF

This means:
- user cannot lock a position mid-trade
- user cannot lock after requesting close

---

### 10.6.2 Lock Activation Action
Contract function:
- enableLock(accountVaultId, positionId, durationEnum)

Effects:
- lockEnabled = TRUE
- lockDurationSeconds set from menu
- lockStartTimestamp = block.timestamp
- lockEndTimestamp computed
- lockCapMultiplier set from schedule
- lockStatus = ACTIVE

Event must emit.

---

## 10.7 Irreversibility Rules (Hard)

### 10.7.1 No Early Unlock (Locked)
Once lock is ACTIVE:
- user cannot unlock early
- user cannot shorten duration
- user cannot cancel
- user cannot transfer lock

This is absolute.

---

### 10.7.2 No Extensions (Locked v1)
Locked v1 policy:
✅ user cannot extend a lock once active

Reason:
- prevents manipulation and reduces edge cases
- keeps audits simpler

In future versions lock extension MAY be considered, but not v1.

---

## 10.8 Lock Expiration (Hard)

### 10.8.1 Expiration Trigger
Lock expires automatically when:
- block.timestamp >= lockEndTimestamp

Then:
- lockStatus becomes EXPIRED

### 10.8.2 Expired Lock Behavior
When EXPIRED:
- CapMultiplier = UnlockedCapMultiplier = 1.0150
- lockEnabled may remain true for historical record
- UI must show: “Lock expired”

Important:
- expired locks do NOT preserve higher caps

---

## 10.9 Interaction With Redemption Engine (Hard)

Redemption Engine must always request the cap:

CapMultiplier =
- lockCapMultiplier if lock ACTIVE
- 1.0150 otherwise

EffectiveMultiplier =
min(BonusMultiplierRaw, CapMultiplier)

This is mandatory.

---

## 10.10 Lock Visibility and Reporting (Hard)

UI MUST show on each Position:
- lock OFF / ACTIVE / EXPIRED
- days remaining
- cap multiplier

Additionally Account-level dashboard must display:
- “Locked balance” (sum of locked positions)
- “Unlocked balance”

---

## 10.11 Normie Mode vs Advanced Mode Lock UI (Hard)

### 10.11.1 Normie Mode Lock Options (Locked)
Normie Mode may only present:

- OFF (default)
- 1 year (cap 1.035)
- 3 years (cap 1.045)
- 5 years (cap 1.050)

Normie mode must not show long list.

### 10.11.2 Advanced Mode Lock Options
Advanced Mode must show full menu:

- 30d / 90d / 180d / 1y / 2y / 3y / 5y / 10y / 25y

---

## 10.12 Disclosures Required For Lock (Hard)

Before enabling lock, UI MUST require explicit acknowledgement:

- “Lock is irreversible.”
- “No early exit.”
- “Lock does not guarantee profit.”
- “Lock only increases redemption bonus cap.”

The acknowledgement MUST be stored off-chain AND (recommended) as an on-chain event.

Locked decision:
✅ must emit event LockDisclosureAck()

---

## 10.13 Mandatory Events (Hard)

Contracts MUST emit:

- LockEnabled(accountVaultId, positionId, durationSeconds, capMultiplier, lockEndTimestamp, timestamp)
- LockExpired(accountVaultId, positionId, timestamp)
- LockDisclosureAck(accountVaultId, positionId, timestamp)

---

## 10.14 Developer Checklist (Hard)

Developer MUST implement:

✅ Position-level locking  
✅ Fixed duration menu only  
✅ Updated unlocked cap = 1.0150  
✅ Lock cap schedule up to 1.0500  
✅ No early unlock  
✅ No lock extensions (v1)  
✅ Clear UI rules (Normie vs Advanced)  
✅ Events + reporting  

This section is final.

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

# 15. UI/UX Requirements (Accounts + Buckets + Locks + Fiat + Disclosures) — UPDATED (LOCKED)

This section defines EXACTLY what the YieldLoop DApp must look like and how it must behave.
It is NOT optional guidance. It is build requirements.

This version is updated to include:
✅ Multiple labeled Accounts/Buckets (Christmas Fund, Car, etc.)  
✅ Position-level locks (VaultOS)  
✅ Updated unlocked redemption cap: dynamic 1.0000 → 1.0150  
✅ Fiat rail integration stubs (Transak/MoonPay feature-flagged)  
✅ Normie Mode default + Advanced Mode toggle

---

## 15.1 UX Principles (Hard)

YieldLoop UX must:
- feel familiar like fintech (CashApp/Coinbase)
- hide DeFi jargon by default
- force disclosures where required
- keep money actions explicit and reversible where possible
- preserve self-custody boundaries without confusion

Hard rules:
- No hidden settings impacting funds.
- No “auto lock”.
- No “auto config changes”.
- No trading begins until the user acknowledges disclosures.
- Every important on-chain action MUST have a UI confirmation and a receipt view.

---

## 15.2 UX Modes (Locked)

The DApp MUST implement 2 UI modes:

1) **Normie Mode (Default)**
2) **Advanced Mode (Toggle)**

Rules:
- first time wallet connect → Normie Mode always
- Advanced Mode must be manually enabled
- user mode preference stored off-chain per wallet
- user can switch modes anytime

---

## 15.3 Terminology Rules (Locked)

### 15.3.1 Normie Mode Terms (Required)
The UI MUST use:

- Vault → **Account**
- AccountVault → **Account**
- Label → **Purpose**
- Position → **Plan**
- Allocation Weights → **Portfolio Mix**
- Compounding → **Reinvest Setting**
- Claim → **Withdraw Profits**
- Redeem LOOP → **Convert LOOP to USDT**
- FloorPrice → **Backed Value**
- BonusMultiplier → **Boost**
- Redemption Queue → **Payout Queue**
- Close Position → **Stop Plan**
- Free USDT → **Available Now**
- In Trade → **Active**

### 15.3.2 Advanced Mode Terms
Advanced mode may use the internal terms:
- Vault / Position / Allocation weights / Execution logs / CR / etc.

---

## 15.4 Mandatory Screens (Updated, Locked)

The DApp MUST include ALL screens below:

### Core:
1) Landing Page
2) Connect Wallet
3) Home Dashboard (Wallet Overview)
4) Accounts Dashboard (List of Accounts/Buckets)
5) Account Detail Page (by label/purpose)
6) Create Account (Purpose Bucket)
7) Deposit / Create Plan (Deposit USDT)
8) Configure Plan
9) Disclosures / Acknowledgements
10) Plan Detail Page
11) Withdraw Profits (Claim)
12) Convert LOOP → USDT (Redemption)
13) VaultOS Lock Screen (per Plan)
14) Stop Plan (Close)
15) Transaction History / Logs
16) Settings
17) Support / Help / Recovery

### Fiat rails (feature flagged):
18) Add Funds (Fiat Buy USDT)
19) Cash Out (Fiat Sell USDT)
20) Fiat Order Status Screen

No screen may be omitted.

---

## 15.5 Landing Page (Required Content)

Must include:
- YieldLoop name + logo
- “Deposit USDT, earn rewards in USDT or LOOP.”
- “Spot trading + arbitrage between PancakeSwap and BiSwap only.”
- “No LP. No lending. No leverage.”
- call-to-action:
  - Connect Wallet
- disclaimers:
  - no guaranteed returns
  - self-custody

---

## 15.6 Connect Wallet Screen (Hard)

Must support:
- MetaMask
- WalletConnect

Must show:
- connected address
- chain/network name
- if not BNB chain:
  - disable deposit/buy buttons
  - show “Switch to BNB Chain”
  - provide switch network button

Must show warning:
- “If you lose your seed phrase, we cannot recover your funds.”

---

## 15.7 Home Dashboard (Wallet Overview) (Hard)

This is the fintech-style wallet home.

### Must show:
- Total Wallet Value (USDT)
- Total Principal (USDT)
- Total Earnings (USDT equivalent)
- Total Claimable:
  - USDT
  - LOOP

### System stats ribbon:
- Backed Value (FloorPrice)
- Boost status:
  - show current “Boost Range”
  - Unlocked max = 1.015
  - Locked max = up to 1.050
- System Status:
  - ACTIVE / PAUSED / SUNSET

### Quick actions:
- Add Funds
- Create Account
- Start New Plan
- Withdraw Profits
- Convert LOOP to USDT

---

## 15.8 Accounts Dashboard (Buckets List) — REQUIRED (New)

Accounts are labeled buckets:
Christmas fund / car / college etc.

### Must show account cards list:
Each Account card displays:
- Purpose label (ex: “Vacation Fund”)
- Account Value (USDT)
- Claimable (USDT + LOOP)
- Locked Balance vs Unlocked Balance (USDT)
- Active Plans count
- Buttons:
  - View Account
  - Start Plan
  - Deposit

### Must include:
- Create New Account button

---

## 15.9 Create Account Screen (Purpose Bucket) (Hard)

### Required fields:
- label/purpose text input (2–32 chars)

### Required preset chips:
- Christmas Fund
- New Car
- College
- House
- Vacation
- Emergency Fund
- Other (custom)

### Rules:
- label stored off-chain only
- on-chain: AccountVault created and returns accountVaultId

### Buttons:
- Create Account
- Cancel

On success:
- route to Account Detail page

---

## 15.10 Account Detail Page (Hard)

Account detail shows bucket-level view.

### Must show:
- Account label (purpose)
- accountVaultId (advanced collapse)
- total principal
- total value estimate
- total earnings
- claimable USDT
- claimable LOOP
- locked vs unlocked breakdown
- list of Plans (Positions)

### Must include actions:
- Start New Plan (within this account)
- Deposit (creates new plan)
- Rename label (off-chain action)

---

## 15.11 Deposit / Start New Plan Screen (Hard)

Deposit flow must always start from:
- wallet OR selected account

User must choose:
- target Account (dropdown)
- deposit amount (USDT)

Rules:
- min deposit 250 USDT
- show USDT wallet balance
- show Approve button (allowance)
- show Deposit button

On deposit success:
- Plan created
- route to Configure Plan

---

## 15.12 Configure Plan Screen (Hard)

This config is per Plan (Position).

### Configuration includes (exactly):
A) Portfolio Mix  
B) Reinvest Setting  
C) Rewards Currency  
D) Optional Lock  

---

### 15.12.1 Normie Mode Configure (Locked Simplicity)
Normie mode MUST show:

**Portfolio Mix (3 options):**
- Conservative
- Balanced (default)
- Aggressive

**Reinvest Setting:**
- Reinvest all
- Half and half (default)
- Withdraw all profits

**Rewards Currency:**
- USDT (20% fee)
- LOOP (17.5% fee) default

**Lock Boost:**
- OFF (default)
- 1 year
- 3 years
- 5 years

Normie mode MUST NOT show:
- percentages sliders
- tier bands
- CR
- bps values

---

### 15.12.2 Advanced Mode Configure
Advanced mode MUST show:

- allocation sliders (BTCB/ETH/BNB/XRP/SOL)
- must sum to 100%
- compounding 3 options
- rewards currency 2 options
- lock full menu (30d–25y)

---

### 15.12.3 Submit Behavior (Hard)
Submit configuration must route into Disclosures screen (mandatory gate).
No trading is allowed before acknowledgements.

---

## 15.13 Disclosures / Acknowledgements Screen (Hard Gate)

User MUST acknowledge 3 disclosures before Plan becomes READY:

1) Strategy disclosure
2) Fee disclosure
3) Risk disclosure

Each disclosure requires:
- scroll box
- checkbox “I understand”
- signature button “Acknowledge”

Stored on-chain timestamps required.

Locked rule:
- Any config change requires re-acknowledgement.

---

## 15.14 Plan Detail Page (Hard)

Must show:

### Overview
- Plan ID
- Account label
- principal
- value estimate
- status badge

### Config
- portfolio mix OR allocation weights
- reinvest setting
- rewards currency
- lock status

### Balances
- Available Now (free USDT)
- Active (in-trade USDT)
- Claimable USDT
- Claimable LOOP

### Controls
- Withdraw Available Now
- Withdraw Profits (Claim)
- Convert LOOP → USDT (Redeem)
- Add Lock / View Lock
- Stop Plan (Close)

---

## 15.15 Withdraw Profits (Claim Screen) (Hard)

Shows:
- claimable USDT
- claimable LOOP

Buttons:
- Withdraw USDT profits
- Withdraw LOOP profits

Rules:
- amount default max
- show tx hash receipt on completion

---

## 15.16 Convert LOOP → USDT (Redemption Screen) (UPDATED)

Must display:

### Required live outputs:
- Backed Value (FloorPrice)
- Current Boost (BonusMultiplierRaw)
- User’s Cap:
  - unlocked max = 1.015
  - locked max based on lock
- Effective Multiplier = min(boost, cap)
- Expected USDT out

### User actions:
- enter LOOP amount
- button: Convert

### Must show queue behavior:
- “Instant” if passes MinCR
- “Queued” if would violate MinCR

Warnings:
- “Boost depends on reserve health and can drop.”
- “Queued payouts process daily in order.”

---

## 15.17 VaultOS Lock Screen (Per Plan) (Updated)

Lock UI must show:
- lock status OFF / ACTIVE / EXPIRED
- remaining time
- cap multiplier benefit

Normie lock choices:
- OFF
- 1y / 3y / 5y

Advanced lock choices:
- full menu 30d–25y

Before enabling lock:
- irreversible confirmation modal required

---

## 15.18 Stop Plan / Close Screen (Hard)

Must explain:
- stops new trades
- waits for active trades to close
- funds become available afterward

Buttons:
- Stop Plan
- Withdraw Available Now (if any)

Must not promise timeline.

---

## 15.19 Transaction History / Logs (Hard)

Must show:

At wallet level:
- deposits
- withdrawals
- claims
- redemptions
- queue fills

At plan level:
- executions
- settlements
- fees
- LOOP minted

Each row:
- timestamp
- type
- amounts
- tx hash + explorer link

---

## 15.20 Settings Screen (Hard)

Must contain:
- current mode toggle Normie/Advanced
- disclosure links
- supported assets/venues
- system parameters (read-only)
- security warnings

Must include:
- official support email
- official discord link
- anti-scam warning

---

## 15.21 Support / Recovery Screen (Hard)

Must include:

- “YieldLoop cannot recover seed phrases.”
- “If seed phrase lost, funds inaccessible.”
- best security practices
- how to recognize official links
- monthly contact email policy:
  - user must provide contact email (optional in UI)
  - email stored off-chain
  - used for monthly reports + security notices

---

## 15.22 Fiat Rails Screens (Feature Flag) (Hard)

If FiatRailsEnabled = ON:

### Add Funds Screen
- Buy USDT (BEP-20) using Transak/MoonPay
- clear provider boundary disclosure

### Cash Out Screen
- Sell USDT to bank via provider

### Fiat Order Status Screen
- shows status: pending / completed / failed
- shows provider order ID
- shows “contact provider” links

If FiatRailsEnabled = OFF:
- show Coming Soon
- no dead buttons

---

## 15.23 Developer Checklist (Hard)

Developer MUST deliver:

✅ Wallet dashboard + Accounts dashboard  
✅ Create/Rename Account labels (off-chain)  
✅ Account detail grouping  
✅ Plan creation per account  
✅ Position-level lock UI + controls  
✅ Redemption screen showing 1.015 unlocked cap  
✅ Full disclosures gate  
✅ Full transaction logs  
✅ Support/recovery flow  
✅ Fiat rails screens behind flag  

This section is final.

---
  
# 16. Parameter Defaults (Full List — Locked v1) — UPDATED (LOCKED)

This section defines ALL default parameters the developer must implement.
Every parameter MUST be stored on-chain (where applicable), readable by UI, and changeable ONLY via timelock unless explicitly immutable.

This version is updated for:
✅ Multiple Accounts/Vaults per wallet (AccountVault object)  
✅ Updated redemption rules: unlocked dynamic cap is **1.0150**  
✅ Lock caps unchanged (up to 1.0500)  
✅ Normie Mode preset portfolio mixes (fixed allocations)

---

## 16.1 Parameter Storage Policy (Hard)

### 16.1.1 On-Chain Required Parameters
All monetary/safety rules MUST be stored on-chain:

- fees
- guardrails
- thresholds
- floor targets
- redemption rules
- queue rules
- lock cap schedule

### 16.1.2 Off-Chain Required Parameters
The following MUST be stored off-chain only:

- Account labels (purpose text)
- user contact email
- fiat order metadata (provider IDs)

Hard rule:
- **No PII stored on-chain.**

---

## 16.2 Core Product Defaults (Locked)

- ChainIdAllowed = BNB Chain mainnet chainId
- DepositAsset = USDT (BEP-20)
- DepositDecimals = 18 (USDT BEP-20 standard)
- MinDepositUSDT = 250

Deposits that do not meet criteria MUST revert.

---

## 16.3 Account/Vault Model Defaults (Updated)

- MaxAccountsPerWallet = UNLIMITED (no cap)
- PositionsPerAccount = UNLIMITED (no cap)
- DepositCreatesNewPosition = TRUE (locked)
- AccountLabelsStoredOnChain = FALSE (locked)
- AccountLabelMaxChars = 32 (off-chain enforcement)

---

## 16.4 Supported Assets (Locked)

Supported assets list is fixed, ordered:

AssetIndex mapping:
- 0 = BTCB
- 1 = ETH
- 2 = BNB
- 3 = XRP
- 4 = SOL

Tokens outside this list MUST be rejected by:
- allocation config
- trading executor
- any swap routes

---

## 16.5 Strategy Defaults (Locked)

- EnabledStrategy_Spot = TRUE
- EnabledStrategy_Arbitrage = TRUE
- EnabledStrategy_LP = FALSE
- EnabledStrategy_Lending = FALSE
- EnabledStrategy_Leverage = FALSE

Venues:
- AllowedVenues = {PCS, BiSwap} only

---

## 16.6 Default Allocation + Normie Presets (Updated)

### 16.6.1 Default Allocation (Advanced Mode Default)
DefaultAllocationBPS:
- BTCB 3000
- ETH  2500
- BNB  2500
- XRP  1000
- SOL  1000

### 16.6.2 Normie Portfolio Mix Presets (Locked)

These are fixed presets in UI config:

**ConservativePresetBPS**
- BTCB 3500
- ETH  2500
- BNB  2500
- XRP  1000
- SOL   500

**BalancedPresetBPS (Default)**
- BTCB 3000
- ETH  2500
- BNB  2500
- XRP  1000
- SOL  1000

**AggressivePresetBPS**
- BTCB 2000
- ETH  2500
- BNB  2500
- XRP  1500
- SOL  1500

---

## 16.7 Compounding Defaults (Locked)

CompoundingMode enum:

- MODE_A = 100% Compound
- MODE_B = 50/50 Compound + Collect
- MODE_C = 100% Collect

Defaults:
- DefaultCompoundingMode = MODE_B

DP (Distributable Profit) rules:
- MODE_A DP = 0
- MODE_B DP = 50% of NetProfit
- MODE_C DP = 100% of NetProfit

---

## 16.8 Reward Currency Defaults (Locked)

RewardCurrency enum:
- USDT
- LOOP

Defaults:
- DefaultRewardCurrency = LOOP

---

## 16.9 Fee Model Defaults (Locked)

Fee applies ONLY to DistributableProfitUSDT.

Fee rates:
- FeeRate_USDT = 0.20
- FeeRate_LOOP = 0.175

Fee splits (must sum 10000 BPS):
- DevOpsSplitBPS = 2500
- MarketingSplitBPS = 2500
- ReserveSplitBPS = 2500
- LoopLabsSplitBPS = 2500

Fee split changes:
- allowed only by timelock
- TimelockDelaySeconds = 172800 (48 hours)

---

## 16.10 Trading Guardrails Defaults (Locked)

### 16.10.1 Execution Limits
- MaxExecutionSizePercent = 10%
- MaxExecutionSizeUSDT = 750
- MaxArbSizePercent = 7.5%
- MaxArbSizeUSDT = 500

### 16.10.2 Slippage / Quote
- MaxSlippagePerLegBPS = 30
- QuoteTTLSeconds = 15

### 16.10.3 Liquidity
- MinimumLiquidityDepthUSDT = 50000

### 16.10.4 Drift
- MaxDriftBPS = 250

### 16.10.5 Gas
- MaxGasPerExecutionUSDT = 1.25

---

## 16.11 Profit Threshold Defaults (Locked)

Expected net profit thresholds:
- MinExpectedProfitSpotUSDT = 0.50
- MinExpectedProfitArbUSDT = 1.50

Buffers:
- MEVBufferSpotUSDT = 0.25
- MEVBufferArbUSDT = 0.50
- SlippageBufferUSDT = 0.25

If profit after buffers fails threshold:
- execution MUST not run

---

## 16.12 Failure Threshold Defaults (Locked)

Rolling failure windows:
- MaxFailedExecutionsPer10Min = 3
- FailureCooldownSeconds = 600 (10 minutes)
- QuoteFailurePauseThreshold = 3

Pause behavior:
- when threshold exceeded:
  - position paused
- if systemic:
  - global pause

---

## 16.13 Oracle Sanity Defaults (Locked)

Oracle use:
- OracleEnabled = TRUE

Deviation cap:
- OracleDeviationMaxBPS = 500 (5%)

If deviation exceeded:
- trade abort
- failure counter increments

---

## 16.14 Reserve + Floor Defaults (Locked)

Reserve:
- ReserveAsset = USDT only
- SRWithdrawEnabled = FALSE (hard)

Collateralization:
- TargetCR = 1.05
- MinCR = 1.00

Floor:
- InitialFloorPrice = 1.00
- FloorEpoch = Monthly
- FloorMonotonic = TRUE

---

## 16.15 Redemption Defaults (UPDATED)

### 16.15.1 TierStep
- TierStep = 0.0075

### 16.15.2 Bonus Tiers by TierIndex
TierIndex → BonusMultiplierRaw:
- 0 → 1.0000
- 1 → 1.0075
- 2 → 1.0150
- 3 → 1.0225
- 4 → 1.0250

### 16.15.3 CR Band Thresholds (Locked)
Bands:
- Band0: CR < 1.0000 → Tier 0
- Band1: 1.0000 ≤ CR < 1.0125 → Tier 1
- Band2: 1.0125 ≤ CR < 1.0250 → Tier 2
- Band3: 1.0250 ≤ CR < 1.0375 → Tier 3
- Band4: CR ≥ 1.0375 → Tier 4

### 16.15.4 Cap Rules (UPDATED)
- UnlockedCapMultiplier = 1.0150 (locked)
- Locked cap multipliers from LockEngine schedule
- EffectiveMultiplier = min(BonusMultiplierRaw, CapMultiplier)

### 16.15.5 Redemption Output
- RedeemOutputAsset = USDT only
- Redeemed LOOP must burn

---

## 16.16 Redemption Queue Defaults (Locked)

Queue:
- enabled = TRUE
- type = FIFO

Daily processing:
- QueueProcessingTimeUTC = 00:00

Daily budget:
- DailyRedemptionMaxPercentSR = 5%

Partial fills:
- Enabled = TRUE

Cancellation:
- Allowed ONLY if:
  - status == QUEUED
  - and no partial fill has occurred

---

## 16.17 VaultOS Lock Defaults (UPDATED alignment)

### 16.17.1 Lock Enabled Default
- DefaultLock = OFF

### 16.17.2 Lock Duration Menu (Locked)
Allowed durations:
- 30d, 90d, 180d, 1y, 2y, 3y, 5y, 10y, 25y

### 16.17.3 Lock Caps Schedule (Locked)
- 30d  = 1.0250
- 90d  = 1.0275
- 180d = 1.0300
- 1y   = 1.0350
- 2y   = 1.0400
- 3y   = 1.0450
- 5y+  = 1.0500
- 10y  = 1.0500
- 25y  = 1.0500

### 16.17.4 Lock Behavior
- No early unlock = TRUE
- No extension = TRUE (v1)
- Lock applied per Position = TRUE

---

## 16.18 Fiat Rails Defaults (Feature Flag)

- FiatRailsEnabled = FALSE (default)
- SupportedFiatProviders = {Transak, MoonPay}
- SupportedFiatAssetBuy = USDT (BEP-20) only
- SupportedFiatAssetSell = USDT (BEP-20) only
- MinFiatBuyUSD = 260

---

## 16.19 Notification Defaults

- MonthlyEmailEnabled = TRUE
- MonthlyEmailSendDay = 1st of month
- MonthlyEmailSendLocalTime = 12:00 local time
- SecurityAlertsEnabled = TRUE

---

## 16.20 Developer Checklist (Hard)

Developer MUST implement:

✅ All parameters as constants or timelock-settable values  
✅ Updated UnlockedCapMultiplier = 1.0150  
✅ CR thresholds and tier mapping exactly  
✅ Lock cap schedule up to 1.0500  
✅ Normie preset mixes exactly as defined  
✅ Fiat rails behind feature flag  
✅ No on-chain label storage  

This section is final.

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

