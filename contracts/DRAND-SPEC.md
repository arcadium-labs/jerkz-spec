# Randomness from drand (replaces the operator commit-reveal and the Pyth Entropy plan)

Decided 2026-09-13 (Scott: "ship it"). JERKZ draws its randomness from the **drand evmnet** beacon, a public
randomness network run by the League of Entropy, verified on chain. Nobody controls the value: not the
operator, not the keeper, not the players.

## The beacon

| | |
|---|---|
| Network | drand evmnet, scheme `bls-bn254-unchained-on-g1` |
| Chain hash | `04f1e9062b8a81f848fded9c12306733282b2727ecced50032187751166ec8c3` |
| Genesis / period | 1727521075 / 3 s; `beaconTime(round) = genesis + (round − 1) × 3` |
| Relays | api.drand.sh, api2.drand.sh, api3.drand.sh, drand.cloudflare.com (`/<chain>/public/<round>`); HTTP 425 until a round exists; every round is archived forever |
| Verification | pure Solidity over the BN254 precompiles (0x05–0x08); `DrandVerifier.verifyBeacon(round, sig) → sha256(sig)`; ~253k gas |
| Verifier source | `src/randomness/drand/DrandVerifier.sol` over `randa-mu/bls-solidity` @ 11af179 (MIT) with the two fail-closed return-size checks taken from Gold-Supply/gldsupply; same equation as GLDDrandVerifier `0xe4E1…d81D` on Arc testnet |

## The adapter: `DrandSource`

Implements the existing `IRandomnessSource` seam, so no consumer changes.

- `requestRandomness(pullId, clientSeed)` opens a request. In **timed** mode the round is chosen now: the first
  round published after `block.timestamp + margin`. In **selected** mode the round stays unset until the operator
  calls `select(requestId, round)`; the contract only enforces `round ≥ roundAfter(requestedAt + margin)`.
- `fulfill(requestId, signature)`: permissionless. Verifies the exact round's beacon, derives
  `word = keccak(domain, chainId, source, requestId, clientSeed, randomness)`, marks the request fulfilled, then calls
  the consumer; a reverting callback keeps the word (`retry`).
- `isPending`: true while the round is unset or its publication is still more than `PUBLISH_SLACK` (90 s) away by
  chain time. After that the beacon may be public, so `cancel` and every consumer refund path are refused.

Consumers on testnet: Jerk vault (**selected**, margin 15 s), Payroll cohort seed (timed, 90 s), gacha (timed, 90 s).

## Trust model, stated plainly

drand proves a beacon is **authentic**, not that it was **unknown** when a request was made. Arc's block timestamps
may lag wall time, so "first round after block.timestamp + margin" is a judgement with a small residual risk (a
proposer who could skew the timestamp far behind real time could bind a request to an already-public round). For
the daily payroll seed and the gacha that is accepted. For mints, where a favourable beacon is a rare title, the
operator (the keeper key) selects the round after seeing the request in a confirmed block. The operator can stall
or, if compromised, pick a round that is already public; it can never forge a value. Compared with the operator
commit-reveal it replaces, the entropy is never in anyone's hands and every delivered word is publicly checkable.

Before mainnet: a dedicated operator key for selection, a longer vault margin, and a documented monitor that
compares every `Selected` round against the request block's time.

## Operations

The keeper (private repo `jerkz-keeper`) selects rounds for selected-mode requests from the request block's
timestamp, fetches published beacons from the relays and posts `fulfill`. Anyone else may post too. There are no
seeds to keep, no queue to top up, no fee balance.

### Retiring the operator source (status 2026-09-14)

Every live consumer (vault, gacha, Payroll v3) draws from `DrandSource`. The keeper still runs the
`OperatorCommitReveal` reveal duty for one reason: the retired r4 payroll (`0x4d72…865E`, New York
calendar) has budgets committed through Sat 2026-09-19 and each of its day seals requests a seed from
the operator source; the reveal lets the day finalize so the keeper's legacy drain can return the
reserve to the live payroll. 85 commitments are queued, enough for those seals. After 2026-09-19 the
reveal sweep, watcher and top-up leave the keeper and the operator source is history. The retired r5
payroll (`0x9F87…8Dbc`) is a drand consumer with budgets through the same date; its consumer flag
must stay on until then (council schedule nonce 24, which revokes it, is to be cancelled rather than
executed).
