# Milestone 1b: Jobs, Shifts, Health, Career

Status: implementing. 2026-09-09. Derived from Work2Earn v0.15 §3, §4, §5 and §11. Builds on M1a's `JerkzNFT` and `MarketCalendar`.

## 1. Modules

| Contract | Owns |
|---|---|
| `EmploymentRegistry` | Job postings, applications and their randomness, employment state, strikes, XP, career streak and promotions, resignations, firing. Writes `currentTitle` on the NFT through the `EMPLOYMENT` role. |
| `WorkRegistry` | Shift custody of the NFT (the spec's QuestEscrow), the permanent token-id/day ledger, health (stress, burnout, overtime leave, weekend passes, Monday crash), entry snapshots and settlement of performance and discipline draws. Bumps metadata through the `WORK` role. |

Payroll (M2) is the only caller of `WorkRegistry.settleEntry`; it passes the epoch's verified word and the registry derives the two draws. Until M2 exists, a role-holder can settle in tests and on testnet. Shop (M3) gets the `SHOP` role on both for JUICE, passes, spouse and hire-now.

## 2. Decisions that turn prose into rules

- **Per-request randomness for applications**, one request per application at `readyAt`, matching the wrap. The spec's 60-second intake is kept as batch-size one.
- **Funding gate is pluggable.** `WorkRegistry.funding` is an `IEpochFunding`; when unset (testnet before M2) every day counts as funded.
- **Executives are employed by birth.** C-suite, CEO and Board are employed unless they resign; `appoint(jobId)` can bind a specific posting. Founder is self-employed on the venture track. VC never works.
- **Health is settled lazily.** Idle recovery is `20 × complete off days` since the last accounted day, computed at the next action, never looped.
- **Character key** is `(tokenId, generation)`. Every mutating call takes `expectedGeneration` and reverts `StaleGeneration` on mismatch. Retirement is detected by comparing against `nft.generationOf`.

## 3. Numbers (proposed v1, frozen per generation at birth via `progressionVersion`)

| Tier | Warm-up | Hire chance | Pay factor ×100 | Strike on mistake | Promotion threshold |
|---|---|---|---|---|---|
| Intern | 8h | 60% | 100 | 100% | 10 |
| Junior | 4h | 75% | 110 | 80% | 15 |
| Specialist | 2h | 85% | 125 | 60% | 20 |
| Manager | 1h | 95% | 150 | 40% | 25 |
| Director | 30m | 99% | 175 | 20% | 30 |
| C-suite | instant | 100% | 200 | 0, immune | 40 |
| CEO | instant | 100% | 250 | 0, immune | 60 |
| Board | instant | 100% | 300 | 0, immune | ceiling |
| VC | no work | — | — | — | — |
| Founder | self-employed | — | 100 | no employer | no ladder |

Shifts: 8h weight 100, 16h 175, 24h 200, 48h 200+200. Stress: 8h −5, 16h +15, full day +20, weekend +10, off day −20, spouse ×0.8 on positive changes. Mistake chance by stress band 0–29 / 30–59 / 60–79 / 80–100 = 5 / 15 / 30 / 50%. Burnout latches at 80 and clears below 60. Two adjacent overtime days block the next day. Two strikes fire Intern–Director; three clean entries clear a strike. Reapply on `max(notificationDay, lastWorkDay) + 2`.

## 4. `EmploymentRegistry`

> **Superseded for hiring by spec v0.21 §3.2 (2026-09-13).** There is no player application flow any more.
> A shift accepted by an unemployed Intern–Director carries one hiring attempt (`onShiftAccepted` →
> `HiringAttempt` keyed by shift id, odds frozen from title and XP); the work registry resolves it with the
> first entry's cohort seed (`resolveHiring`, `JOB_APPLICATION` domain) before that entry's performance
> draw. A failed search settles every entry of the shift as **No Work Found** (zero weight, no discipline,
> one XP). Executives are EMPLOYED from birth with a standing appointment (`defaultJob`), Founder is
> SELF_EMPLOYED, VC never works. `applyForJob`, `requestApplicationResult`, `cancelApplication` and
> `appoint` are gone; `hireNow` (shop) remains the paid bypass. XP is lifetime (every completed entry);
> career credit only while employed. The state and hiring paragraphs below describe the v0.19 design.

**Postings**: `createPosting(employer, tierMask, category) → jobId`, `setPostingActive`. `version` bumps on edit.

**State per character**

```solidity
struct Employment {
    uint32 jobId; uint32 postingVersion;
    bool executiveResigned;           // executives are employed unless they resign
    uint32 careerStreak; uint32 promotionProgress; uint32 xp;
    uint8 strikes; uint8 cleanStreak; uint16 timesFired;
    uint32 reapplyDay; uint32 lastApplicationDay; uint64 pendingApplication;
    uint32 lastCommittedWorkDay; uint8 progressionVersion;
}
```

**Hiring (Intern–Director)**: `cancelApplication(id)` lets the applicant withdraw during warm-up, and lets anyone clear a requested draw the source never fulfilled after `applicationDeadline`. `applyForJob(tokenId, gen, jobId, postingVersion, deadline)`: unemployed, no pending application, one per generation per Payroll day, `today ≥ reapplyDay`, not blocked by health (checked in WorkRegistry view), posting active and tier-eligible. Freezes odds and `readyAt = now + warmup(tier)`. `requestApplicationResult(appId)`: anyone, `now ≥ readyAt`, one randomness request. `fulfillRandomness`: `hired = sample(word) < hireChanceBps`; hired binds the job; failure allows a new application next day.

**Executives**: `appoint(tokenId, gen, jobId)` binds a compatible posting with no roll; also clears `executiveResigned`.

**Resign**: no active or queued shift, no pending results (WorkRegistry views). Below C-suite sets `reapplyDay`.

**Results**: `onEntrySettled(tokenId, gen, day, mistake, strike, completed)` from the `WORK` writer appends to a per-character queue. `applyEmploymentResults(tokenId, gen, max)`: in order: mistake resets cleanStreak else cleanStreak++ (3 → strikes−1, reset); strike → strikes++ (2 → fired: clear job, strikes, cleanStreak, timesFired++, reapplyDay, streak and progress reset; XP kept). If still employed after the entry: xp++, careerStreak++, promotionProgress++. Results are kept in day order. Strikes are trusted as frozen at settlement (an entry accepted under a lower title keeps its odds after a promotion). Promotion runs only when the queue is empty **and** the character has no open entries: if employed and `promotionProgress ≥ threshold[currentTitle]` and title < Board, promote one rung, subtract threshold, `nft.setCurrentTitle`. Founder earns xp only. Executives never gain strikes because their frozen strike odds are zero.

**Views**: `isEmployed`, `canEnroll(tokenId, gen)`, `pendingResultCount`, `employment`, `application`.

## 5. `WorkRegistry`

**Shift**

```solidity
enum Kind { H8, H16, H24, H48 }
struct Shift {
    uint256 tokenId; uint32 generation; address depositor; address beneficiary;
    uint32 startDay; Kind kind; uint64 startTs; uint64 endTs;
    ShiftStatus status;                // Enrolled, Matured (withdrawn or re-enrolled), Cancelled
    uint256[] entryIds;
}
struct Entry {
    uint256 shiftId; uint256 tokenId; uint32 generation; uint32 day;
    uint8 title; uint16 weight; uint8 stressAtStart; uint16 riskBps; uint16 strikeBps; bool spouse; bool weekend; bool overtime;
    EntryStatus status;                // Enrolled, Settled, Forfeited
    bool mistake; bool strike; uint16 effectiveWeight;
}
```

**Health per character**: `stress`, `lastAccountedDay`, `burnout`, `overtimeStreak`, `lastOvertimeDay`, `leaveDay`, `crashDay`, `crashCleared`, weekend pass by week id, `spouse` flag (set by SHOP), `pendingReset` (JUICE, M3).

**Enroll** `enrollShift(tokenId, gen, startDay, kind, beneficiary)`:
1. Window: `dayStart(startDay) − 1h ≤ now < dayStart(startDay)`. Caller **owns** the NFT (operators cannot commit someone else's Jerk) or is the fixed beneficiary of a shift still in escrow (successor, `endTs ≤ dayStart(startDay)`, beneficiary carried over).
2. Character ACTIVE and generation matches; `employment.canEnroll`; VC rejected; backlog: `pendingResultCount + entries ≤ 2`.
3. Every entry day: not used for this token id (permanent), funded, not `leaveDay`, not an uncleared `crashDay`, weekend needs the week's pass, no overlap with an existing shift.
4. Health projection: settle idle recovery to `startDay`; apply each segment's stress change in order and freeze the entry's stress and mistake risk from the **resulting** stress (spec §5.1); a segment reaching 80 latches burnout and blocks any later segment; overtime streak projection: if `lastOvertimeDay == startDay−1` and segment one is overtime, `startDay+1` becomes leave, which rejects a 48h.
5. Take custody (`transferFrom`), write shift and entries with snapshots (title, weight, stress, risk, strike odds, spouse, weekend, overtime), mark days used, schedule Monday crash on first weekend reservation of the week, update overtime streak and `leaveDay`.

**Mature and withdraw**: `withdrawNFT(shiftId, recipient)`: beneficiary, `now ≥ endTs`, no queued successor. Returns the NFT. Never depends on randomness, keepers or payroll.

**Settle** `settleEntry(entryId, word)`: `EPOCH` role, entry Enrolled, `now ≥ dayEnd(day)`. `perf = sample(word, PERFORMANCE, key)`, `disc = sample(word, DISCIPLINE, key)`; `mistake = perf < riskBps`; `strike = mistake && title < CSuite && disc < strikeBps`; `effectiveWeight = mistake ? weight/2 : weight`; emits `WorkAssessed`; calls `employment.onEntrySettled`.

**Emergency**: `cancelShift(shiftId, recipient)` owner-only while paused: started entries stay and settle later, future ones are Forfeited, custody goes to `recipient` (never the registry or the NFT), health and the permanent day ledger are kept. `forfeitStaleEntry(entryId)` (owner, 7 days after the day end) releases an entry the settler never settled so the character is not frozen. `rescue(tokenId, to)` returns a token that reached the registry outside a shift. `beneficiary` may not be the registry or the NFT contract.

**SHOP role** (M3 hooks, present now): `setSpouse`, `grantWeekendPass`, `clearCooldowns` (stress 0, burnout off, overtime streak 0, leave and crash cleared, reapply cleared through employment).

## 6. Randomness domains

`JOB_APPLICATION`, `WORK_PERFORMANCE`, `WORK_DISCIPLINE`, each keyed with `(tokenId, generation, day or applicationId)` so one epoch word settles every entry independently.

## 7. Tests

Hiring odds by tier, one application per day, executives auto-employed and resign/reclaim, Founder path, VC rejection; shift windows and all four kinds across a DST day; permanent day usage; successor chaining; weekend pass and Monday crash; overtime leave including the 48h rejection; stress bands and burnout latch and recovery arithmetic; spouse rounding; settlement draws, half weight, strikes, firing at two, clean-streak clearing; promotion at exact threshold with surplus, ceiling at Board, no Founder/VC ladder; reapply delay; backlog cap; retirement isolates old results.

## 8. Out of scope here

Payroll money (M2) — entries carry the weights and labels it needs. JUICE purchase, items, spouse purchase, hire-now items, referrals (M3) — the hooks exist. Metadata JSON for career fields (M4).
