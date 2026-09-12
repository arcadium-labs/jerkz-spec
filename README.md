# Arcadium Labs : JERKZ
Work2Earn · Product and engineering specification · v0.20

## Start here: JERKZ in plain English

JERKZ is an office-life game built around collectible NFT characters. Choose a shift and **Put your Jerk to work**. The system handles finding a job, working and Payroll. Pay comes in **CRCL, tokenized Circle stock**.

**Arcadium platform:** JERKZ is Arcadium Labs’ first game. Original characters and long-term game development drive return visits; a shared Circle-powered wallet and payment layer supports this game, the separate Gacha product and later titles. Arc is the settlement network, USDC is the payment asset, and tokenized Circle equity is the proposed CRCL reward asset. These are different roles. CRCL’s issuer and Arc availability remain launch gates; no Circle issuance, endorsement or partnership is implied. §10.6 defines the selected integration scope, and §13 defines delivery evidence.

### How a player uses it

1. **Get a Jerk.** Buy an existing NFT, or lock **100,000 JERKZ** and pay **2 USDC** to draw a new character. There are 10,000 permanent NFT IDs. Each Mint draws a Birth Title from percentage weights with no tier supply caps, independently draws one of four expressions at 25% each, then selects a compatible pre-generated IPFS option. The initial base-art/metadata catalog target is about 100,000 reusable options.
2. **Put your Jerk to work.** Choose an **8-, 16-, 24- or 48-hour shift** and confirm the next funded hourly start. This is the only work-start action. There is no separate application, job selection, hiring confirmation or enrollment step. Starts are at :30 each hour; the workday resets at 9:30 a.m. New York. The quote shows the exact lock and end times, including daylight-saving changes.
3. **The system handles hiring.** Intern through Director start **UNEMPLOYED**. Their shift includes a hiring attempt: no job means no pay; finding a job opens the same shift’s Payroll draw. C-suite, CEO and Board start **EMPLOYED** and skip the hiring roll. VCs are EMPLOYED in their passive holding role. Founders are **SELF_EMPLOYED**. Jobs persist until lost. Completed attempts earn XP; experience improves hiring odds and job retention. Optional hiring items or an executive referral guarantee a job before a shift.
4. **Claim after your shift.** Results are **No Work Found, No Pay, Minimum Wage, Bonus or Jackpot**. No Work Found is a failed hiring attempt; No Pay is a zero-result Payroll draw after gaining or keeping a job. Minimum Wage means a small variable game payout, not a legal wage or guaranteed amount. Funded pay is claimable when the full accepted shift has ended and its result has settled. There is no Friday payday, weekly wait, claim fee or early-pay discount. A late result can delay a claim but cannot extend NFT custody. Title, duration and performance affect your share of a fixed funded budget; they create no new money.
5. **Manage your Jerk.** Overtime and weekend work raise stress. Stress increases mistakes; lower ranks can collect strikes and get fired. Two consecutive overtime entries require a complete workday off after accepted work ends. Weekend work needs a pass and causes a Monday crash. **JERK JUICE clears gameplay cooldowns**, but cannot end an accepted shift early or restore a lost job.
6. **Customize or start again.** Spend JERKZ on haircuts, clothes, glasses, watches and other items. Every payment permanently burns those tokens. A spouse perk can reduce stress gains. **Burn** returns **100,000 JERKZ**, but permanently discards that character’s art, title, progress and paid items. Mint again to draw a fresh character. Selling an existing NFT preserves its character; previously earned pay stays with the original beneficiary.

### What happens in a hybrid swap

The NFT is a reusable character slot. **Mint:** deposit 100,000 JERKZ, pay 2 USDC and draw a fresh character on an available ID. **Burn:** return the character to escrow, retire its metadata and receive 100,000 JERKZ. You need not receive the same ID on a later Mint. **Mint / Burn are player labels for wrap / unwrap.** The protocol transfers existing assets; it does not mint or burn ERC-721 IDs or ERC-20 principal during conversion. Item purchases separately burn ERC-20 supply permanently. ERC-4906 tells supported marketplaces to reload the metadata; indexing controls refresh timing. §2 defines the escrow lifecycle.

### Two special roles

- **Founder:** Most venture attempts pay zero; jackpots are more likely than for employees. Founders still lock for work and manage stress, but cannot be fired.
- **VC:** Earns from a reserved share of funded Payroll while held, without a work lock. A new VC starts accruing at the next 9:30 a.m. boundary. Earnings follow actual ownership time; only funded periods pay.

**Build a career.** Your starting title is part of the NFT. Interns through CEOs can earn higher job titles by completing work without getting fired. Time off pauses the streak; firing or resignation resets progress toward the next promotion. Earned titles remain. Founder and VC are separate career tracks.

### Where the money comes from

Actual creator fees, separately verified LP fees, hybrid fees and paid NFT royalties enter one Payroll Treasury. Other received assets are converted into CRCL. A daily spending limit controls Payroll. **No funded budget means no paid work or VC accrual.** The funding plan requests **50,000 USDC for CRCL Payroll**; receipt and purchase are unconfirmed. If received, 75% of the acquired stock releases over 180 days and 25% starts a slow reserve. Without funding, the same game operates from actual fees only. More frequent bonuses change the distribution of this fixed budget, not its size.

### Terms used below

- **Generation:** One character’s life, from Mint until Burn. The NFT ID can be reused; that character cannot.
- **Epoch:** One day’s funded Payroll budget and settlement records.
- **Weight:** A relative share used to divide a budget, not a fixed salary.

**Read the specification below for exact rules.** The v1 game rules and balance settings are selected. Deployment still requires the external evidence in §13.2. Gacha and future property gameplay are separate from this MVP.

## 1. Product contract

JERKZ is Arcadium Labs’ flagship IP: a tokenized Hybrid GameFi system. Put your Jerk to Work! Scope: NFT collection, tokenomics, Hybrid Escrow, jobs, health, Payroll and item shop.

Gacha has a separate product specification and is not an MVP dependency. V2 property is in §14.

**Status: selected v1 implementation baseline.** The calendar, payout distribution, funding policy, discipline, tier parameters, progression, shop prices and administration below are the selected rules, not an option list. Changes require an explicit new version and apply only to future uncommitted work or the stated future-generation/purchase boundary. This is a specification decision, not evidence of deployed contracts, economic profitability, received funding or completed audit. No deployment, spending, sharing, purchase or grant request is performed by this document.

- **Chain / launch / pair:** Arc; O1; JERKZ paired with tokenized Circle stock, CRCL. Exact integrations require evidence (§13). Robinhood is a reference only.
- **JERKZ:** Standard ERC-20; initial and maximum supply 1,000,000,000; no later minting; every item payment reduces totalSupply permanently.
- **Collection:** 10,000 ERC-721 IDs minted once into escrow; no later NFT mint or burn. Each ID holds replaceable character generations, with 100,000 JERKZ reserved per redeemable NFT.
- **Hybrid fee:** 2 USDC per token → NFT draw; NFT → token free. Gas and randomness charges are separate. No batch discount or principal deduction.
- **NFT royalty:** ERC-2981, existing 2% default; count actual receipts only. No token transfer tax.
- **Payroll:** All payouts in CRCL; one economic treasury; no passive JERKZ-holder reflections.
- **Initial funding:** Actual confirmed funding is zero. Requested launch seed: 50,000 USDC for CRCL; use the 75% / 180-day distribution and 25% reserve rule in §6.2 only after receipt. Fees-only operation is the unfunded fallback.
- **Career:** Birth Title is weighted rarity; current Job Title can rise through work (§3.3). Both belong to the NFT generation. Founder draw weight is 2%; VC is 0.1%.
- **Character reset:** Ordinary transfers preserve character state. Unwrap retires all live character data and items; every wrap draws fresh art, Job Title and starting stats.
- **Metadata:** Restricted state updates → versioned tokenURI metadata → ERC-4906 notification. The NFT supports ERC-721, ERC-165 and ERC-4906 (0x49064906); display depends on indexing.
- **Item sinks:** Permanent JERKZ supply burns; never treasury transfers or NFT-principal spending.

“Hybrid Escrow Bridge” means same-chain NFT / token conversion on Arc. O1’s cross-chain availability does not add cross-chain custody or extra supply.

Use **Mint**, **Burn**, **Put your Jerk to work**, **Payroll**, **Payroll Treasury**, **HR**, **On Thin Ice** (one strike), **Pink Slip**, **No Work Found**, **No Pay**, **Minimum Wage**, **Bonus** and **Jackpot**. These labels must expose the exact probability, fee and restriction. Jobs, moods and earnings are game states, not real employment or guaranteed income. Mint / Burn are the only player-facing conversion labels; wrap / unwrap remain technical descriptions of transfers, not ERC-721 supply changes.

## 2. Hybrid Escrow and character life

**Architecture:** Bidirectional hybrid token wrapper on EVM: independent JERKZ ERC-20, JerkzNFT ERC-721 and HybridVault. Wrap deposits tokens and withdraws an assigned NFT; unwrap returns the NFT and releases its protected tokens. Transfers exchange existing assets at a fixed rate. No ERC-404-style automatic balance coupling: ordinary JERKZ transfers never affect NFTs.

**Bootstrap:** Mint exactly 10,000 NFT IDs into inactive vault inventory in verified batches, then permanently close NFT minting. Disable NFT burning. These IDs exist for the collection’s lifetime; generation creation is a state update, not a mint. Verify all IDs and close bootstrap before public conversion. Standard ownership, approvals and transfers apply; every ownership change clears token-specific approval.

Let r = 100,000 JERKZ; N = active redeemable NFTs, including quest and third-party custody; P = funded pending wraps, including assigned but undelivered NFTs; R = protected principal:

totalSupply = 1,000,000,000 − cumulativeBurned; no new minting  
R = r × (N + P); vaultBalance >= R; N + P <= 10,000

Donations create surplus, never claims. Each principal and inventory reservation has one beneficiary and terminal state. Payroll, items, LP operations, rescue and administration cannot spend R. Inactive or assigned vault NFTs cannot work, earn or unwrap. Third-party custody never releases principal. Item burns reduce future wrapping capacity, not existing redemption rights; 10,000 permanent IDs does not guarantee 10,000 circulating NFTs.

O1’s reference launch puts the full supply into permanent liquidity. Initial active NFTs require actual separately acquired or allocated tokens in HybridVault. Never duplicate claims on LP tokens or use launch liquidity for Payroll. [O1 token creation](https://docs.o1.exchange/launchpad/create/token-creation)

### 2.1 Conversion and lifecycle

1. **Request wrap:** Quote quantity, principal, 2-USDC fee per NFT using actual decimals, gas and randomness charges, deadline, inventory, rarity and art versions, and current title odds. Receive exact principal + fee; bind beneficiary and permitted recipient; reserve capacity. The user cannot choose a title, art result or previously returned ID.
2. **Freeze batch:** 60-second public intake; one active inventory-selection batch per pool. Freeze eligible IDs, percentage-weight tables, the pre-generated IPFS catalog and canonical request order before randomness. Stage later returns for a later batch; never silently move accepted requests between inventory versions.
3. **Assign character:** Derive independent verified TOKEN_SLOT, JOB_TITLE, EXPRESSION and ART_VARIANT draws. Sample distinct NFT IDs without replacement; draw title and expression independently with replacement from their weights, then select a compatible catalog variant. Title probabilities do not depend on current holders or remaining artwork. Increment each selected ID’s generation once; save the full assignment and starting state; set ASSIGNED. Commit its metadata and emit ERC-4906. Pre-generated art options are reusable; the same option may appear on multiple active NFTs. A fresh draw can repeat an earlier image or title. Close selection after all assignments; undelivered NFTs remain reserved independently and do not block the next selection batch.
4. **Complete wrap:** Transfer the assigned existing NFT to the permitted recipient and set ACTIVE. Convert pending principal into active NFT backing; earn the 2-USDC fee for later CRCL conversion. Ownership, lifecycle, accounting and metadata changes are atomic. Failed delivery reverts that attempt and retains the same ASSIGNED character in the vault; the beneficiary may retry or change the permitted recipient. No new seed, generation or fee.
5. **Unwrap:** Current owner or authorized operator returns an ACTIVE NFT. Checkpoint VC credits, transfer to HybridVault, clear approval, retire the character, switch to inactive metadata and emit ERC-4906. Return exactly r and stage its ID for a later inventory batch. No title quota is released because no title quotas exist. All changes succeed or revert together. No exit fee, price feed, randomness or image-service dependency. Active quest or third-party custody cannot be bypassed.

**Identity:** Permanent key = (chainId, NFT contract, tokenId). Character key adds generationId. Lifecycle: INACTIVE → ASSIGNED → ACTIVE → INACTIVE. Retirement invalidates the old generation; no path restores it. A later wrap may allocate the same ID to any requester. Ordinary sales, gifts, quest deposits and withdrawals preserve the active generation; only explicit hybrid conversion resets it.

**Birth:** Fresh art, Birth Title and expression profile; initialize current Job Title = Birth Title, careerStreak = 0 and promotionProgress = 0, with a committed progression configuration; expression sets initial stress (§5). Zero XP, strikes, firings, fatigue, leave, crash, hiring attempts, weekend passes, wardrobe and relationship. Set employmentStatus: Intern–Director UNEMPLOYED; C-suite, CEO, Board and VC EMPLOYED; Founder SELF_EMPLOYED. Executives receive compatible standing appointments; VC keeps its passive role and starts next-boundary eligibility on ACTIVE delivery. No separate Unemployable or Self-employed rarity tier exists.

**Retirement:** Remove the current-generation reference and invalidate live art, Birth Title, current Job Title, careerStreak, promotionProgress, employer, application, XP, strikes, stress, burnout, fatigue, recovery, crash, passes, wardrobe, cosmetics and spouse. Generation-keyed storage may remain historically; it must be unreachable as current state and cannot authorize a later character. Use bounded invalidation, not a loop over owned items or history.

Preserve generation and ownership counters, permanent token-ID daily Payroll use and sponsor quotas, frozen entries, funded VC intervals, original beneficiaries, reserved CRCL and paid flags. Old callbacks settle only their retired records and financial obligations. Generation-specific hiring history and career backlog reset; permanent attempt-day, financial and replay limits do not. No dual worker / VC entitlement or second daily entry after reroll.

Quotes must show the permanent loss of art, rarity, career, health and items, plus surviving CRCL claims. Unwrap may escape illness or a bad career, but rewrap has no rare-title guarantee or item refund. “Lost forever” means no live-state restoration; blockchain history, cached metadata, published images and immutable files can remain.

### 2.2 Randomness and grinding

Use an Arc-supported verified adapter. Bind chain, contract, kind, batch / epoch / worker cohort, frozen-input hash and provider request. Each draw domain also binds ID / generation and committed weight/catalog versions. ART_VARIANT selects an existing IPFS option; it never generates an image. Reject duplicate or foreign callbacks. Callback stores seed only; bounded settlement and transfers run separately. Operator, refresh and delivery retry cannot choose or reroll traits. Use domain-separated keccak256 and unbiased rejection sampling; no caller or timestamp entropy or biased modulo.

Expired intake not committed to a frozen batch may refund principal and unearned fees before randomness issuance. After issuance: no discretionary cancel, replacement seed or outcome-based refund. Provider failure stops new intake, preserves reservations and restores the same request; requests may remain pending indefinitely. No generic timeout refund without reviewed provider-specific design. Matured quest exits and ordinary unwrap remain independent. [VRF security](https://docs.chain.link/vrf/v2-5/security)

The 2-USDC fee discourages grinding; it does not prove unprofitability. Test the configured independent tier odds and premiums, higher-tier Payroll, avoided downtime and item costs, re-hiring and executable conversion costs. Wallet limits alone are insufficient. O1’s reference opening FDV is about USD 4,000: marginal 100,000 / 1B tokens ≈ USD 0.40 before impact; 2 USDC is 5× that value. This is a reference, not a JERKZ price forecast; validate against the actual Arc quote. [O1 opening liquidity](https://docs.o1.exchange/launchpad/trading/liquidity)

### 2.3 Wrapper and metadata behavior

The wrapper transfers existing ERC-20 tokens and ERC-721 NFTs. Unwrap switches the NFT to inactive metadata; wrap assigns a fresh character and updates its metadata pointer. The restricted pipeline in §10 publishes versioned current JSON and emits ERC-4906. It never edits an existing IPFS file in place.

Wrap uses a fixed fee, verified asynchronous randomness, random inventory-ID selection, frozen batches and percentage-weighted titles. The user cannot choose the NFT ID or character. Unwrap retires the generation while preserving protected Payroll records. Reusable pre-generated IPFS art and dynamic JSON support jobs and approved cosmetic variants; no runtime artwork generation.

Title supply has no caps. Each wrap draws title and expression independently from committed weights, then selects a compatible pre-generated option. Repeated titles and art options are valid. Use §2.2’s verified adapter and separate randomness domains for inventory selection, title, expression and art because these draws control financial rights.

## 3. Titles and employment

Birth Title is drawn at wrap and fixed within the generation. Current Job Title starts at Birth Title and can increase through earned promotions (§3.3); it controls job access, pay factor, firing protection and executive referral eligibility. Both are on-chain state attached to the NFT, not separate equipment. The catalog records Birth Title; live metadata exposes both titles. Promotions do not redraw birth rarity, art or expression. Cosmetics never promote a character. Every wrap draws independently with replacement using a frozen percentage-weight configuration. Tier weights sum to 10,000 basis points; no live or lifetime tier quotas, reserved rarity slots, depletion, balancing toward target counts or adaptive odds. A 1.5% CEO weight does not cap CEOs at 150; 0.1% VC does not cap VCs at ten. Only the collection’s 10,000 permanent IDs and funded inventory limit simultaneous NFTs. A new configuration may govern future uncommitted batches only; it cannot alter an accepted draw or existing Birth Title.

### 3.1 V1 tier parameters

Draw % is the independent probability of that Birth Title on each wrap; it is not a supply allocation or hiring success. CEO is rarer to draw than Intern (1.5% versus 40%), not less likely to be paid once held. Realized counts fluctuate and can exceed the percentage multiplied by 10,000. The weights remain constant through each frozen batch.

| Birth / current title | Birth draw weight | Current pay factor | Mint status / base hire chance | Strike per mistake |
|---|---|---|---|---|
| Intern | 40% | 1.00× | UNEMPLOYED / 60% | 100% |
| Junior / Associate | 23% | 1.10× | UNEMPLOYED / 75% | 80% |
| Specialist | 15% | 1.25× | UNEMPLOYED / 85% | 60% |
| Middle Manager | 10% | 1.50× | UNEMPLOYED / 95% | 40% |
| Director | 5% | 1.75× | UNEMPLOYED / 99% | 20% |
| C-suite | 3% | 2.00× | EMPLOYED / no roll | 0%; immune |
| CEO | 1.5% | 2.50× | EMPLOYED / no roll | 0%; immune |
| Board Member | 0.4% | 3.00× | EMPLOYED / no roll | 0%; immune |
| VC | 0.1% | Eligible-holder share (§8) | EMPLOYED / passive | No work draw |
| Founder | 2% | 1.00× | SELF_EMPLOYED / no roll | No firing |

Work categories by employee tier: Intern:support / errands; Junior:routine operations; Specialist:skilled work; Manager:team supervision; Director:department delivery; C-suite:executive operations (COO / CFO / CTO); CEO:leadership; Board:governance. Posting fields: job ID, fictional employer, eligible tier(s), work category, version. Categories add no hidden multiplier.

Intern through Board Member use employee quests. Founder uses venture quests with the same shifts and health but a separate high-variance book (§7); no employer, application or firing, and flops do not remove the role. C-suite, CEO and Board retain work locks, Clock In limits and health despite firing immunity. VC uses only holding income (§8), with no application, resignation or quest. Founder and VC are separate tracks, outside employee promotions. Every available birth role has an earning path; actual income still requires funded Payroll.

For duration weight u and integer title factor m scaled by 100, baseWeight = u × m; retain the unscaled product. Clean result keeps it; mistake gives floor(baseWeight / 2). Relative weights determine shares, not salaries. Higher factors improve equivalent uncapped outcomes, not every paycheck.

### 3.2 One-action work and hiring

**Player flow:** Put your Jerk to work → wait for the accepted shift → view the result / claim funded Payroll. The system selects a compatible job from a committed title-to-role mapping; browsing jobs is optional. Employment status is separate from shift status: UNEMPLOYED, EMPLOYED or SELF_EMPLOYED. There is no separate warm-up timer, application transaction or hiring confirmation. Existing shift lengths, hourly starts, health rules and funded limits remain.

1. **Accept once.** An eligible ACTIVE worker, employed or unemployed, selects a shift. Freeze employment, title, XP, effective hiring and strike odds, posting/mapping version, beneficiary and every daily entry before accepting custody. Unemployment is not a rejection reason. One hiring attempt per accepted shift, only if initially unemployed; reserve its first Payroll date against one permanent token-ID/day attempt across owners and generations. No protocol application fee; disclose network and randomness costs. No unfunded search-only XP farming.
2. **Resolve automatically.** Use the first entry’s verified cohort seed with independent JOB_APPLICATION domain, bound to shift, token ID, generation and frozen inputs. Success binds the committed compatible job and qualifies all entries of that shift for their Payroll draws. Failure records No Work Found: zero award, zero payout weight and no strikes for every entry of that shift. Both outcomes retain the full accepted lock, daily use, workload and XP schedule. A 48-hour shift has one hiring roll, not two. Its second group must resolve the first group’s hiring seed before calculating that linked entry; late results never extend custody. No retry, cancellation, item or referral may replace an accepted hiring result.
3. **Keep the job.** Employed workers skip hiring on later shifts; C-suite, CEO and Board start with protected appointments. Founder stays SELF_EMPLOYED through flops; VC remains passive under §8. Apply job acquisition at the first segment’s maturity, before that segment’s discipline. Failed search stays UNEMPLOYED. A hired worker can still draw No Pay or be fired by discipline; these are separate events. Queued successors require EMPLOYED or SELF_EMPLOYED status at acceptance; unresolved job search cannot queue another attempt. Accepted notice after later firing keeps its frozen financial rights.
4. **Return with experience.** Firing or resignation below C-suite sets UNEMPLOYED and the existing reapply delay; after it clears, the same work action handles the next job search. Resignation requires no active/queued work or unapplied results. Protected executives retain their standing appointment. Preserve earned titles, lifetime XP, health, history and calendar limits; firing/resignation resets careerStreak and promotionProgress. JUICE clears cooldowns, not unemployment or used attempts. §9 hiring items and referrals remain optional shortcuts before a shift, never additional required steps.

### 3.3 Career progression and XP

**Ladder:** Intern → Junior / Associate → Specialist → Middle Manager → Director → C-suite → CEO → Board Member. Founder and VC remain separate birth tracks; no promotion into or out of either. Board Member is the employee ceiling. Born executives keep their head start; a promoted CEO has CEO gameplay rights, while Birth Title still records original draw rarity.

**Thresholds:** Completed daily entries required at the current title: Intern 10; Junior 15; Specialist 20; Middle Manager 25; Director 30; C-suite 40; CEO 60. Board, Founder and VC have no next-title threshold. A birth Intern needs 200 qualifying workdays to reach Board without a reset. These are the v1 thresholds; commit progressionConfigVersion at generation birth. Later settings apply only to new, uncommitted generations.

**Credit:** Each completed, originally funded daily entry earns 1 lifetime XP once, including No Work Found, No Pay, Founder flops, a firing entry and accepted notice. An 8/16/24 shift earns one XP; 48 earns two across its reserved dates. No extra XP per hour, payment, item, click or keeper call. Only an employee still employed after ordered discipline gains 1 careerStreak and 1 promotionProgress; failed searches and notice after firing gain neither. A mistake or zero Payroll draw still earns career credit if employment survives. Founder gains a venture streak, never employee promotion progress. VC holding gains no work XP. Unaccepted/unfunded work gains nothing. Custody-fault notice keeps the original maturity schedule.

**Continuity:** Rest, weekends, holidays, cooldowns and unfunded dates pause the career streak; there is no calendar attendance penalty. Firing or voluntary resignation resets careerStreak and promotionProgress, not lifetime XP or already-earned Job Title. The firing entry cannot promote. Selling or transferring preserves all career state; unwrap retires it. Hiring items and referrals cannot buy promotion credit. JUICE preserves career progress; it resets only the distinct health/overtime counters in §5.

**Activation:** Apply performance, strikes, firing and XP in day order. Evaluate promotion only after all accepted career entries for that generation are applied, while employed. If promotionProgress reaches the committed threshold, advance one rung and subtract that threshold; retain surplus progress and careerStreak. Bind an automatic compatible appointment at the same employer, using a precommitted title-to-role mapping, without a warm-up, hiring roll or fee. Validate complete ladder mappings before activation. Update current Job Title and metadataRevision, then emit JobPromoted and ERC-4906. No user-selected title or extra random draw. Missing metadata delivery cannot block the on-chain promotion or exits.

**Experience effect:** Selected v0.20 starting balance: experienceBonusBps = min(2,000, floor(lifetimeXP / 5) × 100). For unemployed Intern–Director, effectiveHireBps = min(9,900, baseHireBps + experienceBonusBps). For their employee discipline, effectiveStrikeBps = floor(baseStrikeBps × (10,000 − experienceBonusBps) / 10,000). Thus every five XP adds one percentage point to hiring and removes 1% of the base strike probability, capped at 20 points / 20% relative reduction. Mistake chance still follows stress. Zero-XP Intern: 60% hire, 100% strike per mistake; 100-XP Intern: 80% hire, 80% strike per mistake. Promotions also improve the tier base. Executives/Founder retain immunity; VC is unchanged. Commit these parameters with progressionConfigVersion at birth; freeze XP and effective odds at shift acceptance for all its entries. XP earned during that shift affects only later accepted shifts. Validate this new balance in §13 before live funds; it is not a measured retention forecast.

**Financial boundary:** Promotion affects only work accepted after activation. Every accepted entry retains its title, job, pay factor, discipline and probability snapshots; no retroactive raise, firing immunity or changed outcome. A pending old-title firing must settle before promotion, preventing promotion from escaping discipline. Lifetime XP and careerStreak add no separate pay multiplier. Higher title factors redistribute the same worker budget and stay inside the per-NFT cap. Promoted C-suite, CEO and Board gain their standard future-entry firing immunity and sponsor eligibility; Founder/VC rights cannot be acquired through this ladder. Promotion changes live JSON, not the pre-generated birth image or owned cosmetics.

## 4. Calendar, shifts and custody

### 4.1 Workday and hourly starts

**Payroll day:** [09:30 America/New_York, next date 09:30). Date IDs, income closing, VC snapshots and named rest/weekend days use this calendar. Regular NYSE opening is the calendar anchor, not a daily player signup window. Never use fixed-offset EST year-round. Normal work is Monday–Friday; a weekend pass is required for any work interval overlapping a Saturday/Sunday Payroll day. Weekday gameplay continues on holidays and early closes. Market closure may delay conversion, never shorten accepted work. [NYSE hours](https://www.nyse.com/trade/hours-calendars)

MarketCalendar uses d = Gregorian calendar days since 1970-01-01, and maps d to Unix start S(d): d × 86,400 + 48,600 in EDT or +52,200 in EST. EDT applies from March's second Sunday through the day before November's first Sunday. A Payroll day contains 23, 24 or 25 elapsed hours. Find the interval containing the timestamp, never floor(timestamp / 86,400). Commit calendar version and actual timestamps; reject unsupported dates before accepting money or custody. [US DST rules](https://www.nist.gov/pml/time-and-frequency-division/popular-links/daylight-saving-time-dst)

**Hourly grid:** Starts are S(d) + k × 3,600 for 0 ≤ k < H(d), where H(d) = (S(d+1) − S(d)) / 3,600. These are local :30 starts. Repeated fall-back 01:30 starts have different Unix timestamps and IDs; spring-forward has no nonexistent 02:30 start. A normal request selects the first grid timestamp strictly later than block.timestamp. At an exact start, entry for that start is closed and the next hour is selected. The four-hour funding group can still accept its later scheduled starts until its final start (§6.3). There is no grace period, late addition or operator-selected cutoff.

Any eligible worker may choose **Put your Jerk to work** at any time, including while unemployed; §3.2 handles hiring inside the shift. Preview the next start, budget, work end, custody wait, dates, risk and remaining blockers before signing. If that next start lacks funding, show it as unavailable and leave the NFT unlocked; do not silently defer or create an indefinite waiting lock. A later hour requires a fresh quote when it becomes the next start; the explicit queued-successor exception is in §4.3. One entry per ID/date, custody, recovery, reapply delay and pending-result limits remain; there is no separate hiring warm-up. Hourly starts do not create hourly XP or extra daily pay entitlements.

### 4.2 Duration, entry dates and custody

QuestEscrow holds the existing ACTIVE ERC-721 and preserves its character. It is not HybridVault inventory. Clock In, maturity and withdrawal never unwrap, rewrap or redraw the NFT. Custody starts only in the atomic successful acceptance transaction, after all required budgets and entries are reserved. Work, including any job search, starts at the quoted grid timestamp; the preceding wait is less than one hour for a normal next-start quote and earns nothing. A failed search still uses that reserved shift and its workload. Employment gates the award, not custody duration.

| Shift kind | End / paid entries | Entry weights | Overtime entries |
|---|---|---|---|
| 8 | Start + 8 elapsed hours / 1 | 100 | 0 |
| 16 | Start + 16 elapsed hours / 1 | 175 | 1 |
| 24 | NextLocalDay(start) / 1 | 200 | 1 |
| 48 | NextLocalDay(NextLocalDay(start)) / 2 | 200 + 200 | 2 |

NextLocalDay uses the following local date at the same wall-clock time. For an ambiguous fall-back time, select the earlier valid Unix occurrence. For a spring gap, shift forward by the gap length. Apply this rule separately to both 48-shift segments, with the resulting timestamp as the next anchor. Publish exact start/end timestamps in the calendar table and verify them against IANA America/New_York over the supported range. Full-day labels can mean 23/25 elapsed hours and two-day labels 47/49; never promise exactly 24/48 elapsed hours. Every resolved endpoint remains on the hourly grid.

Only 8/16/24/48 are accepted; reject 12/36. A paid entry's date is the Payroll day containing its segment start. A 48 shift reserves two entries on consecutive dates at acceptance. Its second segment joins the funding group containing the hourly start at the first segment's end, alongside any 8/16/24 entries using that group. No separate duration budget or duplicate day-two payment exists. Maximum commitment is two paid entries, even when the final custody tail crosses a third Payroll day. That tail creates no additional pay or XP.

One permanent token ID may reserve at most one worker entry per Payroll date across all cohorts, owners and generations; the date's worker count cannot exceed 10,000. Work intervals cannot overlap. Also reserve every Payroll date touched by actual work against a new VC entitlement on that same permanent ID; an unwrapped replacement cannot collect VC pay for the tail of previously paid worker custody. These role-exclusion markers grant no extra worker award or progression.

Validate all segment budgets, employment track or hiring eligibility, projected health, pass permissions, career-backlog slots and physical work intervals before acceptance. A shift crossing a weekend or an already-scheduled crash/recovery interval must satisfy that restriction for the whole overlap. An interval ending exactly at a restricted boundary has no overlap. Crossing a calendar boundary is not free weekend work. Unfunded or invalid long work reverts every reservation and custody change atomically. Accepted start/end, weights, beneficiaries and commitments cannot be extended, repriced or rewritten.

### 4.3 Successors, recovery and exits

Allow one queued successor only when its start equals the current work end. The existing fixed beneficiary may quote and accept it before that successor's own start cutoff; unlike a normal next-hour request, this explicit successor can reserve a later start. All successor budgets must already be published and within the two-day publication horizon. Validate dates, physical intervals, projected leave and the two-entry career backlog at acceptance. Preserve beneficiary and continuous custody; show the extended custody end. A matured NFT remaining in escrow can start a new eligible quoted shift without transfer.

Newly earned overtime leave begins at the first 09:30 boundary at or after the end of the shift containing the second overtime entry, and covers one complete Payroll day. New work is blocked from that shift's end through the leave end; a partial trailing date is not an extra work opportunity. Do not postpone this leave by adding a successor. A separately accepted JUICE reset can clear it under §5. Existing accepted custody is never cut short or extended by a new health result. Existing Monday-crash restrictions must be checked before accepting the full interval.

Matured NFT exit must not depend on randomness, keepers, swaps, career settlement or CRCL transfers. The fixed beneficiary withdraws; sold or transferred NFTs retain health restrictions. The full original shift must end before any of its pay becomes claimable, including a 48 shift's first entry. A queued successor does not postpone pay for an already-ended original shift. Results can settle later; they cannot relock a matured NFT.

Emergency early exit is custody-only, enabled for all affected positions by an objective, recorded systemic custody fault and a new-intake pause; it is not a player cancellation or an outcome-selected refund. Preserve every booked entry, beneficiary, award, daily use, work interval and health/performance snapshot. Work accounting, XP and discipline mature at the original segment ends; claims retain the original full shift end. Reject new work or same-ID VC overlap during the original intervals even after emergency withdrawal. A new owner or generation cannot reuse those reservations. This paid-notice rule removes any ability to change the payout denominator, cancel a bad result or get a new draw. Normal early withdrawal remains unavailable; stock-transfer failure preserves the claim.

## 5. Health, overtime and performance

### 5.1 Stress and recovery

Birth expression is independent of title: Happy / relaxed, Neutral, Crying and Manic / crazy each have 25% weight in every tier. CEOs receive no calmer-face bias. Draw the profile first, then select a pre-generated compatible image; folder file counts cannot affect either distribution. Birth expression sets stress once: Happy 0; Neutral 20; Crying 45; Manic 60. Commit the art-to-expression mapping and weights before randomness. Store Birth Expression and Initial Stress separately from mutable Stress and Mood. No profile starts burned out; initial stress is neither a floor nor ongoing multiplier. Rest or JUICE can reach zero. Facial cosmetics cannot reset health or reroll the profile; new generations draw anew. Stress and Mood come from on-chain health state; metadata publication cannot change them.

| Booked entry / recovery action | Stress change |
|---|---|
| 8h / 16h / full day | −5 / +15 / +20 |
| Saturday or Sunday work | Additional +10 |
| Complete day without committed work | −20 |
| JUICE | Reset to 0; clear burnout and overtime streak |

Health workload is classified by the booked entry, not by counting incidental hours on every touched date: one workload event per paid entry at its segment start. A tail crossing 09:30 adds no second workload event, XP or pay. A complete off day must contain no booked entry and no actual work overlap. Two 8h entries on different Payroll dates remain separate normal entries even if fewer than 24 elapsed hours apart; the one-ID/date limit and non-overlapping custody still apply. Apply one weekend surcharge to an entry if any part of its actual work interval touches a weekend Payroll day. Combine base workload + weekend surcharge. Supportive Spouse reduces positive net change to floor(change × 80%); zero, negative and off-day changes stay unchanged. Then clamp 0–100. Examples: 16h 15→12; full day 20→16; weekend full day 30→24; weekend 8h 5→4. Freeze perk ID and version with each entry.

| Stress | Mood | Mistake chance |
|---|---|---|
| 0–29 | Steady | 5% |
| 30–59 | Stressed | 15% |
| 60–79 | Depressed | 30% |
| 80–100 | Burned Out | 50% for the accepted entry reaching this band |

Apply complete idle days before eligibility; apply each booked workload once at its segment start and freeze resulting stress and risk. Project multi-day segments in order. A segment may reach burnout; the following segment cannot start while latched. Later rest or items cannot improve frozen risk. Boundary JUICE applies after earlier accepted workload and before new workload. Calculate idle recovery with bounded arithmetic, not one loop per missed day. Wallet or non-work custody counts as rest; transfers and job changes do not reset health. A custody-fault withdrawal does not cancel scheduled workload, fatigue or crash restrictions; all accepted accounting matures on its original schedule.

At 80, latch burnout: block new work, including job-search shifts until complete off days reduce stress below 60 or JUICE clears it. Starting at 80 needs two days to 40; 100 needs three. Example: full day at 60 reaches 80; weekend full work adds 30 versus 20, or 24 versus 16 with spouse. Health and calendar restrictions still apply. Overlapping burnout, recovery and crash requirements do not add duplicate off days.

**Overtime:** Examples below use 09:30 starts unless stated; later starts follow the exact interval rule in §4.3. A 16h or full-day booked entry is overtime. Two adjacent overtime entry dates schedule the full recovery interval in §4.3, after the accepted work ends. Normal 8h or a full off day breaks the streak. Without JUICE: Mon 24+Tue 24 or Mon 16+Tue 16 → Wed off, Thu available; Mon 16+Tue 8+Wed 16 → no consecutive-overtime leave. A 48 shift after one overtime day fails because its second segment would be leave. JUICE clears leave and overtime streak without shortening accepted locks.

### 5.2 Weekend and JERK JUICE

Every entry with actual work overlap on Saturday or Sunday needs a pass for that named weekend. A late Friday 8h shift can cross into Saturday; a Friday 48 shift may touch both weekend days. End is exclusive: Monday 09:30 end does not work Monday. One pass covers both weekend days, transfers with its generation, expires afterward and disappears on unwrap. No extra weekend pay multiplier.

JERK JUICE: fictional consume-on-purchase item, 1,000 JERKZ (1% of principal) per dose. Owner or fixed quest beneficiary pays. One dose clears **all current or already-scheduled gameplay cooldowns**: overtime recovery, Monday crash, burnout and firing or resignation reapply delay; reset stress and overtime streak to zero. At purchase the player may select no pass or one pass for the current or next New York week, included in the same price; there is no standalone pass SKU or arbitrary future-week reservation. A week starts Monday at 09:30 under MarketCalendar. Preserve art, both titles, careerStreak, promotionProgress, employer or unemployment, XP, strikes, Times Fired, wardrobe and spouse and claims. No use cooldown or immunity; every later reset costs another dose.

First weekend work reservation schedules one Monday crash per generation per week. Persist used and cleared markers so later reservation or keeper cannot recreate a cleared crash. Saturday-only work still crashes Monday. No weekend work means no crash; a dose before first reservation cannot pre-clear its future crash. Sat 8+Sun 8 → Monday off, Tuesday available unless another dose clears Monday. Sat 24+Sun 24 → one later dose clears both Monday crash and overtime leave. Sunday 48 fails unless the already-scheduled Monday crash was cleared; alternative Sunday 24 → new dose + queued Monday 24.

Consume with no active or queued work, or atomically reset-and-clock-in effective at the selected hourly start if all earlier accepted work ends by then. No reset inside accepted multi-day work. Persist generation, dose nonce, effectiveAt and cleared restriction IDs; apply once, project pending reset before successor validation. Freeze weekly price; bind generation, nonce, maxCost and deadline. Burn, reset, pass and combined Clock In succeed or revert together.

JUICE cannot bypass locks, daily entry / hiring-attempt limits, pending-result caps, funding, settlement waits, pauses; cannot restore a job, erase old mistakes or alter claims. Dedicated hiring services bypass the next hiring roll before work is accepted (§9). Validate repeat-dose participation / cost against the fixed Payroll budget; no added emissions.

### 5.3 Mistakes, discipline and career settlement

One verified cohort seed provides independent WORK_PERFORMANCE and DISCIPLINE domains per cohort / ID / generation / paid date; no second provider request. Freeze risk, title, duration, XP and effective strike odds before requesting the seed. A failed hiring gate has no mistake penalty or discipline; its frozen workload and XP still apply. Mistake halves weight and resets clean streak; an employed Intern–Director gains a strike only when the tier discipline draw succeeds. Protected executives can lose weight but never gain strikes or get fired. Founder has no employer discipline; VC has no performance draw.

Three consecutive clean entries clear one strike and reset clean streak. Two strikes fire an Intern–Director: clear job, strike and clean counters, increment Times Fired, preserve earned Job Title, stress and XP, and reset careerStreak and promotionProgress. Accepted work remains paid notice with committed outcomes and XP; do not fire the already-unemployed character again. Block new work until the reapply delay and other restrictions clear; the same work action then includes hiring. **No Pay never adds a strike or directly fires a worker.** Only the independent, frozen mistake/discipline result can do that; both events can coincide. C-suite, CEO, Board and Founder retain firing immunity. This separates ordinary payout variance from career loss while retaining the health-driven risk of being fired.

Earliest reapply date = max(notificationDay, lastCommittedWorkDay) + 2: one complete uncommitted day after the later date. Other health blocks may delay further. JUICE clears delay and health restrictions, not accepted work, pending results, job loss or used hiring attempts. Resignation below C-suite uses this delay.

Apply matured career results in generation and then day order with a bounded cursor; financial awards can finalize independently from immutable performance snapshots. Max two accepted daily entries with unapplied career results per generation; a 48 shift uses both. Show backlog to buyers; no unlimited Clock In during provider outage. Retirement isolates old career results, not old money. Grant one XP per completed daily entry once; apply career credit and promotion under §3.3. Lifetime XP alone has no pay multiplier. Evaluate promotion after the accepted-result backlog is empty; retired callbacks cannot change the replacement generation. Never apply XP, strikes or promotion before the relevant accepted segment ends, even if its financial result is already public.

## 6. CRCL Treasury and budget

### 6.1 Income, conversion and liabilities

- **O1 creator share:** Actual entitlement; CRCL receipts credited directly. Documented reference: 0.5% of trade amount in paired asset.
- **Distinct LP fees:** Actual separately claimable fee income only, never principal; baseline zero until verified.
- **Hybrid fee:** Earned 2-USDC wrap fee only after successful ACTIVE delivery; pending fees remain protected. Hybrid principal stays in its separate vault. Metadata updates and delivery retries add no protocol fee.
- **NFT royalty:** Convert currency actually received; ERC-2981 does not ensure payment.
- **Optional grant:** Credit actual CRCL bought from approved and received funding, not assumed USD value.

O1 reference launch positions belong to LaunchHook; no project-owned transferable LP position. Never count the same fee as creator and LP income. Public reference deployments do not prove Arc availability. [O1 fees](https://docs.o1.exchange/launchpad/trading/fees-referrals), [O1 liquidity](https://docs.o1.exchange/launchpad/trading/liquidity)

Allowlist exact assets, routers and routes. Bind maxInput, minStockOut, deadline and treasury-only destination; measure balance deltas and limit approvals. Configure bounded batch size, liquidity, freshness and deviation, slippage and execution caps per route; never zero minimum output. Verify CRCL issuer restrictions, corporate actions and market availability. Reject transfer-tax or rebasing assets unless a reviewed and audited adapter proves accounting; v1 needs stable raw units. Failed or invalid conversion retains the fee bucket and stops new conversion, not exits or funded claims. Unconverted, estimated, unclaimed or future fees cannot fund pay. Project-funded trades’ returned fees are recycled, excluded from income EMA.

Use uint256 raw amounts, full-precision multiplication / division and checked ranges; no floating monetary liabilities. Display USD equivalents only as timestamped estimates. One economic treasury has disjoint ledgers:

stockBalance >= seedUnreleased + recurringReserve + committedEpochBudgets + finalizedUnclaimedAwards + vcReserved

Finalization moves liabilities, never counts twice. vcReserved already includes vested and unvested credits. Unused allocations, cap clipping, dust return to recurringReserve, excluded from external-income EMA. Earned claims never expire or fund new epochs. Gas, operator and VRF funding is separate; never spend principal, pending payments or owed stock.

### 6.2 Daily pacing

Start seed S = 0, reserve F = 0, EMA E = 0. Close income once per NY day, including zero days. I = net externally funded CRCL received that day; F = uncommitted recurring stock immediately before this commitment:

E_new = floor((3 × E_previous + I) / 4)  
incomeTarget = floor(4 × E_new / 5)  
slowRelease = floor(F / 180)  
reserveCeiling = floor(F / 30)  
recurringBudget = min(max(incomeTarget, slowRelease), reserveCeiling)

Seven-day-span EMA, alpha ¼. Target retains income; ceiling limits depletion; slow release supports a declining drought stream that approaches zero. Receipt count cannot cause multiple daily EMA updates. Zero funded work budget: show “Payroll funding”; do not lock NFTs. VC funding and rate is also zero until actual funds exist; no promised large opening pay.

**Funded launch schedule:** After approved funds are received and converted, S is the actual CRCL received, not a promised dollar balance. Seed spending is capped by the received amount and the 50,000-USDC request; no purchase is authorized here. Keep the entire deposit protected until first publishing the seedStartDay budget. At that activation set D = floor(3 × S / 4) as distribution stock and credit S − D to recurringReserve once, excluded from income EMA. Earlier budgets may use fee reserves only. Release D on a fixed, declining 180-day schedule. The seedStartDay is the first 09:30 boundary at least 48 elapsed hours after the single seed deposit is committed. For 0 ≤ k ≤ 180, released(k) = floor(D × k × (361−k) / (180 × 181)); tranche(k) = released(k) − released(k−1), 1 ≤ k ≤ 180. This makes larger early payouts taper smoothly toward the fee-supported budget, rather than ending a flat subsidy abruptly. Rounding may change adjacent tranches by one raw unit. Each funded date may reserve only its own tranche; lookahead cannot borrow later tranches. Protect every unreleased tranche. The reserve quarter follows the same 30-day ceiling and 180-day slow release as fees; it is not a second guaranteed bonus or money available to operations. Day 181 has no new tranche. Missing/unfunded dates release their scheduled tranche into recurringReserve once when that date closes, never into a later catch-up prize. New funding cannot reset or shorten this schedule; later donations go to recurringReserve. UI shows tranche, free reserve, committed/owed stock and seed end date separately.

B = recurringBudget + scheduledSeedTranche  
vcBudget = floor(B × 500 / 10,000)  
workBudget = B − vcBudget

V1 split: 5% VC / 95% work. Publish each day once, monotonically, at most two Payroll dates ahead of the current date and strictly before that funded date starts at 09:30. Budgets can be published earlier within that horizon to accept day-two reservations. A budget must exist before any dependent Clock In; publication does not imply player enrollment. Missed date stays unfunded; never backdate or reopen or later release missed budgets as a lump sum. Deduct full B immediately from available funds; freeze stock asset, total and split, calendar timestamps, weights, books and configuration hashes. Later receipts fund later dates. Future publication reads the latest fully closed income EMA; it cannot advance income closure, assume future fees or apply multiple EMA updates. Freeze that source day with each budget. The two-day lookahead only earmarks held stock and cannot increase any daily cap. Two-entry shifts need both paid dates funded before custody starts. Initial funding uses held stock only.

### 6.3 Four-hour funding groups, hourly work starts

Keep hourly Clock In and the four shift lengths. Pool all durations across each consecutive group of up to four hourly starts. On an ordinary day there are six funding groups, not 72 separate start/duration pots. A 48 shift uses one entry in each of its two paid dates' groups. Each entry retains its own segment start/end, full-shift end, duration and risk. Duration and title affect allocation inside the shared group only; no second duration-specific budget exists.

Let H be the day's 23/24/25 hourly starts; k their zero-based index. Group g = floor(k / 4). Its first start index a = 4g and exclusive end index b = min(a + 4, H). Commit:

cohortBudget = floor(workBudget × b / H) − floor(workBudget × a / H)  
firstStart = S(day) + a × 3,600  
sealAt = S(day) + (b−1) × 3,600

There are ceil(H / 4) groups: six on 23/24-hour days, seven on 25-hour days. Each gets its exact share of elapsed start slots; the last 3- or 1-start group is proportionally smaller. Allocations sum exactly to workBudget in raw units. They are sub-ledgers of the single daily commitment, never additional money. Zero-budget groups reject paid intake. All groups, budgets, calendar, outcome books and versions are fixed at publication.

A normal player may join only the next hourly start; a successor uses the explicit §4.3 rule. Each start closes strictly at its timestamp, even while later starts in its group remain open. At sealAt the last start closes, all committed inputs become final, and one verified seed may be requested. Late keepers cannot add entries or change membership. The earliest 8h shift ends at least five hours after sealAt; no work must wait for another member's 16/24/48-hour end. Provider failure can still delay a result.

Financial outcomes can finalize before work ends and are public on-chain. Display “Result set : unlocks at [time]” without pretending results are secret or funds are already claimable. No later entry, cancellation, transfer, JUICE, firing, retirement or custody-fault exit can change a reserved entry's eligibility, weight, outcome, denominator or maturity. Health/risk is frozen at acceptance; career consequences apply only at the original segment end. Financial finalization and custody maturity are separate states.

An empty group returns its allocation only after sealAt. A nonempty group returns unused components, cap clipping and dust once its fixed awards finalize. No current group may borrow another's reserve; unused money feeds only later, unpublished daily budgets under §6.2. Quiet groups may pay more per entrant than busy groups, but all start times in one group share the same denominator. This disclosed timing choice never increases daily spending. No adaptive odds, discretionary top-ups or personalized rewards are permitted.

### 6.4 Reference scenarios and economic limits

Constant-price USDC equivalents with six-decimal integer accounting; no live CRCL quote or income forecast. Each step closes one day's receipts and commits the next dated budget. Assume every budgeted unit is eventually paid, with no unused returns, price moves, execution losses or operating costs. Values are total Payroll B before the 95/5 work/VC split. Funding shown below is hypothetical until actual receipt.

| Step | Fees only; 1,000/day | 50k seed + 1,000/day fees | 50k seed; no fees |
|---|---|---|---|
| 1 | 33.33 | 614.36 | 483.81 |
| 30 | 638.34 | 1,147.24 | 406.69 |
| 60 | 800.00 | 1,078.55 | 328.54 |
| 90 | 800.00 | 1,009.48 | 251.78 |
| 180 | 800.00 | 802.30 | 27.92 |
| 181 | 800.00 | 800.00 | 25.48 |
| 365 | 800.00 | 800.00 | 9.14 |

With the seed and no income, maximum cumulative commitments by day 180 are 45,414.31; 4,585.69 remains free. By day 365, 1,636.07 remains. This is gradual depletion, not perpetual yield; stock-price declines can reduce all displayed dollar values. Fees stopping completely after day 30 leave the funded plan with budget 55.44 and free reserve 9,512.17 at day 180. The fees-only path has budget 41.39 and reserve 7,408.33. Zero funding and zero fees pay zero. No budget can spend future income, unreleased stock, NFT principal or existing claims.

Attendance illustration: a fixed 1,000-unit daily **worker** budget, 24 hourly starts in six groups, 60 simulated days for each population, uniform start selection, 60%/25%/15% 8h/16h/full-day entries, 2% Founder, employee birth-tier weights excluding VC/Founder, and a fixed 15% mistake rate. This illustration assumes employees already have jobs; it predates the v0.20 hiring gate and XP adjustment. It isolates conditional payout distribution and does not model job-search failures, retention, promotions, fees, actual health histories or token prices. Percentiles include zero results; simulated mixed-track positive-result frequencies are near 79%, because Founder loses more often. No Monte Carlo result replaces the exact budget-conservation bound.

| Paid entries/day | Budget distributed | Mean per entry | Median per entry | 95th percentile |
|---|---|---|---|---|
| 100 | 98.78% | 9.8778 | 6.5120 | 35.3798 |
| 1,000 | 100.00% | 1.0000 | 0.6184 | 2.5284 |
| 10,000 | 100.00% | 0.1000 | 0.0614 | 0.2258 |

The four-start groups reduce idle allocations compared with isolated start/duration pots; they do not enlarge B. All-in-one-group attendance can still leave the other groups unused. Timing choice and concentration must be disclosed, not disguised as fixed wages. Conditional jackpots can be large while the typical entry is small; always report both and include zeros.

At total daily Payroll B=800, worker funds are 760: 100 / 1,000 / 10,000 equal-weight daily entries have maximum average worker awards 7.60 / 0.76 / 0.076 before unused funds and rounding. After the seed, sustained average worker pay of 1 per entry for 10,000 entries needs at least 10,000 / (0.95 × 0.80) = 13,157.90 net daily fee units. At the documented 0.5% creator share this corresponds to about 2.63 million of qualifying daily trading volume, before execution costs and assuming no other income or unused funds. It is a funding requirement, not projected volume. More participation divides the same budget; neither a payout label nor the launch seed promises meaningful dollar income at every population size.

## 7. Worker Payroll

### 7.1 Lottery and allocation

Commit one employee and one Founder 10,000-slot outcome book per funding group before any Clock In. Reset only for a new funded group, never a retry. Birth track selects the book; accepted current Job Title supplies pay and discipline factors. Sample without replacement in canonical accepted-entry order using lazy Fisher–Yates, independent EMPLOYEE_LABEL / FOUNDER_LABEL domains and unbiased rejection sampling. No draw occurs before the whole group's final start closes. Duration, XP and mistakes do not change the conditional Payroll book odds. Draw a label for every accepted entry in canonical order, even if hiring fails; a failed gate overrides the displayed result to No Work Found and sets ordinary/bonus weight and award to zero. This preserves book order and prevents outcome-dependent selection. One draw per permanent ID/paid date; a 48 shift draws once in each paid-date group. Partial books need not produce a jackpot.

| Outcome | Employee slots / chance | Founder slots / chance | Components |
|---|---|---|---|
| No Pay / Flop | 2,000 / 20% | 7,000 / 70% | None |
| Minimum Wage | 6,000 / 60% | 2,000 / 20% | Ordinary |
| Bonus | 1,900 / 19% | 800 / 8% | Ordinary + score 1 |
| Jackpot | 100 / 1% | 200 / 2% | Ordinary + score 25 |

These are fixed v1 opening book probabilities, not independent with-replacement draws or promised dollar returns. Conditional on already having or finding work, employee positive-label chance is 80%, with Bonus or Jackpot on 20% of draws; Founder has 30% positive labels and twice the employee jackpot probability, offset by many more flops. Neither track is promised a higher expected income. For an unemployed zero-XP Intern, nominal positive-result chance is 60% × 80% = 48% before rounding; 40% find no work and 12% find work but draw No Pay. Existing employees skip this hiring gate. XP and promotions improve access, not the conditional book or total budget. No Pay has no extra discipline effect (§5.3). Minimum Wage is a variable game label; zero after rounding is displayed as 0 CRCL, not a paid wage. VC has no lottery.

Both books share the same group's budget. Let B_w = cohortBudget; w_i = effective title/duration/performance weight, or zero after a failed hiring gate; W = sum of eligible positive-label weights; q_i = w_i for eligible Bonus, 25 × w_i for eligible Jackpot, otherwise 0; Q = sum(q_i). Allocate 60% to ordinary pay and 40% to larger wins:

ordinaryBudget = floor(3 × B_w / 5); lotteryBudget = B_w − ordinaryBudget  
ordinary_i = positive label and W > 0 ? floor(ordinaryBudget × w_i / W) : 0  
extra_i = Q > 0 ? floor(lotteryBudget × q_i / Q) : 0  
award_i = min(ordinary_i + extra_i, floor(dayWorkBudget / 10))

The 10% daily worker-budget cap applies once per permanent ID/paid date across all components. It uses workBudget, excluding VC funds. Group allocations, not a target return, bound total awards. Zero denominators leave the relevant component unused. Never redistribute clipped, unassigned or rounded money inside the group. Return it to recurringReserve after finalization. Higher title/duration and rare jackpots redistribute held stock only. More wallets or a refreshed generation do not bypass permanent-ID dates; multiple genuine IDs still have multiple entries. The cap is per ID, not a claim of whale-proof or Sybil-proof distribution.

Arithmetic example: B_w=1,000 units, a hypothetical complete employee book, 10,000 equal-weight/no-mistake entries, six decimals and nonbinding daily cap. No Pay 0; Minimum Wage 0.075000; Bonus 0.165909; Jackpot 2.347727. Total 999.999800, dust 0.000200; Bonus≈2.21× and Jackpot≈31.30×Minimum Wage in this example. Real groups are smaller and mix titles, durations and performance; these ratios are not advertised payout multipliers. A lone entrant cannot receive more than the funded group and daily cap. Winning more frequently does not guarantee profit after gas, item purchases or token/stock price changes.

### 7.2 Cohort settlement and end-of-shift claims

Each worker cohort has states FUNDED → OPEN → SEALED → RANDOMNESS_PENDING → ASSESSING → ASSIGNING → FINALIZED; zero budget is UNAVAILABLE. It is the four-hour funding group in §6.3, not one start/duration combination. Daily funding and VC accounting stay separate. Worker randomness needs at most ceil(H / 4) requests per funded date: six on 23/24-hour days and seven on 25-hour days, fewer for empty groups. Separately fund request and keeper costs. Identify a group by funded date, group index and committed version, binding its exact start grid and sealAt. Entries carry their own ends. Seeds, immutable membership, W/Q, cursors and awards belong to that group.

1. Publish the funded immutable daily partition (§6.3). At Clock In reserve every required entry and bind ID, generation, shift, original full shift end, beneficiary, cohort, paid date, canonical index, job, title, duration, track, stress, risk, perk, strike odds and versions. No promise can use unconverted or unreceived fees.
2. Each hourly start rejects late entries; the last start closes all group membership at sealAt. Permissionless seal records all previously accepted entries, including reserved day-two work. At or after sealAt request one bound seed. All accepted entries remain in their immutable cohort; the precommitted hiring gate determines payout eligibility. No post-draw cancellation, discretionary removal or completion-dependent change is allowed. An 8h shift never waits for another member’s longer work.
3. Bounded assessment resolves each linked hiring gate, then derives immutable performance and effective weight and queues career results for application only after each segment matures; draw labels in canonical order using the separate cohort/track domains. Aggregate that cohort's W and Q, then assign awards. Persist phase and cursors; no skipping, reordering or repetition. Max 100 entries per call. Old generation financial processing cannot modify a replacement character.
4. Move exact awards from the cohort's committed reserve to finalizedUnclaimedAwards; release only that cohort's unused allocation once. The daily ledger must reconcile its sub-ledgers without reserving or releasing the same stock twice. Empty sealed cohorts need no randomness and cannot reopen. Cohort completion cannot cancel VC pay or release another cohort's funding.

**Claim rule:** For each entry, claimableAt = max(originalShiftEnd, finalizedAt). Both completed entries of a 48 shift wait for the full original 48-shift end; if one result is still pending, a finalized eligible entry may be claimed without waiting for the other. A queued successor does not move the earlier shift's claim gate. Custody-fault withdrawal cannot accelerate this gate or change the accepted financial result (§4.3).

No Friday release, weekly waiting period, additional claim delay, protocol claim fee or discount applies. Remove Payday Advance and its quote, method, event and fee configuration from this MVP. There is no borrowing against unfinished work or unresolved outcomes. A zero award is recorded and displayed without requiring a zero-value token transfer. VC vested credits remain pull-claimable under §8.

Example on an ordinary weekday: Clock In at 12:10 NY, next start 12:30, 8h work ends 20:30. The 09:30–12:30 funding group seals at 12:30. If its result finalizes at 12:34, the award is known but claimable only at 20:30. If finalization instead arrives at 20:34, claim then. The NFT remains withdrawable at 20:30 in either case. No daily reset, longer member shift or Friday gate applies. These are illustrations, not latency guarantees.

The original work beneficiary is fixed at acceptance. Sale, firing, unwrap, promotion or a later VC role cannot redirect or erase old pay. Pay slips use the frozen character key and title, never current tokenURI. Claims are explicit pull transactions, not automatic wallet transfers. Support at most 20 entry IDs per user claim, reject duplicates and invalid recipients, mark each paid and reduce liability before external transfer; failed transfer reverts the entire claim transaction. Transfer-restricted CRCL may delay delivery but cannot erase the reserve or extend NFT custody. All completed authorized work retains its original financial snapshots.

UI states: Work Scheduled, Searching / Working, Result Set / Locked, Result Pending, No Pay, Available to Claim, Claimed. Show exact start, work end, result readiness and claim status separately. Include Searching for Work, Job Found and No Work Found as substates/results of the same shift; expose a set hiring result as pending maturity until it applies. Show gross and claimable CRCL, with zero protocol claim fee; network costs remain separate. Minimum Wage, Bonus and Jackpot are variable results. Quotes show actual funded budget, current/max group weights, odds, four-start group allocation and ranges, not a fixed stock return. A different funding group’s attendance can change the eventual share even with the same title and duration. Show payout percentiles from settled groups, including zero results, sample size and period; never show winners-only averages, fixed APY, guaranteed minimum income or theoretical maximum as typical pay.

## 8. VC Payroll (MVP)

Only an ACTIVE generation assigned the on-chain VC title has passive MVP income; C-suite, CEO and Board remain workers. Metadata labels cannot create VC rights. VC earns in its wallet without a quest or daily claim requirement. Accrual does not depend on performance, stress, overtime, crash, hiring or backlog. Seven-day accrual covers funded intervals only, not guaranteed future income or APY. Claims are pull transfers, not a transfer every second.

VC count is variable; there are no numbered or reserved seats. Daily allocation: freeze N, the number of eligible ACTIVE VC generations at 09:30, using historical eligibility intervals, not keeper arrival time. A generation qualifies when eligibleFrom <= start and it was not retired before start; retirement exactly at start contributes zero time but remains in that day’s frozen cohort. If N > 0, perVCPay = floor(vcBudget / N); otherwise no accrual and the whole VC budget returns after day-end. Example B = 1,000 CRCL gives 50 VC / 950 work: one full-day eligible VC earns 50 CRCL; ten earn 5 each; twenty earn 2.5 each, before integer rounding. New eligibility waits for the next day; a mid-day retirement does not increase the remaining cohort’s rate. For committed start and end:

vested(t) = floor(perVCPay × clamp(t − start, 0, end − start) / (end − start))

Use actual 23 / 24 / 25h interval and full-precision raw math. Unfunded or sub-unit amounts yield zero. A fresh ACTIVE generation becomes eligible at **next** 09:30, even if activated exactly 09:30; assigned or inactive characters earn zero. Ordinary transfers preserve eligibleFrom; permanent ID/date usage and physical work-tail role exclusions in §4.2 prevent VC/worker dual pay through resets.

**Ownership:** before every ERC-721 transfer, checkpoint vested pay to old actual owner, then move future accrual with the NFT. Before unwrap, checkpoint earnings and end the interval; old credits persist, unearned reserve releases only at day close. Replacement does not inherit credits or eligibility. Third-party custodian owns future credits while holding and must support recovery; disclose this, never assume depositor rights. Noncustodial sale listing changes nothing; HybridVault inventory excluded.

**Index:** I(t)=sum of prior funded perVCPay + current vested amount. Freeze each funded day’s eligible count and perVCPay at its boundary from authenticated eligibility history; later wraps, retirements or delayed keepers cannot change that day’s denominator. Maintain scheduled next-boundary activations, cancellation of pre-eligibility retirements and timestamped retirement counts. Freeze due rates before applying a lifecycle action at the same timestamp. Before publishing another budget, materialize rates for due published epochs; at most three published epochs may await boundary pricing because publication is limited to two days ahead. Transfer/claim hooks may materialize this bounded queue and use immutable index prefixes with bounded binary lookup; they cannot wait for a keeper or scan all NFTs. Gaps=0, no backdated funding. Credit I(now)−I(max(eligibleFrom,lastCheckpointAt)), zero before eligibility; advance checkpoint. Differences of cumulative floors prevent gains from repeated checkpoints or transfers. Test the bounded implementation, including silent periods and exact-boundary event order.

Transfer hooks write accounting only: no CRCL call, external keeper, VRF or image-service dependency. Transfer failure reverts checkpoint; pause cannot disable hooks or funded accrual. Preserve financial eligibility intervals across retirement. For a cohort member retiring during a funded day, record perVCPay − vested(retirementTime) once as retiredUnvested for that date; generations not in that cohort contribute zero. At day-end return unused = vcBudget − N × perVCPay + retiredUnvested once (or full vcBudget if N=0). This includes integer dust and unvested time, never previously vested credits; no per-holder close loop or redistribution. Transfers change beneficiaries, not that day’s per-NFT rate. Delayed reconciliation only delays recycling.

vcReserved = fundedVC − actualVCClaims − reconciledUnused

This already includes wallet credits; never add twice. Worker finalization cannot release VC reserves; VC claims cannot spend worker reserves, bridge reserves or pending funds. Late close keeper delays recycling only. claimVCStock checkpoints ≤20 caller-owned IDs and pays only caller credits; former owner can pass an empty list. Effects before transfer; failed claim restores state. Show funded rate, eligibleFrom, vested and claimable CRCL, custody and unwrap effects and zero rate for unfunded dates. Old worker and VC claims survive later opposite-role generations without new same-ID daily entitlement.

## 9. Items, cosmetics and referrals

### 9.1 Burn and shop contract

Every item uses the buyer’s spendable, approved JERKZ. NFT operator approval alone cannot spend the owner’s tokens. Verify exact buyer-balance and totalSupply decrease before granting the item: burnFrom or atomic receive+burn. No recoverable transfer or dead-address substitute, reserve burn or remint. Burns do not fund CRCL Payroll.

**Integration blocker:** O1’s documented Robinhood / Monad LaunchToken has no burn function. An Arc-compatible, burnable O1 launch path remains unverified. Resolve before item sales; do not silently change launch platform, fee model or burn requirement. [O1 token interface](https://docs.o1.exchange/launchpad/reference/events-functions)

The following names, prices and perks are the v1 shop baseline; exact pre-generated item/appearance preview and selection, no random paid shop rerolls. Prices in JERKZ, frozen by purchase week. Catalog commits ID / version, type, slot, base / arm compatibility, asset hash / URI, price and explicit perk; never edit purchased SKU asset / perk in place.

- **Barber / 250:** Spreadsheet Fade, Liquidity Mullet, Recession Buzzcut. Paid hairstyle service; each new haircut costs again.
- **Tailor / 1,000:** Interview Special, Middle Management Beige, Overleveraged Pinstripe. Suit unlock.
- **Optician / 500:** Insider Shades, Spreadsheet Goggles. Glasses unlock.
- **Watch dealer / 1,500:** Borrowed-Time Watch, Golden Handcuffs. Wrist accessory unlock.
- **Arms / props / 500:** Diamond Hands, Paper Hands, Emotional Support Briefcase, fourth coffee. Compatible optional layer; default none. Watch bundle includes required compatible wrist layer in quote. Complete compatible appearance variants must be pre-generated, approved and present in the committed IPFS catalog before sale; approved source art remains unchanged.
- **Vanity desk / 250:** Fake MBA plaque, Employee of the Minute badge, cubicle backdrop, Expensive Taste nameplate. No pay effect; cannot replace Job Title.
- **Vegas Wedding / 2,000:** Trophy Wife, Office Sweetheart, Retired Influencer; fictional personas, same Supportive Spouse perk (§5). One active; replacement same price, no stacking.
- **Divorce Papers / free:** Clear relationship / perk, no payment barrier. Preserve health and career.
- **JERK JUICE / 1,000:** One reset and optional weekend permission (§5).
- **College Degree / Purchased Referral / Bribe a CEO / 1,000 each:** Single-use immediate hire (§9.2); different names and badges, same benefit.

Wardrobe is generation-bound state, not ERC-1155 items. Free equipment changes and removal of owned suits, glasses, watches and props (gas applies); reject duplicate unlocks. Haircuts are services, not style unlocks. Birth appearance remains free fallback for that generation. Reject incompatible or ungenerated combinations before charge; never silently alter another slot, substitute another character or queue runtime artwork generation. Cosmetics cannot change title or rarity weights, job, XP, odds or pay weight; no promised scarcity or premium. Transfers preserve items; unwrap invalidates their generation without refund. Later wraps cannot equip retired purchases. JUICE preserves wardrobe and spouse; it never creates a generation or redraws birth traits.

Owner or fixed quest beneficiary may purchase or equip with bound generation, shop nonce, catalog version, maxCost / deadline. Burn, grant and equipment change, nonce and metadata commit atomically; failure changes nothing. Each mutation consumes one nonce and updates the metadata revisions in §10. It never invokes hybrid randomness or changes the token ID. Marriage or divorce requires no active or queued work; cosmetic-only changes may occur during work. Spouse lasts until replacement, divorce or unwrap; only positive stress changes shrink (§5), with no reset of stress, strikes or cooldowns, extra hours, bypass, pay factor or CRCL entitlement.

### 9.2 Instant-hire services

An unemployed Intern–Director with no active/queued work or unapplied results may buy an automatic compatible appointment: skip the next hiring roll, clear reapply delay and set EMPLOYED immediately. Preserve title, health, custody and Payroll limits. Degree or referral badge may remain cosmetic history; hire benefit is consumed. Exclude already-employed, Founder and automatically appointed executives or VCs.

An accepted job-search shift cannot be bypassed or changed, even before its seed request. Wait for its full end and apply its results before buying a hire. A failed search that day does not block a later deterministic paid/sponsored hire, but its permanent daily work reservation still blocks another entry that date. Failed appointment reverts burn and state. The work screen may include this optional purchase in the same atomic acceptance transaction, before employment is snapshotted; purchase, hire and shift must all succeed or revert. Health, funding and completed-work requirements remain.

### 9.3 Executive referral campaigns

An owner of an active C-suite, CEO, Board or VC may opt NFT / generation into a campaign; Founder excluded. Link binds chain, NFT contract / ID, generation and campaign nonce. Every transfer advances ownership nonce and invalidates campaign, even round-trip transfers; unwrap invalidates it too. Re-enable for a new link. Link validation reads the active on-chain role and ownership; metadata refresh cannot activate a campaign or restore a retired link.

The system selects a compatible job; the recipient receives §9.2 bypass free of JERKZ, optionally with the same work-start action. V1 quotas: one sponsored hire / sponsor **token ID / NY day across owners / generations**; one sponsored first hire / recipient generation. Recipient: unemployed, zero completed work and XP, zero firings, no accepted shift; reject direct self-referral. New wallet is not proof of a new human; ID quota is the enforceable bound. No CRCL commission or extra budget. O1’s trading-referral system is separate; do not count it as this feature’s income. General referral attribution may be tracked without rewards.

Verify the live sponsor title, generation, owner, ownership nonce and enabled campaign nonce; bind recipient, generation and job, enforce quota before hiring, use §9.2 accepted-shift/result guards. Failure consumes no quota or eligibility. Record source, actor, both generations, day, nonce, terminal result. UI shows sponsor, quota, effect and remaining work blocks; link requests no token-spend approval. Public quota goes to first eligible on-chain claimant; clicking does not reserve it.

## 10. Metadata and application

### 10.1 State → metadata → notification

The contract changes character state. tokenURI exposes that state as metadata. ERC-4906 then tells supported marketplaces to reload it; it is not the state writer, renderer or randomizer.

- **INACTIVE:** tokenURI resolves the neutral escrow placeholder; no retired traits or earning eligibility.
- **ASSIGNED:** It resolves the new generation and frozen assignment, clearly marked pending delivery. No job action, shop use or income until ACTIVE.
- **ACTIVE:** It resolves that generation’s current career, health, work and cosmetics. Ordinary transfers preserve birth seed, both titles, career progress and generation.
- **Unwrap:** It returns to the inactive placeholder. A later wrap commits a fresh generation; it never restores the old JSON as current metadata.

NFT #42 can return as an Intern and later leave escrow as a newly drawn CEO. Both are the same ERC-721 ID with different character generations. An ordinary sale of the CEO preserves that character. A wrap does not promise the requester their previously returned ID.

### 10.2 Pre-generated IPFS catalog and dynamic metadata

Expose these current-generation fields:

- **Identity:** Generation, birth art and base traits, Birth Expression, Initial Stress, Birth Title, current Job Title and track.
- **Career and health:** employmentStatus (UNEMPLOYED / EMPLOYED / SELF_EMPLOYED), employer or role, base/effective hire and strike odds, experienceBonusBps, Stress and Mood, strikes, XP, careerStreak, promotionProgress, next title/threshold, progressionConfigVersion, Times Fired and pending performance count.
- **Work:** Readiness, custody and work state, shift and recovery end, weekend permission, latest JUICE dose and effective reset time.
- **Appearance:** Wardrobe, equipped slots, relationship, support perk, metadataRevision and appearanceRevision.

Birth seed and Birth Title remain fixed for that generation; current Job Title follows authenticated progression in §3.3. Never publish unresolved lottery outcomes. On-chain state controls gameplay and pay; external JSON cannot grant either.

**metadataRevision** advances once for each committed change to visible JSON, including lifecycle and time-derived status refreshes. **appearanceRevision** advances only when the selected pre-generated image option changes. Initialize both to 1 at assignment; later changes increment as specified. Scope both to generationId. A health-only update changes JSON, not artwork. Image availability can change displayed metadata status without changing the selected option.

Catalog target: about 100,000 pre-generated base NFT metadata options, each linked to a pre-generated image on IPFS. These are reusable options, not 100,000 NFT IDs or a finite mint allocation. Use birthTier/expression/variantId.png and matching static option metadata; zero-padded variantId identifies artwork, never tokenId or generationId. A committed manifest maps each option to Birth Title tier, expression, base-character identity, static traits, image CID, option-metadata CID, content hashes and supported cosmetic variants. IPFS CIDs are immutable and must be pinned; IPFS addressing alone does not guarantee availability. Validate every supported tier/expression branch, every listed CID and trait/image match before activating a catalog version. File counts do not set rarity.

**Selection:** JOB_TITLE uses the tier weights; EXPRESSION independently uses four equal 25% weights; ART_VARIANT chooses uniformly among eligible base options in the selected branch with equal weights for all variants in that branch. Reuse is allowed with replacement. Insufficient branch coverage blocks catalog activation; never fall back to another tier/expression, change odds or ask a live image generator. A large option library does not guarantee unique art across 10,000 IDs. Cosmetic menus expose only pre-generated compatible appearance variants that preserve all unchanged birth traits, Birth Title and expression profile. Promotions overlay current Job Title in live JSON; they do not require a different base-art branch. Purchased variants select existing image CIDs; no runtime image rendering, compositing or AI generation.

**Live metadata:** A catalog option is static base metadata. Current Job Title, career progress, employment, stress, work, wardrobe and revisions still change. Build current JSON deterministically from the selected option plus authenticated on-chain state; use versioned tokenURI JSON or publish a new immutable JSON CID for each revision, then emit ERC-4906. Never edit an IPFS file in place or point permanently at base JSON that omits live state. JSON synthesis is permitted; runtime artwork generation is not. The metadata publisher can reference only the committed image CID for the active generation/appearance revision; reject stale, conflicting or unlisted references. Identical retries are no-ops. If IPFS or metadata serving fails, retain the selected CID and show an explicit unavailable status or neutral placeholder; never redraw. Same-generation old-image fallback must be labelled stale; never show a retired generation. Principal exits and funded claims remain independent of availability.

### 10.3 Write authority and ERC-4906

- **HybridVault:** Sole entry point for assignment, activation and retirement under §2. It cannot pick or replace verified outcomes.
- **Employment and work modules:** Write only their authorized current-generation career, health, timing and work fields; validated progression may update current Job Title but never Birth Title, track or birth art. Old results use retired bookkeeping.
- **ItemShop:** Writes only paid or owned-item changes, permitted perks and consume actions; verifies the generation, nonce and burn first.
- **Metadata publisher:** Reads committed state and pre-generated catalog options; publishes current JSON and only the selected committed image reference. No image generation or general trait setter.
- **Refresh caller:** Anyone may request a bounded status refresh. It derives due changes from on-chain state and time; the caller supplies no replacement metadata or traits.

Commit state and revision changes atomically, then have JerkzNFT emit the standard MetadataUpdate(tokenId). Emit when assignment, activation, retirement, jobs, health, work, items, perks or image readiness change visible JSON. Use BatchMetadataUpdate only for appropriate consecutive IDs. Preserve the standard event signatures and supportsInterface(0x49064906). Initial mint emits standard ERC-721 mint events; hybrid swaps emit existing-ID transfers and metadata updates, never mint/burn events. [ERC-4906](https://eips.ethereum.org/EIPS/eip-4906)

Time passing creates no transaction or event. refreshMetadata materializes due visible status without changing accepted work, frozen risk or settled pay; unchanged refresh is a no-op. A supported marketplace reloads tokenURI after the event, subject to its own indexing and chain support. The JERKZ app reads on-chain state directly and shows IPFS/marketplace lag separately. No instant OpenSea refresh promise. [OpenSea refresh](https://docs.opensea.io/docs/updating-metadata)

### 10.4 Required app views

- **Portfolio:** Permanent NFT ID, current generation and revisions, custody, principal, readiness, old claims and items; separate live state from pending images or stale marketplace display.
- **Work:** One primary action, **Put your Jerk to work**, for employed and unemployed workers. Auto-select a compatible job, expose duration and the next funded start, and handle hiring/result processing without an Apply, Accept Job or second Clock In step. Show employment, XP, effective hire odds separately from birth rarity, conditional Payroll odds, No Work Found risk, firing risk, any reapply delay and optional paid/sponsored hire. Wallet approvals remain explicit; one game action does not promise one blockchain transaction. VC gets its passive Payroll view.
- **HR:** Current and projected stress, mistake risk, strikes, clean streak and leave forecast. Show Birth Title, current Job Title, career streak and next promotion with exact progress; distinguish health/clean-work streaks from career progress.
- **Conversion:** **Mint / Burn** quotes from §2; balances, current inventory, permanent reset loss, retained claims, 2-USDC entry fee and zero exit fee. Show request, randomness, assignment and delivery separately; retries retain the assignment.
- **Work detail:** Timing from §4, funded budget, frozen employment/XP/hire gate, weights, total zero-pay probability and estimated award ranges; integrated under the primary Work action, not a second work-start screen.
- **Shop:** Exact traits and replaced slots, entitlement, price and burn, JUICE reset and pass, spouse effects, possible new cooldowns and remaining work blockers.
- **Payroll:** Actual receipts, unconverted fees, reserves, committed and owed stock, seed mode and schedule, funded budgets, worker dates, four-start group allocations, pending results and claims. Show VC rates, credits and ownership effects separately.

Historical pay displays include dates, weights and dated prices. Zero-income days are valid data. Do not promise APY, minimum USD pay or a fixed stock value. Cosmetics have no hidden pay factor.

No unrevealed future rights, markers or teasers belong in launch metadata, UI, marketing or the deck. Future-product disclosure is separate; unlaunched income cannot fund baseline Payroll.

### 10.5 Read API and indexer

Provide paginated reads:

- **Configuration:** /config, /calendar, /treasury, /quotes.
- **Jobs and characters:** /jobs, /nfts/{id}/character, /nfts/{id}/employment, /nfts/{id}/work-state. Character reads return lifecycle, generation, Birth Title, current Job Title, careerStreak, promotionProgress, next threshold and progression version, tier/expression weights, catalog option, metadata/image-reference revisions, IPFS CIDs and availability status.
- **Work and claims:** /epochs/{day}, /cohorts/{cohortId}, /wallets/{address}/shifts, /wallets/{address}/claims.
- **Conversion:** /requests/{id}, /inventory; include reserved/staged inventory, assignment and delivery state. Never list assigned NFTs as available stock.

Every quote includes chainId, asset addresses and decimals, raw amounts as decimal strings, block number and hash, timestamp and expiry, configuration, inventory and calendar versions, NY day IDs, exact start and end timestamps, and minimum outputs. On-chain state authorizes actions. V1 quote expiry is the earlier of five elapsed minutes after its referenced chain timestamp or the selected start cutoff; a transaction at the cutoff is too late. Non-work quotes use five minutes. Recheck before signing; never use the browser clock as authority.

Index by chainId + transactionHash + logIndex. Retain canonical block hashes; roll back orphaned blocks and replay deterministically. Key current characters by chain, contract, token ID and generation; cache JSON and images by their separate revisions. Apply canonical updates in order, reject stale publication and invalidate caches on retirement. Keep retired claims separate. Neither missing metadata/IPFS content nor an ERC-4906 event alone proves character assignment or payment. The indexer cannot assign awards, restore characters or redirect pay.

### 10.6 Circle platform integration

**Selected scope; implementation pending.** Build Arc and native USDC payments first, with Circle user-controlled Wallets for onboarding. Circle Wallets, CCTP multichain onboarding and Gateway are selected product integrations. Let the player connect a supported wallet, detect its network, choose an amount and approve funding; the application handles the supported route to Arc. Target broad CCTP network coverage through tested adapters, starting with Base, Ethereum and Solana, followed by Gateway for a reusable cross-chain USDC balance across Arcadium products. The same wallet can use JERKZ and Gacha, but each product retains its own approvals, custody and accounting. Gacha is not a JERKZ launch dependency. Supported provider functionality is not evidence that Arcadium has integrated it.

**Environment evidence:** The Circle compatibility pages checked on 2026-09-09 list Arc Testnet for Wallets, CCTP and Gateway. Target that environment for the first working demonstration. Production requires fresh confirmation of the exact Arc network, deployed Circle contracts, supported wallet/signature types, service terms and asset routes. Never send mainnet funds to a testnet or label faucet assets as production volume. [Wallets compatibility](https://developers.circle.com/wallets/supported-blockchains), [CCTP compatibility](https://developers.circle.com/cctp/concepts/supported-chains-and-domains), [Gateway compatibility](https://developers.circle.com/gateway/references/supported-blockchains)

**Wallet onboarding:** Use a Circle user-controlled smart contract account with supported email authentication and explicit user authorization. Also permit compatible existing EVM wallets and a Solana wallet connector for source funding. A Solana source wallet and an Arc destination wallet are distinct accounts; bind both explicitly to the funding intent and never derive the destination from the source address. Circle’s Solana Wallets API support does not by itself establish support for CCTP program execution; use and test an appropriate Solana transaction-signing adapter. Bind the authenticated session to its wallet and chain; server API credentials never reach the client. Exercise recovery, expired challenges, rejected signatures and duplicate requests. A shared Arcadium login grants no permission to spend, move an NFT, choose a prize or redirect a claim. Validate ERC-721 receipt, contract calls and contract-wallet signatures before selecting the production account configuration. [Circle wallet onboarding](https://developers.circle.com/wallets/user-controlled/build-a-wallet-app)

**USDC payments and gas:** Keep the 2-USDC fee per hybrid draw, free unwrap, CRCL Payroll and JERKZ item burns unchanged. Gacha purchases and depositor claims use USDC under that product’s rules. Use six-decimal ERC-20 USDC amounts for application payments. Arc’s native gas interface uses 18 decimals over the same balance; never add the two representations or count them as separate assets. Keep enough unencumbered USDC for gas, including after a planned payment. Payment and protected-liability contracts reject unintended native-value transfers through payable application methods and provide no path that spends reserved USDC on gas. Reconcile native and token-interface movements without duplicating receipts. [Arc stablecoin accounting](https://docs.arc.io/arc/concepts/stablecoin-native-model)

**First-use gas support:** Pilot Circle Gas Station for a bounded set of onboarding calls from Circle smart contract wallets. Set allowed contracts/methods, per-wallet limits, a total campaign budget and an immediate sponsorship stop. Charge sponsorship to operations; never to redemption reserves, pending payments, prize backing or Payroll. Show when the player must pay gas after the allowance ends. Sponsorship is conditional on verified account/network support and billing. Arc already uses USDC for native gas; a separate Paymaster integration is not required merely to pay gas in USDC. [Gas Station](https://developers.circle.com/wallets/gas-station)

**CCTP funding:** The first acceptance set is Base, Ethereum and Solana funding into Arc, using matching test networks in the demonstration. Broader support is driven by the verified route catalog below; source-chain support never proves destination-mainnet availability. Native Arc USDC takes the direct payment path without a bridge. Use native USDC burn, attestation and destination mint; do not introduce a wrapped-USDC asset. Bind the source/destination domains, token, recipient, amount, maximum fees and transfer identifier. Expose source confirmation, attestation pending and destination receipt separately. Use Fast Transfer only on supported source routes with an accepted fee quote; never promise one timing for every route. Credit funds only after verified destination settlement. A delayed transfer cannot reserve a work start, lock a prize or create a Payroll entitlement. Retrying delivery must not burn or credit twice. Keep destination completion retryable; do not promise cancellation or a source refund after an irreversible burn. Once funds arrive, obtain a fresh product quote and user approval.

**Network detection and routing:** Read the connected EVM provider’s chain ID or the Solana connection’s verified cluster/genesis identity; never infer the network from a wallet brand or an address alone. Read balances only for the connected/authorized accounts on enabled networks. The user can select a different funded network. Before signing, show the chosen source account/network, exact native-USDC contract or mint, amount, source gas, provider fees, expected destination amount and the bound Arc recipient. Recheck the account and network after any provider change and before authorization. Detection selects a proposed route, not permission to move money. Resume a pending transfer from its persisted intent and chain receipts after a tab closes; no second burn on reconnect.

**Supported-route catalog:** Maintain a versioned allowlist containing source/destination chain and environment IDs, CCTP domains, official USDC token/mint identifiers, protocol version/programs, wallet/signing adapter, fee and finality capabilities, destination completion method and verified test evidence. Enable additional CCTP networks after those checks pass; do not turn on a route solely because its name appears in a provider list. CCTP’s published list checked on 2026-09-09 includes Base, Ethereum and Solana, but does not list Robinhood Chain; Arc is listed as testnet. Treat Robinhood Chain as unavailable for direct CCTP until official support and our adapter are verified. A Robinhood wallet or exchange account is a different matter: its actual connected or withdrawal network decides the route. For exchange withdrawals with no connected source provider, require an explicit supported withdrawal network; there is no reliable automatic network detection. Unsupported networks must show a clear unavailable state before a deposit is requested. Offer a supported source-network choice; any future bridge to an intermediate network requires its own reviewed route and explicit quote, never a hidden substitute. [CCTP route support](https://developers.circle.com/cctp/concepts/supported-chains-and-domains)

**Supported deposit asset:** This onboarding flow moves native USDC. ETH, SOL, USDT, bridged USDC variants and arbitrary tokens are not accepted by this USDC CCTP route. A separate swap into native USDC may be offered only through a verified, user-approved quote with stated fees and minimum output. Source-chain gas may still require ETH or SOL; display and check it before the burn. Destination completion needs its own funded relayer or verified forwarding service; Arc gas sponsorship alone does not pay source-chain gas. Do not advertise unrestricted any-chain or any-token deposits.

**Gateway shared balance:** In the next stage, let users opt into a Gateway balance usable across Arcadium products and supported funding networks. Check Gateway’s own network and wallet capabilities independently of CCTP; CCTP support does not imply a usable Gateway route. When direct Gateway funding is unavailable, a supported CCTP transfer can arrive in the Arc wallet first, with any later Gateway deposit separately authorized. Gateway holds deposited USDC under its own contract rules; it does not combine game treasuries. Show wallet funds, pending deposits, established Gateway funds and product claims as separate balances. Authorize an exact transfer to the user’s bound Arc wallet, verify settlement, then authorize the product purchase. Bind signatures to the published contract/domain, destination, amount, nonce and expiry, and test the selected wallet’s ERC-1271 path. Do not create a blanket cross-game spending allowance. Show deposit confirmation time and the documented delayed trustless withdrawal route; established-balance transfer speed is not an end-to-end onboarding promise. A Gateway outage leaves direct Arc USDC funding available and cannot block funded game claims. [Gateway balance and withdrawal model](https://developers.circle.com/gateway)

**Receipts and reconciliation:** Use one application intent ID with separate source and destination transaction IDs and provider request IDs. The authoritative payment receipt is the finalized Arc contract state and transfer, not a webhook or browser success screen. Validate provider callbacks against documented authentication, persist idempotency keys and reconcile missing, duplicate and reordered callbacks against chain state. Quote USDC product price, bridge/provider costs and gas separately; never promise a fixed CRCL sale value or convert claimed CRCL without a new user-approved route. Orders that expire while funding arrives leave the arrived USDC with the user. Operations funding, user funds, hybrid backing, Gacha backing, depositor claims and Payroll must remain independently reconciled.

**Reusable integration surface:** The shared wallet/funding adapter exposes network configuration, supported assets, authorization, transfer status and receipt reconciliation. It has no award or treasury authority. Each game owns its state transitions and financial contract; another title cannot spend JERKZ backing or reuse a paid request. Deliver an independently runnable testnet example and documented event/adapter interfaces for future Arc builders. Publication of source or samples is a separate release action.

**Measured impact:** Track activated wallets, completed 2-USDC draws, Gacha USDC sales, depositor USDC claims, net inbound CCTP funding, Gateway-funded purchases, CRCL paid and returned users. Separate organic player payments from founder/team purchases, grant receipts, faucet tests and incentives. Count one economic purchase once even if it has a bridge, payment and proceeds distribution. Report gas separately; clock-ins, CRCL awards and JERKZ burns are not USDC payment volume. Deduplicate users across the two products when using an authenticated first-party account; otherwise label the metric unique wallets, not people. §13.4 defines the measurement contract.

**Later product opportunities:** Assess Gateway Nanopayments only for a real, metered x402-compatible API, compute or data supplier in the studio’s tooling. Cap spending, verify delivered service and report operations payments separately from game revenue. No agent receives player-spending authority or outcome control; the launch art remains pre-generated. EURC/StableFX or further tokenized assets require actual user demand, supported routes and a separate reviewed scope. These are optional research items, not implemented integrations or v1 dependencies. [Circle agent nanopayments](https://developers.circle.com/agent-stack/agent-nanopayments)

## 11. Engineering interfaces

### 11.1 Modules

Modules may share an audited contract only if each liability has one accounting owner and all boundaries remain enforceable.

- **JerkzNFT / CharacterRegistry:** One-time ERC-721 minting; permanent IDs, lifecycle and generation/revision state; field-scoped writers; VC ownership checkpoints; ERC-2981, ERC-165 and ERC-4906.
- **RarityConfig:** Versioned independent tier/expression probability weights and the committed pre-generated IPFS catalog. No title supply counters or quotas.
- **HybridVault:** Same-chain principal, inactive/reserved inventory, wrap requests, immutable assignments, ACTIVE delivery and unwrap. Never mints or burns either asset.
- **WorkRegistry:** Permanent token-ID / paid-date usage, physical work-interval role exclusions, generation health and passes.
- **MarketCalendar:** Versioned New York day boundaries, hourly start IDs and NextLocalDay endpoints, including DST ambiguity/gap rules.
- **EmploymentRegistry / JobBoard:** Employment states, automatic role mapping, shift-bound hiring, XP-adjusted odds and ordered career results; no separate player application flow.
- **QuestEscrow:** Validated custody, fixed beneficiary and maturity.
- **PayrollTreasury:** Receipts, ledgers, EMA, seed schedule and budgets.
- **Payroll / VCPayroll:** Worker awards and claims; VC index, credits and reserve reconciliation.
- **FeeConverter:** Allowed bounded swaps; no principal or claim-reserve access.
- **ItemShop / BurnAdapter:** Actual burns, JUICE, shop and hiring services, wardrobe, spouse and referrals.
- **RandomnessAdapter:** Bound proof verification and seed storage.
- **MetadataService / MetadataPublisher:** Deterministic current JSON from on-chain state and pre-generated option metadata; generation/revision-bound references to approved IPFS images. No runtime artwork generation, gameplay or financial authority.

- **CircleIntegrationService:** Shared user-wallet onboarding, CCTP funding, optional Gateway funding and receipt reconciliation under §10.6. Off-chain orchestration only; no custody of player keys, financial assignment, treasury spending or generic transaction-signing authority.

### Records

- **TokenSlot:** Chain, contract, permanent ID, generation counter and current-generation reference, lifecycle, ownership nonce, inventory reservation, permanent Payroll day use and sponsor-day quota.
- **Character:** ID, generation, Birth Title, current Job Title, art seed, config and payload hash; birth profile and initial stress; lifecycle, career and items; metadataRevision, appearanceRevision, catalog option ID, appearance option/hash, image CID/status; equipped and owned item IDs, relationship and perk, shop nonce.
- **NFTWorkState:** Character key; overtime, recovery, weekend use and crash; active and queued shifts; stress, burnout latch and last-accounted day; dose nonce, pending reset and effectiveAt; restriction IDs and resolved markers. Store accepted entry snapshots separately.
- **RarityConfig:** Tier weights in basis points summing to 10,000; four expression weights of 2,500 each; immutable catalog root/version, branch indexes and supported cosmetic mappings. No depleted inventory of rarity or artwork options.
- **ProgressionConfig:** Immutable employee ladder, per-rung thresholds and complete employer/title appointment mappings, XP hiring/retention parameters, versioned and committed at birth. No Founder or VC transition.
- **EmploymentState:** Character, current Job Title, Birth Title, job, employer; careerStreak, promotionProgress, progressionConfigVersion and last credited day; employmentStatus; hiring shift reference, nonce, permanent attempt day and result; result cursor and count; strikes, clean streak, XP, firings and reapply date.
- **JobPosting / HiringAttempt:** Automatic role mapping/version; shift and first-entry cohort reference; frozen title, XP, base/effective odds; NFT ID, generation, date and nonce; bound JOB_APPLICATION domain and PENDING / HIRED / NOT_FOUND result. At most one attempt per shift; all its entries reference that result. No warm-up or standalone application custody.
- **VC financial state:** Generation eligibility intervals, NFT ID and generation, checkpoint index and time, beneficiary, wallet credits, paid totals and once-only day reconciliation.
- **Shift:** NFT ID, generation, depositor, beneficiary, requested and accepted timestamps, exact start/end, kind and elapsed seconds; daily entries, cohort IDs, weights, config, custody, withdrawal and emergency status. Preserve the original shift end separately from any successor custody end.
- **Epoch:** NY day, calendar version, boundaries and H hourly starts, asset, total/work/VC budgets, committed four-start-group allocation table/hash and reconciliation totals; frozen VC count N, perVCPay, retiredUnvested and index prefix. Daily budgets own no worker lottery seed.
- **Cohort:** Funded epoch, group index, first/exclusive hourly indices, exact starts and sealAt, allocation and books/versions; accepted membership hash, canonical indices, sealed eligibility, request and seed, phases/cursors, W/Q and assigned/released totals. Its reserve is part of the epoch commitment, never an additional liability.
- **Entry:** Epoch and cohort, paid date, NFT ID, generation, original shift/end and canonical index; beneficiary and job; title, duration, track, employment snapshot, XP, effective hire/strike odds, hiring-attempt reference, stress, risk and perk/version; performance, discipline and effective weight; hiring gate, payroll label, display result, gross amount, finalizedAt, claimableAt and paid flag. No advance or protocol claim-fee fields.
- **FeeLot:** Source contract and type, unique receipt, input asset and amount, measured stock output, income day and recycled attribution.
- **ConversionRequest:** Principal and unearned fee, beneficiary and permitted recipient, batch and configuration hash, reserved capacity and provider request; frozen assigned ID, generation, weighted title/expression draws and pre-generated option; assignment, delivery and pre-request refund state.
- **Referral:** Source, actor, sponsor and recipient generations, campaign and ownership nonces, persistent sponsor ID / day quota, recipient first-hire use and terminal result.

Use uint256 money, explicit date IDs, Unix timestamps and checked ranges. Before moving funds, reject duplicate IDs, zero or unsupported recipients, wrong assets, stale configuration or generation, and expired quotes. Delayed mutations must verify their generation or take the retired-result path.

Player job, shift-entry and shop mutations require the matching ACTIVE generation. Deferred results and exits use their frozen character keys. Hybrid lifecycle, metadata and publication methods use their stated lifecycle guards. Financial claims use stored beneficiaries, not current NFT ownership or metadata.

Accept safe NFT transfers only for prepared custody operations. Unsafe orphan transfers create no entry; rescue must prove depositor entitlement and preserve protected assets. Donations create no work or redemption rights.

### 11.2 Minimum external methods

Final Solidity types and ABI must preserve these behaviors. User batches: at most 20 NFTs. Settlement calls: at most 100 entries.

#### Hybrid conversion

- **previewConvert(quantity, direction):** Return Mint / Burn labels, fixed principal, fee, versions and draw odds; Burn (unwrap) quotes also bind IDs/generations and show losses and retained claims.
- **requestNFTs(quantity, recipient, maxFee, deadline, configHash):** Mint action (technical wrap); transfer exact principal + fee and reserve existing inventory capacity; return request IDs.
- **sealConversionBatch(batchId):** Freeze intake, inventory, rarity, art and ordering; issue the single bound randomness request once.
- **processConversionBatch(batchId, maxEntries ≤100):** Assign existing IDs and fresh generations from the stored verified seed; persist cursor and assignment before delivery.
- **claimAssignedNFT(requestId, recipient):** Beneficiary completes or retries wrap; deliver the exact assignment and earn its fee once.
- **redeemNFTs(ordered tokenIds, expectedGenerations, recipient):** Burn action (technical unwrap); checkpoint VC, retire current characters, transfer existing NFTs into inventory and return exact principal atomically.
- **refundUncommittedRequest(requestId):** Beneficiary recovers principal and unearned fee only for expired, uncommitted intake before randomness issuance under §2.2; release reservations once. No refund of a revealed or assigned outcome.

#### Metadata

- **tokenURI(tokenId):** Standard ERC-721 metadata URI for its lifecycle and current revision; no state mutation or trait draw.
- **getCharacter(tokenId):** Return lifecycle, generation, both on-chain titles/traits, careerStreak, promotionProgress, next threshold and progression version, metadataRevision, appearanceRevision, catalog option, selected image CID and availability status.
- **refreshMetadata(tokenIds):** Permissionless; at most 20 IDs. Commit due visible status and emit ERC-4906 only for changed JSON. Never randomizes, consumes items, advances a generation or changes old financial snapshots.
- **publishMetadata(tokenId, expectedGeneration, expectedMetadataRevision, expectedStateHash, jsonURI, jsonHash):** Restricted publisher attaches JSON for the current ASSIGNED or ACTIVE state revision and selected pre-generated catalog image. Validate generation, state hash and catalog binding; no state, art or probability mutation. Reject stale/conflicting publications; identical retry is a no-op. Emit ERC-4906 when new current metadata becomes available. Direct state-derived tokenURI serving may implement the same contract without storing a JSON CID.

#### Jobs and career results

- **listJobs / previewEmployment(tokenId):** Read-only role mappings, status, XP, effective odds, cooldown and optional hiring shortcuts. No required job-selection call.
- **resolveHiring(shiftId):** Permissionless deterministic resolution from the stored first-cohort seed; no player confirmation, separate randomness request or new draw.
- **Application entry points:** Do not expose standalone applyForJob or hiring-confirmation mutations in this version; putToWork owns the hiring commitment.
- **applyEmploymentResults(tokenId, expectedGeneration, maxEntries):** Apply ordered discipline, XP and career credit once; update pending count, then activate any earned promotion under §3.3. Return current title and progress.
- **resign(tokenId, expectedGeneration):** Enforce §3 guards; preserve health and apply the reapplication delay where required.

#### Shifts and JUICE

- **previewShift(tokenId, requestedStartAt, shiftKind):** Return the next valid quoted hourly start, exact segment/full-shift ends, custody wait, health and hiring eligibility, employment/XP and frozen hiring/strike odds, risk, weights, rest/pass requirements, funded cohort references and expiry. A normal request cannot silently skip an unavailable next start.
- **putToWork(tokenId, expectedGeneration, startAt, shiftKind, beneficiary, configHash, optionalHire, deadline):** Validate the exact quote before cutoff and reserve all cohort entries, date/role markers and custody atomically. Accept eligible unemployed workers and commit their hiring attempt; apply any optional authorized hire before snapshots. Auto-select the compatible job. Enforce the normal next-start rule or explicit queued-successor exception. A retained clockIn ABI may delegate to this same path; it cannot require a preceding application.
- **withdrawNFT(shiftId, recipient):** Enforce maturity or the narrow emergency path.
- **consumeJerkJuice(tokenId, expectedGeneration, expectedDoseNonce, effectiveAt, optionalWeekId, maxJerkzCost, deadline):** Burn payment, reset cooldowns and optionally grant a pass. Support atomic reset-and-clock-in at the committed hourly boundary.

#### Treasury and worker Payroll

- **closeIncomeDay(dayId):** Close receipts and update EMA once.
- **publishEpoch(dayId):** Publish the permitted funded, immutable budget.
- **sealCohort / requestSeed(cohortId):** Seal the complete four-start group at sealAt; bind all group inputs and one request at or after its final start (sealAt).
- **processCohort(cohortId, maxEntries ≤100):** Resume deterministic assessment, label and award phases; reconcile only its reserved allocation.
- **previewClaims(entryIds):** Return original shift end, result status, full gross/claimable CRCL and zero protocol claim fee.
- **claimStock(ordered entryIds, recipient):** Pay eligible finalized entries once after their original shift end, with at most 20 entries and no advance path.
- **convertFees(routeId, maxInput, minStockOut, deadline):** Return measured stock output.

#### Items and appearance

- **previewItem(tokenId, itemId, action):** Return price, catalog version, slot, compatibility, ownership, perk and resulting appearance.
- **buyItem(tokenId, expectedGeneration, expectedShopNonce, itemId, catalogVersion, maxCost, deadline):** Verify the exact burn; grant or equip the selected item as applicable.
- **equipCosmetic(tokenId, expectedGeneration, expectedShopNonce, slot, ownedItemId):** Equip an owned compatible item or birth fallback.
- **clearRelationship:** Use the same caller, generation and nonce guards; remove persona and perk.

#### VC Payroll

- **previewVCPayroll(tokenId):** Return generation, eligibility, frozen cohort count, current epoch, per-VC rate, credits and ownership effects.
- **checkpointVC(tokenId):** Credit the actual owner.
- **claimVCStock(ownedTokenIds, recipient):** Checkpoint at most 20 caller-owned IDs; pay caller credits. Permit an empty list for old-owner claims.
- **closeVCDay(dayId):** Reconcile unused reserves once.

#### Hiring and executive referrals

- **enableReferralCampaign(sponsorId, expectedGeneration):** Enable a campaign. **disableReferralCampaign(sponsorId, expectedGeneration):** Disable it.
- **previewHireNow(recipientId, jobId, itemOrCampaign):** Return effect, cost or quota, and remaining blockers.
- **buyHireNow(recipientId, expectedGeneration, expectedShopNonce, jobId, itemId, catalogVersion, maxCost, deadline):** Burn payment and hire.
- **redeemExecutiveReferral(sponsorId, sponsorGeneration, recipientId, recipientGeneration, jobId, deadline):** Check quota and grant the free hire.

### 11.3 Events and errors

Custom events include the relevant NFT ID, generation, beneficiary, asset, raw amount and version, with suitable indexed fields. Standard Transfer, Approval, MetadataUpdate and BatchMetadataUpdate retain their exact standard signatures; do not add indexed fields to ERC-4906. CharacterGenerated means a new generation on an existing ID, not an ERC-721 mint.

- **Conversion:** ConversionRequested, IdentityAssigned, CharacterGenerated, CharacterRetired, NFTRedeemed.
- **Work and health:** ShiftAccepted, ShiftWithdrawn, RecoveryScheduled, JerkJuiceConsumed, CooldownsCleared, WeekendPassGranted, WeekendWorked, MondayCrashScheduled, StressUpdated, BurnoutStarted, BurnoutCleared, WorkAssessed, CalendarConfigured.
- **Employment:** HiringAttemptCommitted, HiringResolved, JobResigned, JobFired, ExperienceGranted, CareerProgressed, CareerStreakReset, JobPromoted, HireBypassed, ReferralCampaignChanged, ExecutiveReferralRedeemed.
- **Treasury and randomness:** TokensBurned, FeeReceived, FeesConverted, IncomeDayClosed, EpochFunded, CohortSealed, SeedRequested, SeedStored, CohortFinalized, EpochReconciled, StockClaimed.
- **Appearance:** ShopItemPurchased, CosmeticEquipped, RelationshipChanged, AppearanceUpdated, MetadataPublished. Include generation and metadata/image revisions; also emit ERC-4906 for changed JSON.
- **VC:** VCActivated, VCCheckpointed, VCRetired, VCPayrollClaimed, VCDayClosed.

Preserve these error categories. The UI must show the exact on-chain reason and a specific recovery action.

- **Access and configuration:** WrongChain, UnsupportedAsset, NotOwnerOrOperator, StaleConfig, StaleGeneration, CharacterInactive, UnsupportedCalendarDate.
- **Custody and conversion:** AlreadyInCustody, InsufficientReserve, InsufficientInventory, QuoteExpired, SlippageExceeded, ResultAlreadyAssigned, NotMature, TransferFailed.
- **Work and health:** DayOccupied, RecoveryRequired, WeekendPassRequired, MondayCrash, ShiftTooLong, UnfundedEpoch, UnfundedCohort, ShiftCutoffPassed, BurnoutLeaveRequired.
- **Employment:** HiringPending, ReapplyCooldown, TitleIneligible, EmploymentResultsPending. UNEMPLOYED alone is never a putToWork error.
- **Randomness and claims:** SeedAlreadyRequested, InvalidSeedProof, AlreadyClaimed, PaycheckNotFinalized, ShiftNotEnded.
- **Metadata:** UnauthorizedMetadataWriter, StaleAppearanceRevision, InvalidCatalogOption, ConflictingMetadata; use StaleGeneration and CharacterInactive where applicable.

## 12. Operations and reference evidence

**Permissionless keepers** seal and assign conversion batches, close income days, publish budgets, reconcile VC dates, seal hourly cohort lists, request group seeds after their final start closes, settle work and ordered career results, resolve shift-bound hiring, claim designated fees, convert fee lots and refresh timed metadata. Calls must be bounded and retries idempotent. Monitor backlog and fund gas separately. Hosted keepers improve availability; they cannot choose outcomes or recipients.

**Failures:** A failed randomness provider, converter or reserve check stops new affected intake. Matured NFT exits, eligible finalized claims remain callable when their required asset can transfer. An outage never permits a new seed or reuse of obligations. An IPFS/metadata/indexer outage delays display only: unwrap switches to the on-chain inactive state, and an old image cannot restore rights. Failed delivery remains retryable with its frozen assignment.

**Administration:** Immutable custody and payout cores, no proxy upgrade path. A 3-of-5 multisig controls future versioned settings through a 48-hour timelock; exact distinct signer addresses must be verified before activation. A separate 2-of-3 guardian multisig may pause new exposure immediately. Resume and any custody-fault early-exit activation require the 3-of-5 authority and the same delay, with a recorded fault scope; they cannot select beneficiaries or change financial outcomes. A limited pause can stop new exposure. It cannot confiscate funds, cancel known outcomes, alter accepted epochs or streams, erase characters outside unwrap, disable VC checkpoints or accrual, or delete claims. Item prices change by future purchase week; conversion settings change for future batches only. Rescue excludes principal, assigned NFTs, pending USDC and owed stock. Metadata authority is field-scoped (§10); no admin arbitrary-trait, old-generation restore or live-generation art-base override. New art/catalog versions affect future assignments or purchases only; a restricted publisher can publish only current state-derived JSON referencing an existing committed image. Upgrades or asset replacement require a separately reviewed migration.

### Reference projects

StonkBrokers documents activation-tier-weighted stock rewards, Clock In’s accumulated pot and transfer-cleared activation. JERKZ specifies daily budgets and fatigue that persists through a generation. No fixed return is inferred. [StonkBrokers documentation](https://www.stonkbrokers.io/docs)

Quotrons V2 documents one QUOTRON/WETH market, 2% of trading volume allocated to stock rewards, and bounded conversion epochs. Those are existing pacing controls. Its V1 post-mortem identifies stale approvals during hybrid ownership changes, not excessive emissions. No verified evidence here attributes either project’s lifespan to excessive rewards. These are published-source findings, not bytecode or performance audits. JERKZ’s budget policy is a new design with explicit scenarios. [Quotrons technical documentation](https://www.quotrons.cash/llms-full.txt), [V1 exploit mechanics](https://github.com/mavrkofficial/quotrons-v1-post-mortem/blob/main/docs/01-incident/exploit-mechanics.md)

## 13. Acceptance and delivery

### 13.1 Required verification

Verify every transition and invariant above, including the following cases.

- **Custody and principal:** Bootstrap creates exactly 10,000 IDs and permanently closes minting; no NFT burn path. Round-trip swaps preserve both asset supplies; separate item burns reduce JERKZ supply. Conversion, donations, pending requests, exhausted inventory and burns preserve reserves. Every ownership path clears stale approvals. Test exact decimals, the 2-USDC fee for single and batch draws, free exit, no double charge, and prepared versus orphan custody.
- **Characters and rarity:** Verify title weights sum to 10,000 and every categorical boundary. Draw with replacement; weights do not change with live counts, prior draws or folder sizes. Test repeated CEOs beyond 150 and VCs beyond ten, plus 0 / 1 / 10,000 eligible VCs. Test independent 25% expressions in every tier. Birth Title stays fixed; current Job Title may rise only through §3.3. Founder birth weight is 2%; all ten title weights sum to 100%. Transfers preserve state; wraps select fresh pre-generated art/title/expression options. Delivery retries retain assignment. Retirement invalidates live state in bounded work, isolates old callbacks and preserves counters, sponsor quotas, original beneficiaries, payments and day usage. Verify same-ID reuse and different-ID rewrap; quest custody never converts.
- **Progression:** Verify every threshold at −1 / exact / +1; 200 uninterrupted credited workdays from Intern to Board; promotion after discipline and backlog drain only; no retroactive pay or immunity. Separate lifetime XP from promotion credit: failed searches and notice earn XP but no promotion; employed zero-pay entries can promote; firing, resignation, notice entries, rest and retries follow §3.3. Test XP thresholds at 4/5, 99/100 and cap, 99% hiring ceiling, relative strike rounding, frozen multi-day XP, two-day credit, one ID/day, no hourly or purchased XP, no transfer reset, unwrap retirement, frozen config, complete appointment mappings and ERC-4906. Founder/VC cannot enter the ladder; born/promoted executive rights match. Simulate increasing executive populations under the fixed worker budget.
- **Metadata:** Verify the approximately 100,000-option pre-generated IPFS manifest, all supported tier/expression branches, option/image hashes and static trait matches, cosmetic compatibility and pin/availability checks. Missing art cannot change a weighted outcome. Test INACTIVE / ASSIGNED / ACTIVE URIs, generation and separate JSON/image-reference revisions, same-generation sale, neutral retirement fallback, stale/conflicting publishers, unauthorized writes, no-op refresh, time-derived JSON changes, IPFS outage, exact ERC-4906 events and no runtime image generation. Base option JSON alone cannot replace live gameplay metadata.
- **Time and work:** All four shift kinds and weights; reject 12 / 36; one token-ID / day. Check anytime Clock In, exact :30 cutoffs, group seal times, independent entry ends, exit and settlement edges; queued successors and custody boundaries; EST/EDT, 23/25-start days, repeated/missing local hours, deterministic NextLocalDay endpoints and 47/49-hour long shifts; Mondays, holidays, leap dates, unsupported dates and late keepers.
- **Health and JUICE:** Overtime, normal-work and rest streaks; projected two-day rejection; stress boundaries 29 / 30 / 59 / 60 / 79 / 80 / 100; full idle-day decay and burnout. Test spouse rounding, no stacking and frozen risk. Clear all cooldowns once; reject free repeat resets, stale doses and resets inside locks. Test weekend passes, first-use crashes, no-work / no-crash and cleared-marker replay.
- **Jobs and roles:** Correct mint employment states; one-action dispatch; automatic role mapping; no separate warm-up/application; frozen hiring odds and independent cohort domain; one attempt per shift and permanent ID/day; 48-hour shared hiring result; failed search zero award/strikes with scheduled XP; no queued unresolved search; no free retry; resignation, transfer, retirement and two-entry backlog. Separate performance and discipline; half-weight mistakes, graduated strikes, two-strike firing, clean-streak recovery, ordered results and old-award protection. Verify instant protected executives, Founder flops and VC-only streams.
- **Items, hiring and referrals:** Exact supply decrease, no remint or substitute sink, atomic rollback, preview-to-trait match, paid haircuts versus free owned-item changes. Reject duplicate, stale, incompatible and unauthorized purchases; no principal or Payroll spending. Test immutable SKUs, selected pre-generated variants and ERC-4906. Check pre-acceptance optional hire, atomic hire+work rollback, rejection of any change to accepted hiring even before seed, wrong job / owner / title, no double hire, invalid campaigns, quotas across transfers and rerolls, direct self-referral, first-hire limits and unchanged Payroll eligibility.
- **Treasury and seed:** Zero funding in both earning paths; once-daily EMA; exact split; no backdating or duplicate receipt / VC accounting; reserve conservation. Schedule actual stock; test the 75/25 split, days 1/180/181, skipped dates, one-time reserve credit and exact tranche totals; never assume a grant. Verify lookahead cannot advance income or seed maturity, and zero-income stress paths cannot touch committed or owed stock.
- **Cohorts and claims:** Exact per-cohort books/domains, committed tracks, no replacement, mixed tracks, empty/zero/one/full cohorts, all/partial failed hiring gates, cross-cohort dependencies, daily caps and dust. Prove hourly allocations sum exactly to the daily worker budget and cannot be spent twice; test 23/24/25-hour days, last-group sizes 3/4/1, tiny budgets and empty returns. Verify 8h settlement never waits for 16/24/48 work, and future 48 reservations survive earlier group finalization. Prove every group seals at least five hours before its earliest possible 8h maturity; outcomes are public before maturity, but claims and career mutation remain time-gated. Emergency withdrawal must not change W/Q, awards, dates or original maturity. Reject late membership, duplicate/reentrant claims and rerolls; preserve failed-transfer claims and independent NFT exit. A 48 first entry waits for its original shift end, a successor cannot delay earlier pay, and late results pay in full with no Friday/advance fee. Test physical tail role-exclusion markers, per-date paid quotas and exact-boundary VC transitions.
- **VC:** Variable cohorts 0 / 1 / 10 / 20 / 10,000; 95 / 5 split, per-VC floor, no fixed seats and exact dust/unvested returns. Verify denominator snapshots, scheduled eligibility and cancellations, exact-boundary retirement order, bounded rate materialization after silence, no survivor windfall within a day and no late-entry dilution. Test before/at/after 09:30, DST, mid-day and repeated tiny checkpoints, custody and failed transfers, retirement/rerolls and old-owner claims, unfunded gaps, no backdating or dual pay, repeated close, late keepers, VRF outage and conservation.
- **Circle integration:** Verify wallet ownership, recovery, rejected/expired authorization, exact 2-USDC payment and native/ERC-20 decimal conversion without double counting. Test gas after payment, sponsorship caps, recipient/domain substitution, fee/quote expiry, duplicate or missing callbacks, delayed CCTP attestation, duplicate mint attempts, Gateway deposit finality and contract-wallet signatures. No product purchase occurs before destination funds and a valid product authorization. Provider failures cannot spend protected balances or block independently callable funded claims. Reconcile demo receipts to exact chain transactions and classify test, team and user activity separately. Cover Base/Ethereum/Solana adapter paths, wrong Solana cluster or destination encoding, provider account/network changes, unsupported Robinhood Chain, exchange withdrawal selection, non-native-USDC rejection, insufficient source gas, unavailable destination completion, route disablement before burn and transfer recovery after reconnect. Existing accepted burns retain a safe completion path if new intake is disabled. Test CCTP and Gateway support matrices separately.

- **Security and scale:** Frozen inputs; duplicate, foreign and out-of-order callbacks; no cancel or reroll. Process 10,000 entries in resumable calls of at most 100; safe callback gas, no collection-wide loop. Pause, rescue and admin actions cannot change accepted work, draws or streams, or spend protected assets. Exclude unrevealed features from launch outputs.

**Implementation verification before live funds:** The reference integer checks in §6.4 validate unchanged budget arithmetic; its attendance illustration does not validate v0.20 hiring/XP outcomes. Run the revised acceptance tests and simulations; they do not validate undeployed Solidity or prove demand. Test the four-hour group partition, sparse/crowded hours, strategic cohort selection, fragmented unused funding and 48 reservations. Use 100 / 1,000 / 10,000 entrants; title, work and stress mixes; job-search downtime and paid / sponsored hiring; expression profiles, Founder variance and collector rerolls; variable VC populations, including 0, 1, 10, 20 and 10,000, with transfers and retirement; 50%/90%/100% fee declines; stock / token price changes, thin liquidity and concentrated ownership; rarity and career-health rerolls; repeat JUICE, cleared Mondays, spouse and increased participation; cosmetic demand and net burns. Report typical awards and worst obligations. Low expected pay requires clear display or a revised funded plan, never hidden liabilities. Shop demand adds no CRCL budget.

### 13.2 External launch gates : evidence, not undecided game rules

- **Launch:** Exact Arc environment; O1 factory, hook, fee contracts and claim rights; a compatible burnable token. Verify token distribution and funded NFTs without duplicate use of launch liquidity. Distinct LP entitlement is zero until verified. Verify the selected Circle wallet type, recovery and authorization path; USDC dual-interface accounting; supported CCTP/Gateway routes, contracts, fees and service conditions. Product checkboxes in a grant application require actual code and transaction evidence, not a design statement.
- **Assets:** JERKZ, JerkzNFT, HybridVault, CRCL and USDC addresses; fungible decimals, issuer restrictions, corporate actions, acquisition and sale routes. Verify one-time NFT bootstrap and permanent conversion rate.
- **Metadata:** Pre-generated IPFS images/options and committed manifests, approximately 100,000 base options, full tier/expression coverage, compatible cosmetics, inactive/pending assets, pinning/availability evidence, reproducible JSON mapping, URI policy, publisher identity and field permissions; ERC-165 / ERC-721 / ERC-4906 support and actual Arc marketplace indexing.
- **Randomness and operations:** Arc VRF proof validation, confirmations, callback limits and operating funding. Verify route liquidity, freshness, deviation, slippage, size and gas limits.
- **Control and funding:** Named signers, pause role and delay; actual grant and stock receipts; no receipt is inferred from the request. MarketCalendar initially supports Payroll dates 2026-01-01 through 2035-12-31, with all required end/recovery lookups inside the supported table; later extensions require a new verified future-calendar version. Publish the IANA timezone-data version and full table hash used to build it.
- **Selected balance, not an open design choice:** Verify implementation of independent title weights, equal expression weights, catalog coverage and reuse, variable VC dilution, reroll economics, single-action work, XP-adjusted hiring and discipline, stress and performance, shop prices, spouse and holiday rules, four-hour group attendance/partitioning, end-of-shift claims, the selected 20/60/19/1 employee and 70/20/8/2 Founder books, 60/40 award split, two-strike discipline without direct No Pay firing and the 5% VC share. Independently review VC ownership-income contracts before live funds.

Arc and CRCL are settled product choices. Missing integration evidence does not reopen Robinhood or JERKZ payouts.

### 13.3 Implementation order

1. Establish §10.6’s Circle user-controlled wallet and Arc Testnet USDC payment demonstration, with receipt evidence and safe repeat/reject behavior. Build and test §2’s bidirectional hybrid token wrapper, backing, generation guards and separated randomness domains against mocks. Include CharacterRegistry, RarityConfig, WorkRegistry, MarketCalendar, JobBoard and EmploymentRegistry.
2. Implement fee and stock accounting, worker and VC budgets, VC index and checkpoints, zero / optional-seed pacing, four-hour funding groups with hourly starts, cohort lotteries and end-of-shift claims. Reproduce reference arithmetic in Solidity tests.
3. Build job, work, health, weekend and shop UI; generation wardrobe/spouse state; art, metadata, scoped publisher and indexer. Test delayed settlement and IPFS/metadata outages; preserve independent exits and financial rights.
4. Verify Arc integrations, add the selected CCTP funding routes and exercise provider failures in a limited test deployment. Add Gateway for the shared Arcadium balance in the separate Gacha stage; direct Arc funding keeps JERKZ independent. Demonstrate supported test assets with explicit test labels and measurable receipts.
5. Complete independent contract, economic, issuer and jurisdiction review for paid random outcomes and stock Payroll before real-money launch.

### 13.4 Delivery evidence and adoption measures

**Evidence ladder:** An architecture choice is planned; runnable integration code with a passing test is implemented; a verified transaction on the named network is demonstrated; release evidence and real users establish production use. Record each state separately. The existing art tools and proposal are shipped creative/product work, not proof of a live game, Circle integration, revenue or retention.

**Milestone sequence:** First deliver wallet onboarding and an exact USDC payment on Arc Testnet. Next deliver JERKZ’s funded work/redemption cycle and the selected CCTP funding route. Then connect the separate Gacha product through the shared wallet and Gateway balance. Before live assets, complete independent security and asset-eligibility review. After a bounded launch, evaluate retention and net revenue before expanding paid acquisition, inventory or another game. Target dates and funding releases belong to the grant agreement; this sequence does not assume receipt of the requested $100,000.

**Activation and retention:** Activation is a wallet/account completing its first successful product action after funding, not wallet creation alone. Report new signups, wallet creation, funding completion and first-action completion separately. D7 and D30 retention use the initial activated cohort and a defined return-action window; include all eligible cohort members in the denominator, not only those who return. A return action must be a completed game action, not a page load or automated keeper call. Separate repeat paid purchase from repeat free gameplay. Use UTC reporting periods, with an explicit mapping to New York Payroll dates.

**Payment volume and revenue:** Completed product payment volume equals accepted hybrid-draw fees plus delivered Gacha purchase prices, less refunds/reversals under their applicable rules. Do not add bridge deposits, gas or redistributions to that total. Report USDC depositor proceeds and collection royalties as allocations of sales, not additional sales. Net studio revenue is only revenue assigned to the studio after refunds, royalties, depositor liabilities and directly attributable service costs; do not count principal, grants, team capital, RWA inventory or total Gacha sales as profit. Preserve any unassigned service-fee allocation. Report CRCL in token units and, only with a dated source, an indicative value. Never call a stock payout a USDC transfer.

**Liquidity and ecosystem:** Report unique externally funded wallets, inbound less outbound USDC by route, completed CCTP transfers, Gateway-funded buyers, average funded balance, spend conversion and cost/time to first action. Keep user balances, unspent grants, protected backing and treasury free reserves separate; do not sum overlapping balances into AUM. Report audited adapters, independent integrations and actual downstream usage separately from repositories, downloads or discussions. Source code and transaction evidence must make the measurements reproducible without exposing personal identifiers or wallet secrets.

**Grant demonstration:** Provide a video of at most five minutes showing the actual Circle integration code, an authorized Arc Testnet USDC flow, its transaction/receipt and failure handling. Label unfinished CCTP, Gateway and game features as planned. A deck, mock transfer or local simulation cannot substantiate a claim of completed Circle integration. Production traction and externally verified founder results require their own evidence.

## 14. V2: the on-chain Monopoly

V2 extends the office-life economy into property ownership.

- Own land, apartments, houses and offices.
- Spend JERKZ on furniture, renovations, permits, maintenance and status upgrades.
- Add neighborhoods, businesses, lease and rent gameplay, and offices that host jobs and social activity.
- Explore mortgages, city events and corporate competition in a separate V2 design.

Before property sales or income claims, define ownership, supply, pricing, rent and payment sources, custody, transfers and character retirement. Do not assume unwrap deletes property. Never promise rent from uncreated assets. VC Payroll and executive protection are MVP; property gameplay and separate gacha remain outside launch claims.
