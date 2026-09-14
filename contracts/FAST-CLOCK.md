# Fast clock for testnet: minutes instead of hours

Status: BUILT and DEPLOYED to Arc testnet on 2026-09-14 as "r6" with the fast-180 profile (480 s days):
FastCalendar `0x1E41e9E3341c86Af5E963c07974e39b3a34AB32F`, WorkRegistry v4 `0xA734…55FD`, EmploymentRegistry
v2 `0x95Fe…9B5C`, Payroll v3 `0xF313…7281`, ItemShop v3 `0xa5b7…9295`; 360 tests (127 of them the work suites
on the fast calendar). Originally written as the plan; the design below is what shipped. Scott: "cut work shifts down to 8 minutes instead of hours
and any other intervals down to minutes from hours." Matches the **"Faster test mode"** row of
[CHANGELOG.md](../CHANGELOG.md) (2026-09-14, "Approved direction after v0.21; PRD amendment still
needed"): a separate 60× game-clock profile where 8/16/24/48-hour shifts become 8/16/24/48 minutes,
one clock applied consistently to starts, cooldowns, weekends and Payroll scheduling, production
timing unchanged, real network/randomness waits disclosed. Section 6 maps that row's three open
details (clock anchoring, deployment profile, boundary tests) to this plan.

## 1. What the clock is today

Every game interval hangs off one thing: the **Payroll day**. `MarketCalendarLib` defines a day as
09:30 → 09:30 America/New_York (86,400 s, DST-aware) and numbers days by Unix day ordinal. Nothing
else defines time:

| keyed on the calendar day (scales for free) | independent seconds (must be changed) |
|---|---|
| shifts (an entry is one day; H24 = 1 day, H48 = 2 days) | `CareerLib.shiftSeconds`: H8 = `8 hours`, H16 = `16 hours` literals |
| Payroll epochs, seal after `dayEnd`, `maxPublishAhead` (days) | `WorkRegistry.enrollOpensBefore` (setter, 1 h default, 7 d on testnet) |
| weekend passes (`weekId = (d+3)/7`), `isWeekend` | `WorkRegistry.STALE_GRACE = 7 days` (constant) |
| re-apply cooldown (`REAPPLY_DELAY_DAYS = 2`) | `Payroll.seedDeadline` (setter) with floor `MIN_SEED_DEADLINE = 1 hours` (constant) |
| off-day stress recovery, Monday crash, leave days | `AdminCouncil.delay` (3600 s testnet; self-call `setDelay`) |
| | `HybridVault.refundDeadline` (irrelevant under drand: 0) |
| | `RarityConfig.MIN_DELAY = 1 days` (catalog switches only) |
| | `DrandSource` margins 15/90 s and `PUBLISH_SLACK` 90 s (external beacon, cannot shrink below ~90 s) |
| | `TestnetFaucet.cooldown` 60 s (fine) |

The keeper and the UI each re-implement the day: `day * 86_400_000` for labels, `(day+4)%7` for
weekday, `America/New_York` for the 20:00 Sunday publish, and prose that says "8, 16, 24 or 48
hours" and "09:30 to 09:30 New York".

The calendar is an **immutable** in WorkRegistry, EmploymentRegistry, Payroll and ItemShop. It is
typed `MarketCalendar` but only called through its ABI, so any contract with the same functions can
be passed at deploy time. The vault, NFT, rarity, token, council, drand source, gacha machine and
faucet never touch the calendar.

## 2. Design: one scale factor, one new contract

**The day length is a deploy parameter; two testnet profiles are defined.** Every relative rule in
the spec is a Payroll day or a fraction of one, so once the calendar shrinks the whole game keeps its
ratios and testnet play exercises the same logic mainnet will run. Mainnet never gets a parameter:
production deploys `MarketCalendar` (86,400 s New York days) and nothing else.

| profile | `DAY_SECONDS` | H8 / H16 / H24 / H48 shift | week | re-apply (2 d) | stale grace (7 d) | who asked |
|---|---|---|---|---|---|---|
| **fast-60** | 1,440 (24 min) | 8 / 16 / 24 / 48 min | 2 h 48 min | 48 min | 2 h 48 min | changelog's "60× game-clock profile" |
| **fast-180** | 480 (8 min) | 2:40 / 5:20 / 8 / 16 min | 56 min | 16 min | 56 min | Scott: "we want 8 minute day as an option for testnet" |

Both are testnet only. fast-60 is the one the team's changelog describes and the one the PRD
amendment in §7 should name; fast-180 is a faster iteration setting for engineering and QA sessions
where a full week of game days should fit in an hour. Switching profiles is a redeploy of the four
calendar-holding contracts (the calendar is immutable in them), so pick one per testnet reset.

What does not shrink with the day (see §2.3): drand needs ~90 s after a day ends before the payroll
seed exists, so settlement is ~6% of a fast-60 day and ~19% of a fast-180 day; the council timelock
and block finality are real time. Both profiles remain workable; fast-180 just makes those waits
visible, which the changelog asks the app to disclose anyway.

### 2.1 `src/testnet/FastCalendar.sol` (new, testnet only)

Same external surface as `MarketCalendar` (`version, supportedFrom, supportedTo, dayStart, dayEnd,
dayIdAt, currentDay, isWeekend, weekday, isDst, nextFriday, civil, ordinal`), backed by
`genesis` and `daySeconds` immutables:

- `dayStart(d) = genesis + (d - firstDay) * daySeconds`, `dayEnd(d) = dayStart(d + 1)`,
  `dayIdAt(ts) = firstDay + (ts - genesis) / daySeconds`.
- `firstDay` is chosen so `weekday(firstDay) = (firstDay + 4) % 7` lands on a Monday at genesis,
  and `firstDay ≥ 20454` so ids stay in the same range the UI and keeper expect.
- `isWeekend`, `weekday`, `nextFriday` keep the `(d+4)%7` arithmetic (a "week" is 7 fast days).
- `isDst` returns false; `civil`/`ordinal` map ids to synthetic dates (day 1 of the run = 2026-01-01)
  so any caller that formats a date still gets one.
- Range-checked like the real one (`supportedFrom..supportedTo` = a year of fast days: 21,900).

`MarketCalendarLib` and `MarketCalendar` are untouched; mainnet keeps them.

### 2.2 Production code changes (small, reviewable, mainnet-neutral)

1. **`CareerLib.shiftSeconds` → fraction of the day.** H8 = day/3, H16 = 2·day/3, where day =
   `calendar.dayEnd(startDay) - calendar.dayStart(startDay)` read in `WorkRegistry._plan`. On the NY
   calendar this is exactly 8 h / 16 h (DST switch days get 7h40 / 8h20, which is arguably more
   correct than today's fixed 8 h). Signature change: `shiftSeconds(kind, daySeconds)`.
2. **`WorkRegistry.STALE_GRACE`** → `7` days expressed through the calendar:
   `staleAt = calendar.dayEnd(e.day + 7)`. Same behaviour on mainnet.
3. **`Payroll.MIN_SEED_DEADLINE`** → constructor parameter `minSeedDeadline` (mainnet deploys pass
   1 hour; testnet passes 2 minutes). The floor exists so the owner cannot re-roll a seed request
   (review 2026-09-12 #3); a parameter set once at deploy preserves that.
4. Nothing else in `src/` changes. `enrollOpensBefore`, `seedDeadline`, `maxPublishAhead`,
   council delay and drand margins are already runtime settings.

### 2.3 Testnet settings under the fast clock

| setting | value | why |
|---|---|---|
| `daySeconds` | 1,440 s (fast-60) or 480 s (fast-180) | per-reset choice, testnet only |
| `enrollOpensBefore` | 7 fast days (2 h 48 min) | today's testnet rule "book any published day" |
| `maxPublishAhead` | 7 | one fast week ahead, as now |
| `seedDeadline` | 5 min (floor 2 min) | a seed that never lands re-requests in minutes |
| council `delay` | 5 min (via the delayed `setDelay` self-call; no redeploy) | guarded admin calls testable in a sitting |
| drand payroll margin | 90 s (unchanged) | beacon floor; a day ends, seal, seed lands ~2 min later, well inside the next 24-min day |
| drand vault margin | 15 s (unchanged) | |
| `RarityConfig.MIN_DELAY` | leave | only catalog switches |
| faucet cooldown | 60 s (unchanged) | |

Under fast-180 the spec's 1-hour enrol window would be 20 s, so testnet keeps "book any published
day" (`enrollOpensBefore` = 7 fast days) under either profile.

### 2.4 Keeper (`jerkz-keeper`)

- `FAST_CLOCK=1` (or detect: `calendar.dayEnd(d) - calendar.dayStart(d) != 86400`).
- Day labels from the calendar, not from `day * 86_400_000`: "day 20981 (Tue) 14:12 → 14:36".
- Publish rule: drop "Sunday 20:00 New York". Keep `maxPublishAhead` days published at all times,
  splitting the free reserve evenly over the unpublished days in the horizon (the daily safety net
  becomes the only rule). The weekly rule returns on mainnet behind the same flag.
- Ticks: payroll duty 5 min → 20 s; settle walk-back window in days stays (days are just shorter);
  drand duty already 15 s.
- `/status`, `/payroll` copy in minutes.

### 2.5 UI (`jerkz-ui`, `jerkz-gacha/ui`)

- Read day boundaries from the calendar contract (`dayStart`/`dayEnd`) everywhere `useWork.ts`
  currently multiplies by 86,400,000; label days as "Day N · Tue 14:12–14:36" under the fast clock
  and as dates on the NY calendar.
- Shift chips: "8h/16h/24h/48h" → "⅓ day / ⅔ day / 1 day / 2 days" with the minute count next to
  it, driven by `dayEnd - dayStart`.
- `fmtDur`, quote deadlines (900 s stays), `useUpcomingDays` refetch 30 s → 5 s.
- Docs page: the hour-based prose becomes clock-aware ("Payroll days are 24 minutes on testnet").
- Fix the two stale reads flagged in the inventory while there: `Work.tsx` "Warm-up ends in" and
  `Admin.tsx` "application deadline" reference v0.20 fields that no longer exist.

### 2.6 Tests

- `test/testnet/FastCalendar.t.sol`: boundaries, weekday alignment, range check, `dayIdAt` inverse.
- `WorkFixture` gains a `daySeconds` knob and the whole work/employment/payroll/shop suite runs
  twice (NY calendar and a 1,440 s fast calendar) via a small abstract-fixture switch.
- `MarketCalendar.t.sol` and the NY fixture stay exactly as they are.

## 3. Deployment plan (testnet, "r6")

This is the same four-contract cycle the pending spec v0.21 deployment already needs (WorkRegistry,
EmploymentRegistry, Payroll, ItemShop are all v0.21-changed and all hold the calendar), so do both in
one reset rather than two.

1. Contracts: FastCalendar + the three production changes + `DeployR6.s.sol` reading
   `DAY_SECONDS`, `ENROLL_OPENS_BEFORE`, `PAYROLL_AHEAD_DAYS`, `SEED_DEADLINE`, `MIN_SEED_DEADLINE`;
   deploys FastCalendar, WorkRegistry, EmploymentRegistry, Payroll, ItemShop; wires writers/settlers;
   hands ownership to the council (accept via `exec`); registers the new Payroll on the DrandSource
   (`setConsumer`, timed, 90 s) and revokes the old one; redeploys the gacha `ExecutiveTitleGate`
   (it holds the WorkRegistry address) and points the machine at it. Vault, NFT, rarity, token,
   council, drand, gacha machine, packs, faucet: unchanged.
2. Fund the new Payroll (move the old one's free reserve through the council, as the keeper's legacy
   drain already does) and publish the first fast week.
3. Council `setDelay` 5 min (schedule → wait the current 1 h → execute).
4. Keeper: copy registries, `FAST_CLOCK=1`, rsync, rebuild. Arb bot: unaffected (no calendar).
5. UI: config flag + labels, Vercel deploy. Gacha UI: gate address only.
6. Player-visible reset: shifts, employment, careers and unclaimed payroll on the old contracts end;
   Jerks, JERKZ, packs and gacha prizes carry over. Announce before step 1.

Rollback: the old contracts stay deployed; pointing the keeper and UI back at the previous registry
files restores the hour clock.

## 4. Effort

About a day: FastCalendar + production tweaks + tests (half a day), keeper and UI clock-awareness
(half a day), then the r6 deployment (an hour, mostly council waits). Doing it together with the
v0.21 rollout avoids a second reset.

## 5. Open choices for Scott

- Which profile for the next testnet reset: fast-60 (24-min day) or fast-180 (8-min day). Both stay
  supported; a later reset can switch.
- Keep `enrollOpensBefore` at a full fast week (book any published day, today's testnet rule) or use
  the spec's 1 hour scaled (1 min / 20 s), which is too tight for humans.
- Whether to ship this together with spec v0.21 (recommended) or as a separate reset first.
- Whether AKLO adopts the §7 text as the PRD amendment the changelog says is still needed.

## 6. Alignment with the 2026-09-14 changelog

| changelog requirement | where this plan meets it |
|---|---|
| "Separate 60× game-clock profile" | one deploy profile (`FAST_CLOCK=1`, `DAY_SECONDS=1440`) deploying `FastCalendar`; mainnet profile deploys `MarketCalendar` unchanged (§2.1, §2.3) |
| "8/16/24/48-hour shifts become 8/16/24/48 minutes" | fast-60: shift length = day/3, 2·day/3, 1 day, 2 days of a 1,440 s day = 8/16/24/48 min (§2.2 item 1). fast-180 (8-min day) is a testnet-only extra beyond the changelog row, same ratios, three times faster; the PRD is a mainnet document and does not need to name it |
| "Apply one clock consistently to starts, cooldowns, weekends and Payroll scheduling" | every day-keyed rule reads the calendar; the three second-based rules (shift seconds, stale grace, seed-deadline floor) are re-expressed through it; the keeper's publish rule becomes calendar-driven (§2.2, §2.4) |
| "Keep production timing unchanged" | `MarketCalendarLib`/`MarketCalendar` and the NY fixture untouched; the production tweaks are identities on an 86,400 s day (§2.2) |
| "Disclose real network/randomness waits" | drand margins (15 s vault, 90 s payroll/gacha) and `PUBLISH_SLACK` do not scale; the UI and `/status` must say "settles about 2 minutes after the day ends (drand)" rather than implying instant settlement; council delay 5 min is a real wait too (§2.3, §2.5) |
| open: **clock anchoring** | `FastCalendar.genesis` = deploy timestamp rounded down to the minute; `firstDay` chosen so that day is a Monday and ids stay ≥ 20454; a "week" is 7 fast days from that anchor; no DST (fast days are always 24 fast-hours, so the hourly grid a later §4.1 implementation needs is a clean 60 s) (§2.1) |
| open: **deployment profile** | `DeployR6.s.sol` env: `FAST_CLOCK`, `DAY_SECONDS`, `ENROLL_OPENS_BEFORE`, `PAYROLL_AHEAD_DAYS`, `SEED_DEADLINE`, `MIN_SEED_DEADLINE`; registry gains `calendarKind: "fast" | "ny"` so the keeper and UI switch labels from it, not from a guess (§3) |
| open: **boundary tests** | `FastCalendar.t.sol` (day edges, `dayIdAt` inverse, weekday anchor, range) plus the whole work/employment/payroll/shop suite run on both calendars through one fixture knob; NY DST-edge tests stay as they are (§2.6) |
| "Identify all state resets and obtain deployment approval" | §3 step 6 lists the reset; nothing in this document authorises a deployment. The r6 cycle waits for the changelog's acceptance items 1–4 and an explicit go from Scott |
| "exact `putToWork` interface remains incomplete" | the same r6 cycle is the moment to expose `putToWork` (alias or rename of `enrollShift`) so the fast-clock reset and the v0.21 interface land in one reset |

Not covered here, and not changed by this plan: functional suits (changelog, undecided boost), hourly
:30 starts and four-hour funding cohorts (existing gap; the fast calendar keeps a 24-slot "hour" grid
so that work maps onto it later).

## 7. Proposed PRD amendment text (for AKLO, §4.1 or a new §4.6)

> **Test-mode clock profile.** Test deployments may run a 60× clock: the Payroll day is 1,440 s
> instead of 86,400 s, anchored at a deployment-time Monday boundary, with 24 one-minute start
> slots and no daylight-saving adjustment. Shift lengths, cooldowns, weekend passes, re-apply delays,
> stale-entry grace and Payroll publishing horizons are all defined in Payroll days or fractions of
> one, so they scale with the day. Randomness (drand rounds and margins), admin timelocks and network
> finality are real-time and are disclosed as such in the app. Production deployments use the New
> York calendar only; no test-mode parameter exists on production contracts.
