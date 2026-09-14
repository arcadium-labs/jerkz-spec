# JERKZ spec changelog

## 2026-09-14 | Next implementation handoff

The main change is one action: **Put your Jerk to work.** Hiring happens inside the shift. Experience improves future hiring and job retention. Mint and Burn each cost 1 USDC.

This is the “v2” implementation handoff, not a new protocol release or the property expansion in PRD §14. The approved PRD remains **v0.21**. This file records changes; [README.md](README.md) defines the system. [Contract conformance](contracts/CONFORMANCE.md) tracks implementation and deployment.

**Comparison:** initial v1 reviewed on 2026-09-12, contract revision `cf6d62534640d67e5f0a97120ff5d9f513ee19f5`, against PRD v0.19 → v0.21. Current code was inspected at `5b5ea852205c67bf2c887e7dd78f4579658dd27b`; deployment status below comes from the 2026-09-13 conformance record at spec-repo commit `f0e5942`. It is a dated handoff, not a new live-chain audit.

### Changes already approved in v0.20–v0.21

| Area | Initial v1 | Required behavior | Reference |
|---|---|---|---|
| Bridge fees | 2 USDC Mint; free Burn. | **1 USDC Mint + 1 USDC Burn**, per collectible. Each reroll cycle still costs 2 USDC, excluding gas. Return the full 100,000 JERKZ principal on Burn; charge USDC separately. Only completed conversions earn fees for Payroll. | §2, §6.1 |
| Work flow | Apply → wait through a title-based warm-up → resolve hiring → enroll in work. | **Put your Jerk to work** accepts eligible unemployed or employed workers into a funded shift. Select a compatible job automatically. No separate application timer or required hiring transaction. | §3.2 |
| Initial employment | Lower tiers apply; executives have an appointment action. | Intern–Director start **UNEMPLOYED**. C-suite/CEO/Board start **EMPLOYED** with standing appointments. VC is employed/passive; Founder is **SELF_EMPLOYED**. | §3.1–3.2 |
| Hiring result | Separate application result before paid work. | One frozen hiring roll per accepted unemployed shift, using the first entry's cohort seed in its own domain. A 48-hour shift shares that result across both entries. Success enters Payroll; failure is **No Work Found**, with zero award/weight/strikes but the full lock, workload and scheduled XP. No retry or item can replace the accepted result. | §3.2, §7 |
| Experience | Work progression exists; hire/strike odds depend on title. | Every completed funded entry earns 1 lifetime XP, including failed searches, No Pay, Founder flops and firing/notice entries. 48-hour shifts earn 2. XP improves later hiring and strike odds; freeze both at acceptance. | §3.3 |
| Employment continuity | Separate application/appointment controls. | Keep a job between shifts. Firing/resignation below C-suite returns the worker to unemployment after the existing reapply delay. Protected executives cannot resign. An unresolved search cannot queue a successor. | §3.2, §4.3 |
| Hiring shortcuts | Separate shop-assisted appointment. | Degree, purchased referral, bribe or eligible sponsor remain optional shortcuts before acceptance. They cannot change a pending search or bypass funding, health, custody or daily-use limits. Optional purchase + hire + work must be atomic if offered as one transaction. | §9.2–9.3 |
| App and interfaces | Separate job board; conversion/application labels and old Burn ABI. | Use **Mint / Burn**, **Put your Jerk to work**, **Searching**, **No Work Found**. Show hiring and XP-adjusted odds in quotes. Update payable Burn, fee bounds, events and claim reasons. Warn that Burn permanently retires the generation's XP and character state. | §10.4, §11 |

**XP formula:** `bonusBps = min(2000, floor(lifetimeXP / 5) × 100)`; `hireBps = min(9900, baseHireBps + bonusBps)`; `strikeBps = floor(baseStrikeBps × (10000 − bonusBps) / 10000)`. Stress still determines mistake chance. Firing resets career streak/promotion progress, not lifetime XP or earned title. Burn retires them all. This does not replace the existing promotion ladder.

### What Azoth has already built

- The September 13 code includes the fee split, in-shift hiring, employment states, No Work Found, lifetime-XP coverage and XP-adjusted odds. The conformance record marks these **built, not deployed**. Do not rebuild them from scratch.
- Work uses `enrollShift` as the current implementation of the spec's work action. The optional atomic hire purchase and exact `putToWork` interface remain incomplete. The reskinned staging UI exists but still uses the older deployed ABI; new contract integration is pending.
- The recorded work/employment/payroll/shop deployment would reset testnet jobs, XP and strikes. The separate vault cycle would replace the collection and reset characters. These are migration effects, not approval to reset or deploy.
- drand is reported live since September 13. The old operator commit/reveal and planned Pyth migration are no longer the current randomness plan. [DRAND-SPEC.md](contracts/DRAND-SPEC.md) records the trust boundary: selected-mode Mint rounds still depend on the operator's choice. Reconcile this with PRD §2.2 before claiming production conformance.

### Approved direction after v0.21; PRD amendment still needed

| Change | Engineering requirement | Open detail |
|---|---|---|
| Faster test mode | Separate **60× game-clock profile**: 8/16/24/48-hour shifts become 8/16/24/48 minutes. Apply one clock consistently to starts, cooldowns, weekends and Payroll scheduling. Keep production timing unchanged and disclose real network/randomness waits. | Clock anchoring, deployment profile and boundary tests. |
| Functional suits | An equipped suit must improve hiring odds. It is **not purely cosmetic**. Amend §9's cosmetic-only restriction; preserve accepted-shift snapshots and permanent token burns. | Exact boost, eligible suit variants and stacking rule. +5 and +10 percentage points were studied; neither is selected. |

### Proposals still undecided

- **Small payment for every completed shift:** an attendance/“Bus Fare” allocation was simulated at 10% and 20% of the existing worker budget. No share is approved. v0.21 still permits zero pay; do not promise a minimum or add unfunded liabilities.
- **One Career XP value:** starting XP from birth traits plus earned XP, replacing separate streak/promotion counters, was proposed. It is not approved. Keep the current §3.3 model until specified otherwise.
- **Progression balance:** promotion speed needs review alongside payout simulations. No new thresholds or guaranteed earnings are approved.

### Existing gaps, not new v2 features

These requirements already predate v0.20. Keep them on the implementation backlog:

- Hourly `:30` work starts, four-hour funding cohorts and exact next-start quotes; the demo uses daily 09:30 New York starts. Preserve 8/16/24/48 shifts, DST rules and claims after the full original shift ends.
- Actual CRCL fees/conversion, protected reserves and automatic budget pacing; the demo uses manually funded cirBTC. VC accrual/checkpoints remain incomplete.
- Approximately 100,000 pre-generated IPFS options, committed branch counts/base identities, dynamic state and ERC-4906 updates. Title/expression weights are independent percentages, never fixed tier supplies. The placeholder catalog is not final delivery.
- Remaining wardrobe/referral systems, indexer, Circle integrations and production admin/security controls. The separate Gacha is not a JERKZ launch dependency.
- Resolve recorded deviations in conversion batching, callback delivery, emergency/forfeiture handling and promotion-to-job mapping. A conformance entry saying “built” is not approval of a documented deviation.

### Acceptance before deployment

1. Verify exact Mint/Burn quotes, per-item fees, full principal return, atomic failure and no double charge on delivery retry.
2. Verify one-action hiring for every role, one roll across a 48-hour shift, failed-search XP, no retry through items/successors/Burn, and frozen odds.
3. Verify XP caps/rounding, promotion and firing order, generation retirement, and original-beneficiary claims after transfer/Burn.
4. Pin the deployed addresses and ABIs before updating the UI. Identify all state resets and obtain deployment approval. Keep unavailable features labeled as such.

Use PRD §13 for the full acceptance criteria. This changelog does not certify tests or authorize a deployment.

### Spec history

| Date | Version / commit | Change |
|---|---|---|
| 2026-09-14 | This handoff | Added deployed-v1 comparison, existing implementation status and pending decisions. PRD body unchanged. |
| 2026-09-13 | `97cae94` → `f0e5942` | Added engineering specs, conformance/review records and drand documentation. |
| 2026-09-12 | v0.21 · `dae4147` | Split bridge fees into 1 USDC Mint and 1 USDC Burn; updated accounting, quotes and acceptance rules. |
| 2026-09-12 | v0.20 · `843b339` | Combined hiring/work, added XP-adjusted odds and Mint/Burn language; updated dependent interfaces and tests. |
| 2026-09-10 | v0.19 · `8712d1c` | First published working PRD used for the initial v1 comparison. |

For later edits, prepend a dated entry with the previous/new spec version, changed behavior, affected sections and migration impact. Keep implementation status in the conformance table; distinguish approved requirements, proposals, code and deployment.
