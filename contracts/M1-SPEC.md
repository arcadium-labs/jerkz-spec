# Milestone 1: Hybrid Wrapper, Characters, Rarity, Calendar

Status: ready to implement. Written 2026-09-09 against Work2Earn spec v0.15 and the Arcadium deck (jerkz.work); the deck governs product scope.
Companions: `SPEC.md` (escrow v2, superseded by this for the swap), `GACHA-SPEC.md` v0.2 (depositor gacha, built after M1), `ENTROPY-SPEC.md` (randomness switch).

## 1. Scope and decisions

Milestone 1 is spec §13.3 step one, split in two halves. This document is the first half:

| Module | Spec §11.1 name | Status after M1a |
|---|---|---|
| `JerkzNFT` | JerkzNFT / CharacterRegistry | New. Replaces `HybridCollection`. |
| `RarityConfig` | RarityConfig | New. Versioned weights + catalog descriptor. |
| `HybridVault` | HybridVault / "404" wrapper | New. Replaces `HybridEscrow`. Reusable: any ERC-20, any principal, any collection. JERKZ is instance one; RWA packs (deck slide 15) are further instances. |
| `PackCollection` | Arcadium Pack NFT | New, small. Capped ERC-721 with a fixed denomination, no character logic. |
| `MarketCalendar` | MarketCalendar | New. Pure library plus a versioned config contract. |
| `RandomnessAdapter` | RandomnessAdapter | Existing `IRandomnessSource` + operator source; domain separation added here. |
| `JERKZ` | JERKZ token | Existing `HybridToken` plus `ERC20Burnable`. |

M1b (WorkRegistry, JobBoard, EmploymentRegistry) follows as its own document once this lands; it depends on the character record and calendar defined here.

Decisions already taken with the product owner, recorded so the spec text and the code do not drift:

1. **One-transaction wrap.** The randomness callback assigns the character and pushes the NFT to the recipient. `claimAssignedNFT` exists only as the retry path when the push fails. Spec §2.1's "Complete wrap" step happens inside the callback.
2. **Per-request randomness.** One randomness request per `requestNFTs` call. The batch fields from spec §2.1 are kept in storage with batch size one, so 60-second batching can be enabled later without a migration.
3. **No title caps.** Titles and expressions are drawn with replacement from weights. Art variants are drawn uniformly within the tier/expression branch, with replacement.
4. **Configurable catalog.** Branch counts, IPFS root, manifest hash and base-identity count are inputs, per AKLO. Testnet ships a placeholder catalog served by the UI.
5. **Gacha v0.1 stays live on testnet until M1c.** NFTs it hands out are ACTIVE characters and keep their generation on transfer; only the wrapper changes a generation.
6. **Timeout refund is source-specific.** Kept while the operator commit-reveal source is live, disabled for a VRF, matching spec §2.2.
7. **The vault is the deck's reusable "404" protocol.** One implementation, deployed once per (token, principal, collection). JERKZ uses the character assignment policy; RWA packs use the deterministic policy. See §5.0.
8. **Gacha v2 follows M1.** The deck's depositor gacha (`GACHA-SPEC.md` v0.2) needs `JerkzNFT.currentTitle` for its executive gate, so it is built once JerkzNFT exists.

## 2. Invariants (spec §2)

With `r = 100_000e18`, `N` = ACTIVE NFTs outside the vault (including quest and third-party custody), `P` = pending wraps plus assigned-but-undelivered NFTs:

- **V1** `token.balanceOf(vault) ≥ r × (N + P)`. Principal is never spendable by anything but unwrap and pre-issuance refund.
- **V2** `N + P ≤ 10_000`, and `inventory ≥ P` where inventory = INACTIVE NFTs held by the vault.
- **V3** NFT supply is exactly 10,000 forever: bootstrap mints them into the vault, then `bootstrapClosed` is set and there is no mint or burn path.
- **V4** Every pending fee has one beneficiary and one terminal state: earned on delivery, or refunded pre-issuance. `earnedFees` is the only withdrawable USDC.
- **V5** Every wrap increments the token's generation exactly once; retirement makes the old generation unreachable as current state.
- **V6** Only the vault can move a token that is INACTIVE or ASSIGNED.

## 3. `RarityConfig`

Versioned, append-only. Version 0 is invalid. Activation validates; the vault snapshots the active version into each request.

```solidity
enum Title { Intern, Junior, Specialist, Manager, Director, CSuite, CEO, Board, VC, Founder } // 10
enum Expression { Happy, Neutral, Crying, Manic }                                              // 4

struct Catalog {
    uint16[10] titleWeightsBps;       // sum == 10_000
    uint16[4]  expressionWeightsBps;  // sum == 10_000; 2_500 each by default
    uint32[10][4] variantCount;       // per tier × expression branch, all > 0 to activate
    uint32 baseIdentityCount;         // informational, from the manifest
    string ipfsRoot;                  // e.g. ipfs://<cid>/ ; images at <root><tier>/<expression>/<variantId>.png
    bytes32 manifestHash;             // keccak256 of the committed manifest file
    uint8[4] initialStress;           // Happy 0, Neutral 20, Crying 45, Manic 60
}
```

- `propose(Catalog) → version` then `activate(version)` after a configurable delay (48h on mainnet, 0 on testnet). Activation requires every branch count > 0 and both weight arrays summing to 10,000.
- `active()` returns the current version; `get(version)` returns a frozen catalog. Old versions never change.
- Proposed v1 weights from spec §3.1: Intern 4000, Junior 2300, Specialist 1500, Manager 1000, Director 500, C-suite 300, CEO 150, Board 40, VC 10, Founder 200.
- Testnet placeholder: `variantCount` all 3, `ipfsRoot` pointing at the UI's `/catalog/` route, manifest hash of the placeholder manifest.

## 4. `JerkzNFT`

ERC-721 with ERC-165, ERC-2981 (2% default, owner-settable), ERC-4906, `Ownable2Step`. No `ERC721Burnable`.

**Bootstrap.** `mintBootstrap(uint256 quantity)` owner-only, mints sequential ids into the vault in batches of 200 (Arc's 16.7M estimate cap), until `totalMinted == 10_000`, then `closeBootstrap()` sets `bootstrapClosed` and disables minting permanently. Minted tokens are INACTIVE with generation 0.

**Lifecycle and character**

```solidity
enum Lifecycle { INACTIVE, ASSIGNED, ACTIVE }

struct Character {
    uint32 generation;
    Lifecycle lifecycle;
    Title birthTitle;
    Title currentTitle;     // == birthTitle at birth; EmploymentRegistry may raise it (M1b)
    Expression expression;
    uint32 variantId;
    uint32 catalogVersion;
    uint8 initialStress;
    uint32 metadataRevision;
    uint32 appearanceRevision;
    uint64 activatedAt;     // 0 until ACTIVE; VC eligibility uses next calendar boundary after this
}
```

`character(tokenId)` returns the current generation's record. `ownershipNonce(tokenId)` increments on every transfer (spec §9.3).

**Writers.** Role-scoped, set by the owner: `VAULT` (assign, activate, retire), `EMPLOYMENT` (currentTitle, M1b), `WORK` (health fields, M1b), `SHOP` (items, M3), `PUBLISHER` (image status, M3). M1a wires only `VAULT`. Every writer call bumps `metadataRevision` and emits `MetadataUpdate(tokenId)`.

- `assign(tokenId, Title birth, Expression, variantId, catalogVersion)`: requires INACTIVE and `msg.sender == VAULT`. Increments generation, writes the record, sets ASSIGNED, revisions to 1. Emits `CharacterGenerated(tokenId, generation, ...)`.
- `activate(tokenId)`: ASSIGNED → ACTIVE, sets `activatedAt`.
- `retire(tokenId)`: ACTIVE → INACTIVE. Clears the current record reference (bounded, no loops). Emits `CharacterRetired`. Generation-keyed history stays readable via `characterAt(tokenId, generation)`.

**Transfers.** `_update` override: reverts unless the token is ACTIVE or `msg.sender == VAULT`; increments `ownershipNonce`; if `vcPayroll != address(0)` calls `IVCPayroll.checkpoint(tokenId)` before the move (M2, interface stubbed now). OpenZeppelin already clears the per-token approval on transfer.

**Metadata.** `tokenURI(tokenId) = metadataBase + tokenId`. The MetadataService builds live JSON from on-chain state plus the catalog option and returns the image URL `<ipfsRoot><tier>/<expression>/<variantId>.png`. INACTIVE resolves to the escrow placeholder, ASSIGNED to the frozen assignment marked pending delivery. No JSON CIDs are stored on-chain in M1; spec §11.2 permits state-derived serving.

## 5. `HybridVault`

`Ownable2Step`, `Pausable` (blocks new requests only), `ReentrancyGuard`, `IERC721Receiver`, `IRandomnessConsumer`.

### 5.0 One wrapper, many instances

The deck sells the wrapper as shared infrastructure: "tokens → NFT pack → gacha prize → tokens" for JERKZ, CRCL packs, NVDA packs and other launch RWAs. So the vault is parameterized at construction and everything JERKZ-specific lives behind an assignment policy:

```solidity
constructor(IERC20 token, uint256 principal, IERC721 collection, IAssignmentPolicy policy, uint256 wrapFee, address treasury, address owner)
```

| Instance | token / principal | collection | policy | randomness |
|---|---|---|---|---|
| JERKZ | JERKZ, 100,000 | `JerkzNFT` | `CharacterPolicy`: draws ID, title, expression, variant from `RarityConfig` | required |
| CRCL pack (later) | CRCL, denomination | `PackCollection` | `SequentialPolicy`: next inventory ID, no traits | none |
| Any approved RWA | that token, denomination | its `PackCollection` | `SequentialPolicy` | none |

`IAssignmentPolicy` has two functions: `needsRandomness()` and `assign(vault, requestId, i, word) → tokenId`. With a policy that needs no randomness, `requestNFTs` assigns and delivers in the same transaction. Everything else in this section, principal accounting, reservations, fees, refunds, redeem, is shared and tested once.

**Underlying-asset eligibility.** RWA tokens on Arc are FiatToken-style with blacklists and pause (verified on cirBTC). The vault checks `paused()` and `isBlacklisted(vault)` where the token exposes them before accepting principal, and a wrap or redeem that hits a transfer restriction reverts with `UnderlyingTransferRestricted` rather than leaving a half state. Rebasing and fee-on-transfer tokens are rejected by the balance-delta check.

**`PackCollection`.** Capped ERC-721 minted into its vault at bootstrap like JerkzNFT, `tokenURI` per denomination (all packs of a denomination share art), ERC-2981, ERC-4906 for the INACTIVE/ACTIVE flip. No generations, titles or roles.

**Storage**

```solidity
uint256 constant PRINCIPAL = 100_000e18;
uint256 public wrapFee = 2e18;                 // native USDC, 18 decimals on Arc
address public source;                         // IRandomnessSource
uint64  public refundDeadline;                 // 0 = disabled (VRF); 1h while on the operator source

uint256[] private _inventory;                  // INACTIVE token ids held; swap-remove
uint256 public reserved;                       // P: pending qty + assigned-undelivered
uint256 public earnedFees;

enum RequestStatus { None, Pending, Assigned, Delivered, Refunded }
struct Request {
    address beneficiary; address recipient;
    uint16 quantity; uint16 delivered;
    uint32 catalogVersion; uint64 requestedAt;
    uint96 fee; RequestStatus status;
    bytes32 randomnessId; address source;
    uint256[] tokenIds;                        // filled at assignment
}
```

**`requestNFTs(quantity ≤ 20, recipient, maxFee, deadline, configHash) payable → requestId`**

1. Not paused; `block.timestamp ≤ deadline`; `configHash == keccak(rarity.active(), wrapFee, PRINCIPAL)` (spec's stale-config guard); `msg.value == wrapFee × quantity ≤ maxFee`.
2. `inventory − reserved ≥ quantity`, else `InsufficientInventory`.
3. Pull `PRINCIPAL × quantity` JERKZ from the caller (`safeTransferFrom` with balance-delta check). `requestNFTsWithPermit` variant uses the token's EIP-2612 permit so the whole wrap is one transaction.
4. `reserved += quantity`; store the request as Pending with the active catalog version; `randomnessId = source.requestRandomness(requestId, clientSeed)`.
5. Emit `ConversionRequested(requestId, beneficiary, quantity, fee, catalogVersion)`.

**`fulfillRandomness(randomnessId, word)`** — source only, request must be Pending.

For `i` in `0..quantity`:
1. **TOKEN_SLOT**: `idx = sample(word, "INVENTORY", i, inventory.length)`; swap-remove the id from `_inventory`.
2. **JOB_TITLE**: `t = weighted(word, "TITLE", i, titleWeightsBps)`.
3. **EXPRESSION**: `e = weighted(word, "EXPRESSION", i, expressionWeightsBps)`.
4. **ART_VARIANT**: `v = sample(word, "VARIANT", i, variantCount[t][e])`.
5. `nft.assign(id, t, e, v, catalogVersion)`; emit `IdentityAssigned(requestId, id, generation)`.

Then status Assigned, and immediately `_deliver(requestId)`.

**`_deliver(requestId)`**: for each undelivered id, `nft.activate(id)` then `nft.transferFrom(vault, recipient, id)` inside try/catch. On success `delivered++`, `reserved--`, `earnedFees += fee/quantity`. On the first failure stop and leave the request Assigned; emit `DeliveryDeferred`. When all delivered: status Delivered, emit `NFTDelivered`. Assigned tokens stay ASSIGNED inside the vault; V6 stops anyone else moving them.

**`claimAssignedNFT(requestId, recipient)`**: beneficiary only; may change `recipient`; reruns `_deliver`. No new seed, generation or fee.

**`redeemNFTs(tokenIds ≤ 20, expectedGenerations, recipient)`**: caller owns or is approved for each; each is ACTIVE with the expected generation. For each: `nft.retire(id)` (the VC checkpoint runs inside the NFT's transfer hook in M2), `transferFrom(caller, vault, id)`, push to `_inventory`, then `token.safeTransfer(recipient, PRINCIPAL × n)` once. Emit `NFTRedeemed` per id and `CharacterRetired` from the NFT. Free; no randomness.

**`refundUncommittedRequest(requestId)`**: beneficiary only. Allowed if the source has not issued (a VRF request that reverted at issuance) or, only when `refundDeadline > 0`, when Pending for longer than the deadline. Returns principal and fee, `reserved -= quantity`, cancels on the source. Never after assignment.

**Owner**: `setWrapFee` (applies to future requests; `configHash` protects users), `setSource`, `setRefundDeadline`, `pause/unpause`, `withdrawEarnedFees(to)`, `rescue` for foreign assets only. There is no principal withdrawal and no inventory withdrawal.

**Sampling** (`RandomLib`): `sample(word, domain, i, n)` derives `h = keccak256(abi.encode(word, domain, i, k))` with a counter `k`, rejecting values ≥ `2^256 − (2^256 mod n)` so `h mod n` is unbiased. `weighted(word, domain, i, bps[])` samples `r` in `[0, 10_000)` the same way and walks the cumulative table.

## 6. `MarketCalendar`

Pure library `MarketCalendarLib` plus a small `MarketCalendar` contract holding `version`, `supportedFrom`, `supportedTo`.

- `dayStart(uint32 dateOrdinal) → uint64`: `ordinal × 86_400 + 48_600` in EDT, `+ 52_200` in EST. EDT from the second Sunday of March through the day before the first Sunday of November, computed from the ordinal with a civil-date routine (no lookup table).
- `dayIdAt(uint64 timestamp) → uint32`: the ordinal `d` with `dayStart(d) ≤ t < dayStart(d+1)`. Two candidate ordinals, no loop.
- `weekday(d)`, `isWeekend(d)`, `nextFriday(d)`, `dayEnd(d) = dayStart(d+1)`.
- Reject ordinals outside `[supportedFrom, supportedTo]` (`UnsupportedCalendarDate`). Initial range 2026-01-01 to 2035-12-31.
- Test fixture: a committed JSON of every day start for the range, generated once with Node's `Intl` for `America/New_York`, checked in full by a Foundry test. This is the "verify against full America/New_York data" requirement.

## 7. Metadata service (M1 slice of §10)

Extend the UI's `/meta` route into `/meta/{tokenId}`:

- Reads `character(tokenId)`, lifecycle, owner, and the catalog version's `ipfsRoot`.
- INACTIVE: neutral placeholder JSON. ASSIGNED: frozen assignment with `"status": "pending delivery"`. ACTIVE: name, image URL, and attributes Birth Title, Current Title, Expression, Initial Stress, Generation, Catalog Version, Metadata Revision. Career and health fields are added in M1b/M2.
- `/catalog/{tier}/{expression}/{variantId}.png` serves the placeholder art on testnet, generated deterministically from those three values so the same variant always looks the same.
- No CID publication, no image generation, no state writes.

## 8. Tests

- **RarityConfig**: weight-sum and branch-count validation, version immutability, activation delay.
- **RandomLib**: fuzz that `sample` is unbiased over small `n` (chi-square-style bound over many words) and that `weighted` hits each bucket in proportion; rejection never loops unboundedly.
- **JerkzNFT**: bootstrap exactly 10,000 then closed, no mint or burn path; writer roles; lifecycle transitions and reverts; V6 transfer guard; ownership nonce; generation history; ERC-2981, 165, 4906 interface ids and event signatures.
- **HybridVault**: the full wrap with mock randomness for quantities 1 and 20; ID uniqueness within a request; deferred delivery on a rejecting recipient then successful claim; unwrap round trip restores inventory and principal; refund paths (pre-issuance, operator deadline, and rejection after assignment); stale config hash; fee accounting; V1–V6 as invariants under a handler that wraps, unwraps, refunds, pauses, and sweeps fees.
- **MarketCalendar**: fixture comparison for every day in range; DST transition days; 23/25-hour days; leap days; unsupported dates.
- **Integration with gacha**: a wrapped NFT stocked into the gacha and won keeps its generation and title.
- **Fork**: one live wrap and unwrap on Arc Testnet with the operator source revealing.

## 9. Testnet deployment

The NFT and the vault are new contracts, so both redeploy. Token, randomness source, gacha and faucet stay.

1. `RarityConfig` with the placeholder catalog, activation delay 0.
2. `JerkzNFT` bootstrap: 50 mint calls of 200 into the vault address (precomputed via CREATE2), then close. Expect roughly 5 USDC of gas; the deployer holds about 19.
3. `HybridVault` (JERKZ instance, `CharacterPolicy`) with `refundDeadline = 1h`, registered as a consumer on the operator source, granted the `VAULT` role.
3b. A cirBTC `PackCollection` + `HybridVault` (`SequentialPolicy`, denomination 0.001 cirBTC) as the first RWA pack, to prove the shared wrapper before mainnet assets exist.
4. `MarketCalendar` v1.
5. Unstock the v2 JERKZ from the gacha, wrap ten new ones through the vault, stock them.
6. Point the UI at the new addresses; `/swap` shows request → assigning → delivered and the character card.

## 10. Out of scope here, in order

- **M1b**: WorkRegistry, JobBoard, EmploymentRegistry, QuestEscrow, career ladder writes to `currentTitle`.
- **M1c**: Gacha v2 per `GACHA-SPEC.md` v0.2: depositor inventory, executive gate, 93/5/2 split, revenue ledger. Replaces the v0.1 machine on testnet.
- **M2**: PayrollTreasury, Payroll, VCPayroll (with cirBTC as the stand-in stock on testnet), FeeConverter, Payday Advance. The gacha's 2% payroll royalty lands here as a fee lot.
- **M3**: ItemShop, burns, JUICE, referrals, cosmetic variants, publisher role.
- **M4**: indexer and the remaining app views. **M5**: integrations and the limited test deployment.

## 11. Open items

1. Spec text and deck slide 4 notes still describe 60-second batches and a two-step wrap; decisions 1 and 2 above deviate on implementation only. The product promise (verified randomness, fresh character, 2 USDC, free exit) is unchanged. Record the deviation in §13 acceptance, or reinstate batching later without a migration.
2. Catalog counts, IPFS root and base-identity count are pending from AKLO. Nothing here blocks on them.
3. Cosmetic variant scope (spec §9.1 requires pre-generated variants) needs a count decision before M3 art is produced.
4. Randomness provider for launch is pending the foundation; see `ENTROPY-SPEC.md`.
5. `prd-model.mjs` reference model is pending; M2 tests reproduce it.
