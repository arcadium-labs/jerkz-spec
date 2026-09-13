# Spec conformance

The working specification lives in the private repo **arcadium-labs/jerkz-spec** (local clone: `../spec`).
This file pins the spec commit the code was last reconciled against and records, section by section,
what is built, what deliberately deviates, and what is deferred. Update it in the same commit as any
code change that moves a row, and re-pin after every spec pull.

| | |
|---|---|
| Spec version | v0.21 |
| Spec commit | `dae4147` (2026-09-12; v0.19 → v0.21: Mint/Burn fees, one-action work with the hiring attempt inside the shift, lifetime XP and XP-adjusted odds) |
| Reconciled on | 2026-09-13 (code updated the same day; see "v0.21 delta" below) |
| Demo scope | "barebones version for our first demo": daily shift model, injected wallets, Payroll-lite |

Workflow: `git -C ../spec pull`, read the diff, update this table, then change code. Reference arithmetic
in spec §6.2/§6.4/§7.1 is the test oracle for Payroll pacing; `prd-model.mjs` is retired.

Status keys: **built** (matches), **deviates** (intentional, with reason), **partial**, **deferred**, **external**.

| Spec | Code | Status | Notes |
|---|---|---|---|
| §1 Product contract | `token/JerkzToken`, `core/JerkzNFT`, `core/HybridVault` | built (code); live vault on v0.19 fees | 1B fixed supply, 10k permanent ids, ERC-2981 2%. v0.21 fees in code: 1 USDC Mint (`wrapFee`) and 1 USDC Burn (`unwrapFee`, collected separately in `redeemNFTs`, earned only on an atomic batch, `FeeEarned(MINT/BURN)` receipts, `previewRedeem`). The live testnet vault still charges 2 USDC / free until the next vault cycle (a vault redeploy re-mints the collection). O1 launch and burnable-token path are external gates (§13.2). |
| §2.1 Conversion lifecycle | `core/HybridVault`, `core/JerkzNFT` | deviates | One-transaction wrap with push delivery in the randomness callback and per-request randomness instead of 60-second frozen batches and a separate `processConversionBatch`. `claimAssignedNFT` is the retry path. Kept deliberately for UX; revisit only if the VRF forces batching. |
| §2.2 Randomness | `randomness/DrandSource` + `randomness/drand/DrandVerifier`, `lib/RandomLib` | built | drand evmnet beacon verified on chain (BN254, ~253k gas), request bound to a future round, permissionless `fulfill`, timed or operator-selected round per consumer, `isPending` false once the beacon may be public; see `DRAND-SPEC.md` for the trust model. Domain-separated rejection sampling unchanged. The operator commit-reveal (`OperatorCommitReveal`) and the Pyth Entropy adapter are retired once every consumer points at the drand source (`setSource`, council-delayed). `refundUncommittedRequest` is refused once the source holds or may hold a word. |
| Deck: RWA packs ("404" wrapper reused) | `core/PackCollection`, pack `HybridVault` (Kind.Pack) | built (testnet stand-in) | cirBTC packs at 0.000005 cirBTC each (500 raw units at 8 decimals), 100 packs, 1 USDC wrap fee, sized under the 0.93 USDC a depositor earns per prize consumed at the 1 USDC spin price. CRCL and NVDA packs are further instances of the same wrapper once those tokens exist on Arc. |
| §2.3 Wrapper and metadata | `core/RarityConfig`, `ui/api/meta` | partial | Weighted independent title/expression/variant draws built; placeholder catalog (3 variants per branch) until AKLO delivers the ~100k options and IPFS root. |
| §3.1 Tier parameters | `work/CareerLib` | built | Mint employment status by title, base hire odds, pay factors, base strike odds, thresholds match the v0.21 table; warm-ups removed. |
| §3.2 One-action work and hiring | `work/EmploymentRegistry`, `work/WorkRegistry` | built (code) | `enrollShift` accepts an unemployed Intern–Director; `onShiftAccepted` commits one `HiringAttempt` per shift (compatible job from the committed mapping = lowest active posting whose tier mask includes the title; hire odds frozen from title and XP); `settleEntry` on the shift's first entry resolves it with that entry's cohort word (`JOB_APPLICATION` domain) before the performance draw; a failed search settles every entry of the shift as **No Work Found** (zero weight, no discipline, XP scheduled); a 48-hour shift rolls once and day two waits for day one (`HiringUnresolved`). Executives EMPLOYED from birth with a standing appointment (`defaultJob`, bound on first work), Founder SELF_EMPLOYED, VC never works. Unresolved search cannot queue a successor or be bypassed by the shop. Cancel/forfeit of the first entry abandons the attempt. No `applyForJob`/`appoint`; `hireNow` (shop) remains. Reapply delay after firing/resignation and the notice-entry guard (review #17) gate the next search. Deviation kept: daily shifts (one cohort per day) instead of hourly starts, so "the first entry's cohort" is the day's cohort. |
| §3.3 Career progression and XP | `work/EmploymentRegistry`, `work/CareerLib` | built (code) | Lifetime XP: one per completed originally funded entry including No Work Found, No Pay, Founder flops, the firing entry and notice; careerStreak / promotionProgress only while employed after ordered discipline. `experienceBonusBps = min(2000, floor(xp/5)×100)`, `effectiveHireBps = min(9900, base + bonus)`, `effectiveStrikeBps = floor(base × (10000 − bonus) / 10000)`, both frozen at acceptance (`Entry.strikeBps`, `HiringAttempt.effectiveHireBps`). Thresholds, resets, day-ordered application, promotion gate unchanged. Gap: promotion keeps the same posting instead of re-binding the title-to-role mapping. |
| §4.1 Hourly starts | `work/WorkRegistry` | deviates (demo) | Daily model: entries start at 09:30 NY, enrollment opens `enrollOpensBefore` (testnet 7 days) before the day. The :30 hourly grid, "clock in anytime" and the exact next-start rule are deferred with §6.3. |
| §4.2 Durations and custody | `work/WorkRegistry` | built (for 09:30 starts) | 8/16/24/48 with weights 100/175/200/200+200; 24/48 end at the next day start(s), which equals `NextLocalDay` for a 09:30 start; one entry per token id per date; custody by `transferFrom`. |
| §4.3 Successors, recovery, exits | `work/WorkRegistry` | partial | Successor chaining when the current shift end is at or before the next start; matured withdraw never waits on randomness; overtime leave = next day (equals the 09:30-boundary rule for 09:30 starts). Emergency exit is an owner `cancelShift(shiftId, recipient)` while paused that keeps the permanent day ledger; `forfeitStaleEntry` releases an entry the settler never settled after 7 days; `rescue` returns stray tokens. The recorded-fault authorization model is deferred with §12. |
| §5.1 Stress and recovery | `work/WorkRegistry`, `work/CareerLib` | built | Table values, spouse ×0.8 (rounded half-up; identical to floor for every v1 delta), bands, risk frozen from the resulting stress after the booked workload, burnout latch at 80 / clear below 60, lazy idle recovery. |
| §5.2 Weekend and JUICE | `work/WorkRegistry` (shop hooks) | partial | Weekend passes, Monday crash with cleared markers, `clearCooldowns` built as SHOP-role hooks. JUICE purchase, burn and pass selection are §9 (deferred). |
| §5.3 Discipline and settlement | `work/WorkRegistry.settleEntry`, `EmploymentRegistry` | built (code) | Hiring gate first, then performance and discipline domains from the same cohort seed; a failed gate has no mistake or discipline; half weight, graduated strikes at the XP-adjusted odds, two-strike firing, clean-streak clearing, reapply `max(notification, lastCommittedWorkDay) + 2`, two-entry backlog. |
| §6.1 Income and conversion | — | deferred | No fee converter; treasury is funded by direct `fund()` deposits of the stand-in. |
| §6.2 Daily pacing | — | deferred | Formulas verified to reproduce §6.4 exactly (Python, 2026-09-10); to be implemented in Solidity with those numbers as tests. Demo uses owner-published budgets. |
| §6.3 Four-hour groups | `payroll/Payroll` | deviates (demo) | One cohort per Payroll day instead of ceil(H/4) groups. |
| §6.4 Reference scenarios | — | verified | Reproduced from §6.2; becomes the pacing test oracle. |
| §7.1 Lottery and allocation | `payroll/SlotBook`, `payroll/Payroll` | built (code) | 10,000-slot books sampled without replacement (20/60/19/1 and 70/20/8/2), every accepted entry draws in canonical order; a failed hiring gate keeps its draw at zero weight (`Award.noWorkFound`, `previewClaim` "no work found"); weight = effective duration weight × pay factor, 60/40 split, 10% daily cap, unused returns to the free reserve. |
| §7.2 Cohort settlement and claims | `payroll/Payroll` | built | FUNDED → SEALED → RANDOMNESS_PENDING → ASSESSING → ASSIGNING → FINALIZED, sealing is O(1) (entries read by index), owner `rerequestSeed` after `seedDeadline`, ≤50 entries per call (assessment; assignment takes up to 400), claims at max(original shift end, finalizedAt), ≤20 entries per claim, no Friday gate or advance. |
| §8 VC Payroll | `JerkzNFT` checkpoint hook (`IVCPayroll`) | deferred | 5% split applied; the VC share returns unused at finalization (N = 0). Index and checkpoints not built. |
| §5.2 JERK JUICE, §9.1 shop, §9.2 instant hire | `shop/ItemShop` over the SHOP-role hooks | partial | v0.21 §9.2: `buyHireNow` is the automatic compatible appointment; refused while an accepted job-search shift is unresolved; the optional same-transaction purchase inside `putToWork` is deferred (two transactions on testnet). | Testnet consumables at spec prices: JUICE (reset + optional pass for this or next NY week, not inside an unmatured shift), Vegas Wedding (three personas, one perk, a different persona costs the same again, the same one is refused), Divorce Papers (catalog-bound, free at v1), College Degree / Purchased Referral / Bribe a CEO (§9.2 rules enforced by `hireNow`). Burns are `burnFrom` on the test JERKZ and verified by supply delta. Deferred: wardrobe, cosmetics and appearance variants (need the art catalog), executive referral campaigns (§9.3), and the production burn path (O1 token has no burn, external gate). |
| §10.1–10.3 Metadata | `JerkzNFT` (ERC-4906, revisions), `ui/api/meta`, `ui/api/catalog` | partial | State-derived JSON served from chain state; placeholder SVG catalog; no restricted publisher or IPFS pinning yet. |
| §10.4 App views | `ui/src/pages/{Swap,Work,Gacha}` | partial | Conversion, job board, work desk, payroll and pay slips exist; portfolio, HR and shop views pending. |
| §10.5 Read API and indexer | `ui/api/meta/[id]` | partial | Only the metadata route; no indexer. |
| §10.6 Circle integration | — | deferred | Injected wallets only for the demo; Wallets, CCTP and Gateway are a later milestone with their own evidence. |
| §11.2 External methods | various | partial | Built under these names: `requestNFTs`, `redeemNFTs(ids, gens, recipient, maxFee, deadline, configHash)` (payable Burn), `previewConvert` + `previewRedeem` (spec: `previewConvert(quantity, direction)`), `claimAssignedNFT`, `refundUncommittedRequest`, `applyEmploymentResults`, `resign`, `previewShift` (exposes `hiring`, `hireBps`, `strikeBps`), `enrollShift` (spec: `putToWork`; no `optionalHire` parameter yet), `withdrawNFT`, `publishEpoch`, `sealCohort`, `processCohort`, `previewClaim`, `claimStock`, `hiringAttempt`, `employmentStatus`, `defaultJob`. `resolveHiring` is WORK-only, driven by `settleEntry` (the keeper's payroll pass) rather than a permissionless standalone call. Removed per v0.21: `applyForJob`, `requestApplicationResult`, `cancelApplication`, `appoint`. Missing: `sealConversionBatch`/`processConversionBatch` (by design, §2.1), `consumeJerkJuice`, `closeIncomeDay`, `convertFees`, item and VC methods. |
| §12 Operations and admin | — | deferred | Testnet contracts are owned by the deployer EOA; multisig, timelock and guardian roles come before live funds. |
| §13.1 Verification | `test/` (217 tests) | partial | Covers built rows incl. v0.21 hiring gate, No Work Found, 48-hour single roll, XP thresholds (4/5, 99/100, cap, 99% ceiling, strike rounding), Burn fee quote/atomicity; the list in §13.1 is the backlog for the rest. |
| Gacha | separate repo `arcadium-labs/jerkz-gacha` (its `GACHA-SPEC.md` v0.3) | built | Separate product; not an MVP dependency. Depends on this repo as a submodule. v0.3 prices spins at the draw's expected value × (1 + 100% surcharge) with odds inverse to prize value, after the FWA review; the deck's 93/5/2 split is kept on the full price. |

## Review 2026-09-13 (pre-production audit) — status of the critical/high fixes

| # | Finding | Fix | Deployed on testnet |
|---|---|---|---|
| 1 | Gacha spins priced on the pre-draw pool | Ticket fixed at settlement, escrow up to `maxPrice`, void + refund past the limit (jerkz-gacha v0.5) | yes: GachaMachine v5 `0x3c0b…5DaF` |
| 2 | Undeliverable prize bricks the machine | Delivery in try/catch, `undelivered` + `deliver()` | yes: same |
| 3 | Pyth adapter catch starved / pending after CallbackFailed | `CALLBACK_RESERVE`, Entropy-aware `isPending`, CEI retry, zero gas refused | n/a (Entropy not on Arc testnet) |
| 4 | Council 1-of-N with no delay | AdminCouncil v2: guarded selectors and admin removal via schedule/execute with a delay, any admin cancels | yes: council v2 `0x8fd3…6B2E` owns everything |
| 5 | Deployer key concentration | Payroll publisher role for the keeper key; council owns Payroll and randomness; treasuries still deployer on testnet | yes: r5 Payroll v2 `0x9F87…8Dbc` (publisher = deployer key for now) |
| 6 | forfeitStaleEntry strips a sealed cohort | Gate on `IEpochFunding.isSettling`; `rerequestSeed` permissionless after the deadline; deadline floor | yes: r5 WorkRegistry v3 `0x0EC9…9DCD` |

### Mediums (review 2026-09-13)

| # | Finding | Fix | Deployed on testnet |
|---|---|---|---|
| 7 | Pyth `isPending` blind to Entropy state | Entropy-aware `isPending` (with #3) | n/a until Entropy is on Arc |
| 8 | Reveal seed visible in the mempool after a deadline | Accepted for the operator source (testnet only): the keeper reveals within seconds and deadlines are 1 h / 1 day; production uses a VRF where the word is not in calldata | — |
| 9 | Keeper marks a reverted reveal as revealed | receipt status checked before recording | yes (VPS keeper) |
| 10 | Concurrent seed commits | commits serialized | yes |
| 11 | Dead commitments cannot be skipped | `OperatorCommitReveal.discard(from,to)` | code only; the live source predates it (redeploy needs `setSource` on 4 consumers) |
| 12 | Shared Entropy fee pot drained by the free hiring draw | per-consumer fee budgets in `PythEntropySource` | n/a until Entropy is on Arc |
| 13 | Uncapped single-key valuer | `GachaMachine.maxPrizeValue`, `TitleValuer` bounds, valuer setters and the cap guarded (delayed) on the council | yes: GachaMachine v6, guards scheduled |
| 14 | Weight-0 prize bricks the machine | `ZeroWeight` at deposit, cap < 1e36 | yes: v6 |
| 15 | Settlement gas with staged activations | `ACTIVATE_PER_SETTLE` 2, keeper `activateStaged` duty | yes: v6 + keeper |
| 17 | `applyForJob` with open work | `_requireNoWork` | code only; needs an EmploymentRegistry redeploy (cascades to ItemShop) |
| 18 | Cancel grace from `readyAt` | from `requestedAt` | code only; same redeploy |
| 20 | Keeper settle window fixed at 3 days, one failure aborts the tick | walks back to the last finalized day, per-day try/catch | yes |
| 21 | One batch per tick | up to 12 batches per tick | yes |
| 22 | Vault refund pushes native fee | `feeRefundOf` + `claimFeeRefund` | code only; needs a HybridVault redeploy (the Jerk vault holds the collection: next vault cycle) |

### Low / info (review 2026-09-13)

Verified 34 of 39; #28 and #43 refuted (already fixed by earlier work), #40/#51/#52 downgraded (council v2 delays cover the instant-repoint part). Fixed in code, all tests green; the items on contracts that are not redeployed ride with their next cycle:

| Area | Fixed | Deployed |
|---|---|---|
| HybridVault | #23 party checks on `claimAssignedNFT`, #24 supply must match the collection, #27 recipient can never be the vault or the collection, #53 `ERC20Rescued` | next vault cycle |
| RarityConfig | #25 `initialStress` ≤ 79, #26 re-activation waits `MIN_DELAY` from the last switch | next rarity deploy |
| PackCollection / JerkzNFT | #53 `VaultSet`, `UnderlyingSet`, `VCPayrollSet` | next deploys |
| Payroll | #38 zero budget refused, #39 collection ≤ slot book, #41 assignment batches of 400, #53 `CohortProgress`, #54 refuses to assess without the settler role, `epochWord` view | next payroll deploy |
| WorkRegistry | #29 predecessors closed with their tail, #31 preview mirrors the result drain, #32 `EntryReserved`, #33 dead errors removed, #36 `canWork` covers leave and crash days, #51 settle word bound to the payroll's word, #56 chained beneficiary mismatch reverts | next work deploy |
| EmploymentRegistry / ItemShop | #34 stale posting gives the day's attempt back, #35 quoted `postingVersion` on `appoint`/`hireNow`/`buyHireNow`, #37 spouse flag on the registry is the truth, #52 `Pausable`, `TitleNotInPosting` | next employment/shop deploy |
| AdminCouncil | #46 `NoCode`, #51 `setEnrollOpensBefore` guarded (scheduled on v2) | guard yes; `NoCode` next council deploy |
| All Ownable2Step contracts | #40 `renounceOwnership` disabled | next deploys |
| Scripts / legacy | #45 retired consumers revoked (scheduled), #47 faucet script on the M1 registry with a testnet guard, #48 legacy Hybrid stack deleted, #49 packs moved to the council + `PACK_DENOMINATION` default 500 + chain guard, #50 `DeployEntropySource.s.sol` + `MigrateSource.s.sol` | yes (on-chain parts) |
| Gacha | #44 gate accepts the shift beneficiary, #58 documented as a credential, #59 30% default + docs, #60 documented design, #61 submodule bumped | gate next deploy; rest yes |
| Docs | #42 processCohort cap, #55 calendar horizon below | yes |

**Calendar (#55).** `MarketCalendarLib` hard-codes the post-2007 US DST rule and `MarketCalendar.supportedTo` bounds the horizon; the registries hold the calendar as an immutable. A rule change or the horizon needs a new calendar and a registry redeploy; plan it a year ahead of `supportedTo`.

**Payroll dependencies (#54).** A day can only be assessed while Payroll is a settler on the registry, the stock token is neither paused nor blacklisting the payroll, and the randomness source is funded. Recovery: re-grant the settler role (scheduled on the council), or `rerequestSeed` after the deadline; a paused stock token simply delays claims.

**Retired-source note (#30).** Cancelling an unstarted shift forfeits its pay and XP but keeps the projected stress and used-day marks; reversing them needs per-entry deltas and is deferred.

## v0.21 delta (2026-09-13)

What changed in the spec and where the code stands. Everything below is in `main` and green; nothing is deployed yet. Deploying needs two cycles: (a) a work/employment/payroll/shop redeploy ("r6"), which resets every Jerk's job, XP and strikes on testnet; (b) a vault redeploy for the fees, which re-mints the collection and resets every character. Both are player-visible resets and wait for Scott's call.

| Spec change | Code | Status |
|---|---|---|
| Mint 1 USDC, Burn 1 USDC, receipts, `redeemNFTs` quote | `core/HybridVault` (`unwrapFee`, `previewRedeem`, `FeeEarned`), `script/DeployM1.s.sol` defaults 1e18/1e18 | built, not deployed |
| One-action work; hiring inside the shift; `putToWork` | `work/WorkRegistry.enrollShift` + `work/EmploymentRegistry.onShiftAccepted/resolveHiring/abandonHiring` | built, not deployed |
| Employment status at birth (UNEMPLOYED / EMPLOYED / SELF_EMPLOYED) | `employmentStatus` view; executives auto-bound to `defaultJob` on first work | built |
| No Work Found outcome | `Entry.noWorkFound`, `NoWorkFound` event, `Award.noWorkFound`, `previewClaim` reason | built |
| Lifetime XP; XP-adjusted hire and strike odds, frozen at acceptance | `CareerLib.experienceBonusBps/effectiveHireBps/effectiveStrikeBps`; `_applyResults` | built |
| Burnout blocks job-search shifts | already the case: the search is a shift | built |
| Executives keep their standing appointment (no resign) | `resign` reverts `TitleIneligible` for executives | built |
| JobBoard → Work view, labels (Mint, Burn, Put your Jerk to work, Searching, No Work Found) | jerkz-ui | not started (ABI changes land with the r6 deploy) |
| `HiringAttemptCommitted` / `HiringResolved` events | built | — |
| Optional hire inside `putToWork`, hourly grid | deferred (two transactions; daily shifts) | — |
| Hiring randomness no longer a separate draw | `EmploymentRegistry` is no longer a randomness consumer; Entropy budget scripts drop it | built |
