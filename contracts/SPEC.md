# Hybrid Escrow Specification

Status: draft v0.2, 2026-09-08 (updated to match the implementation in `contracts/src`)
Target chain: Arc (testnet chain ID 5042002, mainnet TBD)
Reference design: Metaplex MPL-Hybrid / MPL-404 (Solana), adapted to EVM

## 1. Summary

A fixed-rate, two-way atomic swap between one ERC-20 token and one ERC-721 collection, held in a single escrow contract.

- Rate `R` = 100,000 tokens per NFT.
- Collection size `N` = 10,000 NFTs.
- Token supply `S` = N × R = 1,000,000,000 tokens.

Every NFT outside the escrow is backed by exactly `R` tokens inside the escrow. Every token outside the escrow is backed by a fractional claim on an NFT inside the escrow. Users move between the two states at will:

- **NFT → tokens**: deposit one NFT from the collection, receive `R` tokens.
- **Tokens → NFT**: deposit `R` tokens (plus fee), receive one NFT from the escrow, if any are held.

This is not a liquidity pool. The price is fixed at `R` forever. It is not a bridge. Both assets live on Arc. "Escrow" is the correct word.

## 2. Terminology

| Term | Meaning |
|---|---|
| Token | The ERC-20 fungible asset. |
| NFT / Asset | One ERC-721 token in the registered collection. |
| Escrow | The contract holding tokens and NFTs that are not in user hands. |
| Rate `R` | Tokens per NFT, in token base units. Immutable per escrow. |
| Backing | `R × nftsInCirculation`. Minimum tokens the escrow must hold. |
| Redeem | NFT → tokens. Metaplex calls this `release`. |
| Claim | Tokens → NFT. Metaplex calls this `capture`. |

Metaplex's `capture`/`release` naming is easy to invert. This spec uses `redeem` and `claim` in prose and `redeemNft` / `claimNft` in code.

## 3. Invariants

These must hold after every state-changing call. Tests assert them.

- **I1 Backing.** `token.balanceOf(escrow) ≥ R × (N − escrow.nftCount())`.
- **I2 Conservation.** `escrow.nftCount() + nftsInCirculation == N`. Only NFTs from the registered collection count.
- **I3 No partials.** A claim consumes exactly `R` tokens. A redeem pays exactly `R` tokens. There is no fractional NFT.
- **I4 Fees never touch backing.** Fees are paid to the fee recipient in the same transaction and are never counted toward I1.
- **I5 Admin cannot break I1.** No admin function can withdraw tokens below backing or withdraw NFTs.

## 4. Actors

- **User**: anyone holding tokens or NFTs. Calls `redeemNft`, `claimNft`.
- **Owner**: deploys and configures the escrow. Can pause, set fees, set fee recipient, sweep surplus. Cannot touch backing. Two-step ownership transfer.
- **Fee recipient**: receives swap fees. Any address, set by owner.

## 5. Contracts

### 5.1 `HybridEscrow` (core, this repo)

One escrow per (token, collection) pair. Not upgradeable. Configuration that affects backing is immutable.

**Immutable configuration**

| Field | Type | Notes |
|---|---|---|
| `token` | `IERC20` | Must not be fee-on-transfer or rebasing. Enforced by balance-delta check on deposit. |
| `collection` | `IERC721` | Only tokenIds from this contract are accepted. |
| `rate` | `uint256` | `R` in token base units. e.g. 100_000 × 10^decimals. |
| `maxSupply` | `uint256` | `N`. Used for invariant checks and reporting only. |

**Mutable configuration (owner only)**

| Field | Type | Notes |
|---|---|---|
| `claimFee` | `uint256` | Flat token fee charged on `claimNft`, in token base units. May be 0. |
| `redeemFee` | `uint256` | Flat token fee deducted from the `R` payout on `redeemNft`. May be 0. Default 0. |
| `nativeClaimFee` | `uint256` | Optional fee in native USDC (18 decimals) paid via `msg.value` on `claimNft`. Default 0. See §8.1. |
| `feeRecipient` | `address` | Receives all fees. |
| `paused` | `bool` | Blocks `redeemNft` and `claimNft`. Funding stays open. |
| `selectionMode` | `enum` | `UserChoice` or `Sequential`. See §6.3. |
| `rerollEnabled` | `bool` | Whether `claimNft` / `claimNext` call the collection's re-roll hook. Default false. See §9. |

**State**

- `heldNfts`: enumerable set of tokenIds currently in escrow. Implemented as array + index map for O(1) add, remove, and length.
- `nftCount()`: `heldNfts.length`.
- `backing()`: `rate × (maxSupply − nftCount())`.
- `surplus()`: `token.balanceOf(this) − backing()`.

### 5.2 Token (ERC-20)

Either an existing token or a new one deployed by the launch script. Requirements:

- Standard ERC-20. No transfer fee, no rebasing, no hooks that call back into the escrow.
- Total supply should equal `N × R` for a clean 1:1 hybrid. Supply larger than `N × R` is allowed but the excess is not backed by NFTs.
- Decimals are free. `rate` is always expressed in base units so 18-decimal and 6-decimal tokens both work.

### 5.3 Collection (ERC-721)

Either an existing collection or a new one. Requirements:

- Standard ERC-721 with `transferFrom` and `safeTransferFrom`.
- Exactly `N` tokenIds exist, or minting is capped at `N`. The escrow does not mint.
- No transfer hooks that reenter the escrow. `onERC721Received` on the escrow is supported as a deposit entry point (§6.1).
- Optional: implements `IHybridMetadata` so the escrow can re-roll metadata on claim (§9).

## 6. Flows

### 6.1 Redeem: NFT → tokens

`redeemNft(uint256 tokenId, address recipient)`

Preconditions:
1. Not paused.
2. `collection.ownerOf(tokenId) == msg.sender` and the escrow is approved.
3. `token.balanceOf(escrow) − (rate − redeemFee) ≥ backing after the call`. Always true if I1 held before, because backing drops by `rate` when an NFT enters escrow. Checked anyway.

Effects, in order (checks-effects-interactions, `nonReentrant`):
1. Add `tokenId` to `heldNfts`.
2. `collection.transferFrom(msg.sender, escrow, tokenId)`.
3. If `redeemFee > 0`, `token.safeTransfer(feeRecipient, redeemFee)`.
4. `token.safeTransfer(recipient, rate − redeemFee)`.
5. Emit `Redeemed(msg.sender, recipient, tokenId, rate − redeemFee, redeemFee)`.

Alternative entry: `onERC721Received`. If a user calls `collection.safeTransferFrom(user, escrow, tokenId)` directly, the escrow treats it as a redeem to `from`, with optional `recipient` in `data`. This avoids a separate approve transaction. Transfers of NFTs from any other collection revert.

### 6.2 Claim: tokens → NFT

`claimNft(uint256 tokenId, address recipient) payable`: caller names the NFT. `UserChoice` mode only.
`claimNext(address recipient) payable`: escrow hands out the most recently deposited NFT (LIFO). Allowed in both modes.

Preconditions:
1. Not paused.
2. `nftCount() > 0`. Revert `NoNftsAvailable()` otherwise.
3. For `claimNft`: mode is `UserChoice` (else `WrongSelectionMode()`) and `tokenId` is in `heldNfts` (else `NftNotInEscrow(tokenId)`).
4. `msg.value == nativeClaimFee`. Revert `WrongNativeFee()`.
5. User has approved `rate + claimFee` tokens.

Effects:
1. Remove `tokenId` from `heldNfts` (`claimNext` pops the last element).
2. `token.safeTransferFrom(msg.sender, escrow, rate)` with balance-delta check. Revert `FeeOnTransferToken()` if the delta is less than `rate`.
3. If `claimFee > 0`, `token.safeTransferFrom(msg.sender, feeRecipient, claimFee)`.
4. If `nativeClaimFee > 0`, forward `msg.value` to `feeRecipient` with a low-level call. Revert on failure.
5. If re-rolling is enabled, call `collection.reroll(tokenId, seed)` (§9).
6. `collection.safeTransferFrom(escrow, recipient, tokenId)`.
7. Emit `Claimed(msg.sender, recipient, tokenId, rate, claimFee, nativeClaimFee)`.

### 6.3 NFT selection on claim

Metaplex re-rolls metadata randomly on exit and does not let users choose. That relies on on-chain randomness that Arc does not have: `PREVRANDAO` returns 0 on Arc.

This spec supports two modes, set by the owner:

- **`UserChoice`** (recommended default). The user passes the tokenId they want. Fully deterministic, no randomness dependency, no MEV around "good" NFTs because pricing is flat anyway. Front-ends read `heldNfts` via a paginated view and let the user pick.
- **`Sequential`**. Only `claimNext` is allowed; the escrow hands out the most recently deposited NFT (LIFO pop). Cheapest gas. Predictable, so a user who wants a specific NFT can time their claim after someone else's redeem. Acceptable when all NFTs are equivalent. `claimNext` also works in `UserChoice` mode for callers who do not care which NFT they get.

Random selection is deliberately out of scope for v1. If wanted later, add a `Random` mode backed by a VRF oracle (see Arc oracle docs) or commit-reveal. Blockhash-based randomness is rejected: Arc blocks are 0.5s and proposer-controlled.

### 6.4 Funding

Anyone can fund. Funding only ever increases surplus or escrow NFT count, so it cannot break invariants.

- `fundTokens(uint256 amount)`: pulls tokens into escrow. Emits `TokensFunded`.
- `fundNfts(uint256[] tokenIds)`: pulls NFTs into escrow **without paying out tokens**. Used at launch to seed the NFT side. Emits `NftsFunded`. Owner only, to prevent a user from accidentally donating an NFT they meant to redeem.
- **Minting straight into the escrow** (`from == address(0)` in `onERC721Received`) registers the NFT as funded with no payout. This is how the token-first launch seeds the NFT side in one step.
- `registerHeld(uint256[] tokenIds)`: owner-only. Registers NFTs the escrow already owns but is not tracking, for example ones sent with plain `transferFrom`, which has no callback. Requires `collection.ownerOf(id) == escrow`.
- Direct `token.transfer` to the escrow also works and counts as surplus.

### 6.5 Launch configurations

Because `S = N × R` exactly, the launch state is any split where I1 holds. Three practical options:

| Mode | Escrow holds at launch | Users hold | Use when |
|---|---|---|---|
| **Token-first** | all `N` NFTs, 0 tokens | all `S` tokens | Token already distributed, NFTs are new. Users claim NFTs by depositing tokens. |
| **NFT-first** | all `S` tokens, 0 NFTs | all `N` NFTs | NFTs already distributed, token is new. Users redeem NFTs for tokens. |
| **Mixed** | `k` NFTs and `(N − k) × R` tokens | the rest | Partial distribution of both. |

The user's description ("pools tokens into escrow, users redeem tokens by depositing an NFT") matches **NFT-first**. The launch script supports all three via parameters. See §11.

### 6.6 Admin

- `setFees(claimFee, redeemFee, nativeClaimFee)`: emits `FeesUpdated`. Token fees are bounded to `rate / 10` each, so a fee can never exceed 10% and can never make a redeem payout zero. `nativeClaimFee` is a different asset and is unbounded; a hostile value only makes claims unattractive, it cannot move funds.
- `setRerollEnabled(bool)`: toggles the metadata re-roll hook call on claim (§9).
- `setFeeRecipient(address)`: non-zero.
- `setSelectionMode(mode)`.
- `pause()` / `unpause()`.
- `sweepSurplus(address to, uint256 amount)`: requires `amount ≤ surplus()`. Emits `SurplusSwept`. This is the only way tokens leave the escrow other than redeem.
- `rescueERC20(address foreignToken, ...)`: only for tokens that are not `token`.
- `rescueERC721(address foreignCollection, ...)`: only for collections that are not `collection`.
- `transferOwnership` / `acceptOwnership` (two-step).

There is no `withdrawNfts` and no `withdrawBacking`. There is no upgrade path. If the escrow needs to change, deploy a new one and migrate by having users swap out.

## 7. Views

- `nftCount() → uint256`
- `heldNftAt(uint256 index) → uint256`
- `heldNfts(uint256 offset, uint256 limit) → uint256[]`
- `isHeld(uint256 tokenId) → bool`
- `backing() → uint256`
- `surplus() → uint256`
- `claimCost() → (uint256 tokens, uint256 native)` = `rate + claimFee`, `nativeClaimFee`
- `redeemPayout() → uint256` = `rate − redeemFee`
- `config()` returns the full immutable and mutable configuration in one call, including `rerollEnabled` and `paused`.

## 8. Arc-specific considerations

### 8.1 Native fee is in USDC with 18 decimals

`nativeClaimFee` is denominated in native USDC, which has 18 decimals on Arc even though ERC-20 USDC at `0x3600…0000` has 6. A $0.10 native fee is `100_000_000_000_000_000` (1e17), not `100_000`. The deploy script takes the fee in whole USDC cents and converts. Front-ends display it with `formatUnits(x, 18)`.

### 8.2 If the ERC-20 token is USDC itself

Not the primary case, but supported. If `token` is USDC at `0x3600…0000`, `rate` is in 6-decimal units (100k USDC = `100_000 × 10^6`). The escrow never touches the native 18-decimal view for the token side, so there is no mixing. The one hazard is a native `msg.value` fee plus a USDC `claimFee` in the same call: they are the same asset at different decimals and must never be summed in accounting or UI.

### 8.3 EIP-7708 system emitter

Every native USDC movement, including the `nativeClaimFee` forward, emits a `Transfer` log from `0xffffFFFfFFffffffffffffffFfFFFfffFFFfFFfE`. Indexers tracking escrow fees must not double-count these against the escrow's own `Claimed` event.

### 8.4 Self-destructed accounts

Native value transfer to a self-destructed account reverts on Arc. `feeRecipient` is validated to be non-zero; the owner is responsible for keeping it a live account. A failed native fee forward reverts the whole claim, which is the safe outcome.

### 8.5 Randomness

`PREVRANDAO` is 0. See §6.3. No code path may depend on `block.prevrandao` or `blockhash` for anything security-relevant.

### 8.6 Timestamps

Arc block timestamps can repeat across consecutive blocks. Nothing in this design depends on `block.timestamp`.

### 8.7 Fees and gas

Base fee floor is 20 gwei in USDC. A claim or redeem is roughly 100k–150k gas, so the network fee is on the order of $0.002–0.003. Fees in this spec are protocol-level, separate from gas.

### 8.8 Local testing

Anvil does not model Arc's native USDC precompile or EIP-7708 logs. Unit tests run on Anvil with a mock ERC-20. Integration tests run against a fork of Arc Testnet and at least one live testnet deployment before mainnet.

## 9. Metadata: captured on entry, unique re-roll on exit

Mirrors MPL-Hybrid, whose program blanks an asset's URI to `baseUri + "captured.json"` when it enters escrow and rewrites it to `baseUri + index + ".json"` with a random index when it leaves. Two deliberate departures: indices are **unique** (each piece of metadata is in circulation at most once), and the random draw is **asynchronous and unsnipable** rather than derived from block data.

**States** (in `HybridCollection`):

| State | When | `tokenURI` |
|---|---|---|
| `Captured` | in escrow, or freshly minted | `baseURI + "captured.json"` |
| `Unrevealed` | left escrow, reveal request in flight | `baseURI + "unrevealed.json"` |
| `Revealed` | holds an index | `baseURI + index + ".json"` |

**Hooks.** The escrow calls `onEnterEscrow(tokenId)` on every deposit path (redeem, `safeTransferFrom`, `fundNfts`, `registerHeld`) and `onExitEscrow(tokenId)` on every claim, when `rerollEnabled` is on. Mints into escrow skip the hook since fresh tokens are already `Captured`.

**Index pool.** `[minIndex, maxIndex]`, at least `maxSupply` wide, held as a lazy Fisher-Yates shuffle so there is no initialization cost. Exit draws one uniformly; entry returns the token's index. Fuzz tests assert every index in range is drawn exactly once and uniqueness survives arbitrary churn. The range is locked after the first draw.

**Randomness.** The collection is a consumer of the same `IRandomnessSource` the gacha uses (`GACHA-SPEC.md` §6). On exit it requests a word; the user already holds the NFT in the `Unrevealed` state. When the source fulfils, the index is drawn and the token becomes `Revealed`. A reveal that arrives after the token re-entered escrow, or after a re-request, is discarded, not applied. The owner may re-request any time; the holder may re-request after `revealTimeout` (default 1h).

Why not Metaplex's block-hash seed: on Arc a contract can claim and revert unless it drew a rare index, at about a third of a cent per attempt. The async draw removes that entirely, and switching to a VRF later is one owner call on the collection.

**ERC-4906** `MetadataUpdate` is emitted on every state change so marketplaces refresh. ERC-721 has no on-chain per-token name; "Captured" lives in the placeholder JSON.

## 10. Security

Threats considered and their mitigations:

| Threat | Mitigation |
|---|---|
| Reentrancy via ERC-721 hooks or malicious token | `nonReentrant` on all state-changing entry points. CEI ordering. |
| Fee-on-transfer or rebasing token drains backing | Balance-delta check on every token pull. Token is immutable so a bad token is a deployment error, caught in tests. |
| Foreign NFT deposited to spoof escrow count | `onERC721Received` reverts unless `msg.sender == collection`. `fundNfts` and `redeemNft` only call the registered collection. |
| Admin rug | No function can move backing or NFTs out. Only surplus is sweepable. Non-upgradeable. |
| Fee set to 100% | Fees capped at 10% of rate. |
| Griefing the LIFO queue | `Sequential` mode is opt-in; `UserChoice` is default. |
| Front-running a specific tokenId claim | Possible in `UserChoice` mode. Cost to the victim is one failed tx; pricing is flat so there is no economic loss. Accepted. |
| Pause used to trap funds | Pause blocks swaps only. Owner cannot extract anything while paused. Documented as a circuit breaker. |
| Native fee forward to a contract that reverts | Whole tx reverts, user keeps tokens. Owner must set a receivable `feeRecipient`. |

Out of scope: risks inside the token or collection contracts themselves when they are pre-existing.

## 11. Deployment and launch

Foundry scripts, one per step, all idempotent against a JSON deployments file keyed by chain ID.

1. `DeployToken.s.sol` (skip if token exists): mints `N × R` to the deployer or directly to the escrow address computed via CREATE2.
2. `DeployCollection.s.sol` (skip if collection exists): mints `N` NFTs to the deployer or escrow.
3. `DeployEscrow.s.sol`: deploys `HybridEscrow(token, collection, rate, N, feeRecipient, fees, mode)` via the canonical CREATE2 factory at `0x4e59b44847b379578588920cA78FbF26c0B4956C` so the address is the same on testnet and mainnet.
4. `Fund.s.sol`: moves the chosen launch split into escrow (§6.5) and asserts I1 and I2 before finishing.
5. Verify on Blockscout: `forge verify-contract --verifier blockscout --verifier-url https://testnet.arcscan.app/api/`.

Testnet first, using `https://rpc.testnet.arc.io`, chain 5042002. Mainnet addresses and chain parameters are not yet published by Circle; the scripts read them from config.

## 12. Testing plan

- **Unit (Anvil, mock ERC-20 and ERC-721)**: every flow, every revert path, fee math, selection modes, admin bounds, `onERC721Received` path, foreign-asset rejection.
- **Invariant fuzzing (Foundry `invariant_*`)**: random sequences of redeem (both entry points), claim, fund, sweep, fee and mode changes from a mixed launch state. Assert I1, I2, I4, surplus math, and token supply conservation after every call, with `fail_on_revert = true` so the handler must model preconditions rather than hide reverts. This is the most important test in the repo.
- **Fork tests against Arc Testnet**: native fee forward, EIP-7708 log emission, real USDC as the token.
- **Gas snapshots** committed and checked in CI.
- **Static analysis**: Slither in CI.

## 13. Repository layout

The escrow is one component of a larger project. It lives in `contracts/` as a self-contained Foundry project so other components (front-end, indexer, additional contracts) can sit beside it at the repo root.

```
jerkz/
  contracts/                        # this component
    SPEC.md                       # this file
    foundry.toml
    src/
      HybridEscrow.sol
      interfaces/IHybridEscrow.sol
      interfaces/IHybridMetadata.sol
      tokens/HybridToken.sol      # optional, plain ERC-20 with fixed supply
      tokens/HybridCollection.sol # optional, ERC-721 with capped supply and reroll hook
    script/
      DeployToken.s.sol
      DeployCollection.s.sol
      DeployEscrow.s.sol
      Fund.s.sol
    test/
      HybridEscrow.t.sol
      HybridEscrow.invariant.t.sol
      fork/ArcTestnet.t.sol
      mocks/
    deployments/
      5042002.json
  <other components>/             # added alongside contracts/ as the project grows
```

Solidity 0.8.x, `evm_version = "osaka"` or the newest Foundry supports below Arc's baseline. OpenZeppelin Contracts v5 for ERC-20, ERC-721, `Ownable2Step`, `Pausable`, `ReentrancyGuard`, `SafeERC20`.

## 14. Open questions

The implementation was scaffolded with the bold defaults on 2026-09-08. Each can still be changed; items 1–3 are script inputs, 4–6 are owner calls after deploy.

1. **Token and collection: new or existing?** Default: **deploy both new** in this repo with fixed supplies.
2. **Token decimals** if new. Default: **18**.
3. **Launch split.** Default: **NFT-first** (all tokens in escrow, all NFTs to the deployer for distribution).
4. **Fees.** Default: **claimFee 0, redeemFee 0, nativeClaimFee 0**. Fees can be switched on later by the owner.
5. **Selection mode.** Default: **UserChoice**.
6. **Metadata re-roll.** Decided 2026-09-09: **on**, unique indices, async reveal (§9).
7. **Should `fundNfts` be owner-only or open?** Default: **owner-only**.
8. **Who is the owner at launch?** Default: deployer EOA, with a note to move to a multisig before mainnet.

## 15. Non-goals

- Price discovery or AMM behavior. The rate is fixed.
- Cross-chain movement. Both assets stay on Arc. CCTP or Gateway integration would be a separate front-end concern.
- ERC-404 style dual-nature tokens where one contract is both fungible and non-fungible. This design keeps two ordinary contracts and an escrow between them, exactly like MPL-404.
- Upgradeability.
