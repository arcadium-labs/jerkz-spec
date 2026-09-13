# Pyth Entropy Integration Specification

Status: ready to implement, blocked on a Pyth Entropy deployment on Arc. Written 2026-09-09.
Companions: `GACHA-SPEC.md` §6 (randomness interface), `SPEC.md` §9 (metadata reveal).

## 1. Goal

Replace the POC `OperatorCommitReveal` source with Pyth Entropy as the randomness behind both
consumers, the gacha machine and the collection's metadata reveal, without changing either
consumer. The switch is one owner call per consumer and is reversible.

## 2. What we are waiting for

Confirmed 2026-09-09 that none of this exists yet on Arc (see `GACHA-SPEC.md` §6.3):

| Need | How to confirm it has arrived |
|---|---|
| Entropy contract on Arc Testnet (5042002) | `cast call <addr> "getFeeV2()(uint128)" --rpc-url https://rpc.testnet.arc.io` returns a number instead of reverting. `0x2880aB155794e7179c9eE2e38200202908C17B43` is Pyth Core (price feeds), not Entropy. |
| Fortuna provider fulfilling on Arc | `curl https://fortuna-staging.dourolabs.app/v1/chains` lists an `arc-testnet` entry, and `getDefaultProvider()` on the contract returns a non-zero address. |
| Listed on the chainlist | https://docs.pyth.network/entropy/chainlist shows Arc under Testnets with reveal delay, default gas limit and fee. |
| Same for Arc mainnet (5042) | Later. Not needed for the POC. |

Also ask Pyth for: the reveal delay they chose for Arc (0.5 s blocks; one block is plenty), and
confirmation that fees are quoted in native USDC at 18 decimals.

## 3. Entropy interface we build against

`@pythnetwork/entropy-sdk-solidity` 2.2.1 (pin it). Relevant surface:

```solidity
interface IEntropyV2 {
    function requestV2() external payable returns (uint64 sequenceNumber);
    function requestV2(uint32 gasLimit) external payable returns (uint64 sequenceNumber);
    function requestV2(address provider, uint32 gasLimit) external payable returns (uint64 sequenceNumber);
    function requestV2(address provider, bytes32 userRandomNumber, uint32 gasLimit) external payable returns (uint64);
    function getFeeV2() external view returns (uint128);
    function getFeeV2(uint32 gasLimit) external view returns (uint128);
    function getFeeV2(address provider, uint32 gasLimit) external view returns (uint128);
    function getDefaultProvider() external view returns (address);
}

abstract contract IEntropyConsumer {
    function _entropyCallback(uint64 sequenceNumber, address provider, bytes32 randomNumber) external;
    function entropyCallback(uint64 sequenceNumber, address provider, bytes32 randomNumber) internal virtual;
    function getEntropy() internal view virtual returns (address);
}
```

The SDK ships `MockEntropy.sol`; tests use it so nothing here waits on the deployment.

## 4. The adapter: `PythEntropySource`

`src/randomness/PythEntropySource.sol`. Implements `IRandomnessSource` toward our
consumers and `IEntropyConsumer` toward Pyth. Roughly 120 lines. Non-upgradeable, `Ownable2Step`.

**State**

| Field | Notes |
|---|---|
| `entropy` (immutable) | The Entropy contract on this chain. |
| `provider` | Defaults to `entropy.getDefaultProvider()`; owner may override. |
| `consumers[address] → bool` | Only allowed consumers may request. |
| `gasLimit[consumer] → uint32` | Callback gas per consumer. See §6. |
| `requests[requestId] → { consumer, sequence, status }` and `bySequence[uint64] → requestId` | Two-way map. |

**`requestRandomness(pullId, clientSeed) → requestId`** (consumer only)

1. `fee = entropy.getFeeV2(provider, gasLimit[msg.sender])`. Revert `InsufficientFeeBalance(fee, balance)` if the adapter's balance is short (§5).
2. `seq = entropy.requestV2{value: fee}(provider, clientSeed, gasLimit[msg.sender])`. The client seed goes in as Pyth's `userRandomNumber`, so the player's entropy is still folded in.
3. `requestId = keccak256(abi.encode(address(this), msg.sender, pullId, seq))`. Store both maps, status `Open`. Emit `Requested(requestId, consumer, pullId, seq, fee)`.

**`entropyCallback(seq, provider, randomNumber)`** (Pyth only, enforced by the SDK base)

1. `requestId = bySequence[seq]`. Unknown → revert (a Pyth bug, worth surfacing).
2. If status is `Cancelled`, emit `Discarded` and return without reverting. Our consumers also reject non-pending work themselves, so this is belt and braces.
3. Status `Fulfilled`. `IRandomnessConsumer(consumer).fulfillRandomness(requestId, uint256(randomNumber))`.
4. If the consumer call reverts, catch it, store the word in `pendingWords[requestId]`, emit `CallbackFailed`, and expose `retry(requestId)` (anyone) that re-delivers. This matters because Entropy will not call back twice; a consumer revert (for example a gacha delivery that runs out of gas) must not lose the word.

**`cancel(requestId)`** (requesting consumer only): mark `Cancelled`. Entropy itself has no cancel; the callback will still arrive and be discarded. The fee is not refunded.

**Owner**: `setConsumer`, `setGasLimit`, `setProvider`, `fund() payable`, `withdraw(to, amount)`.

## 5. Fees

Entropy charges per request, in native value, quoted by `getFeeV2`. On Arc that is USDC with 18
decimals. Expect something on the order of a cent on testnet; confirm with Pyth.

The adapter pays from its own balance. The owner tops it up with `fund()`. A `LowBalance(balance,
fee)` event fires when a request leaves less than ten fees behind, so the keeper-replacement
monitoring (§9) can alert.

Not in scope now: making `IRandomnessSource.requestRandomness` payable so the gacha forwards part of
the pull price. Easy to add later; it changes the interface both consumers implement, so it is a
deliberate second step once the fee is known.

## 6. Callback gas limits

Entropy's callback runs with a gas limit chosen at request time and the fee scales with it.
Measured costs of our consumers' `fulfillRandomness`:

| Consumer | Work in the callback | Gas limit to request |
|---|---|---|
| `HybridCollection` | pool draw + two storage writes + two events | 200,000 |
| `GachaMachine` | Fenwick find + bookkeeping + deliver every asset in the pack | 250,000 + ~60,000 per ERC-20/721 asset, ~90,000 per ERC-1155. 64-asset packs need ~4M. |

Set `gasLimit[gacha]` to cover the largest pack actually stocked. The deploy script reads
`ENTROPY_GAS_GACHA` and `ENTROPY_GAS_COLLECTION` with defaults 1,500,000 and 200,000. If a pack
exceeds the limit the callback reverts; §4 step 4 keeps the word so `retry` can finish it, and the
owner can raise the limit for future pulls.

## 7. Consumers: what changes

Nothing in `GachaMachine` or `HybridCollection`. Both already:

- record the source that issued each request and only accept settlement from it, so pending
  pulls and reveals issued by the operator source still settle through the operator source after
  the switch;
- discard stale or duplicate fulfilments without reverting.

The switch is `gacha.setSource(adapter)` and `collection.setRandomnessSource(adapter)`.

## 8. Tests

`test/gacha/PythEntropySource.t.sol` against the SDK's `MockEntropy`:

- request pays exactly `getFeeV2`, stores both maps, forwards the client seed;
- callback settles a gacha pull and a collection reveal end to end;
- callback for a cancelled request is discarded, not reverted;
- consumer revert is caught, word retained, `retry` delivers;
- unknown sequence reverts; non-Entropy caller reverts (SDK check);
- fee balance exhaustion reverts the request with the right error; `LowBalance` fires;
- source swap mid-flight: a pull opened on the operator source settles there, the next pull goes to Entropy.

Fork test on Arc Testnet once the address exists: deploy adapter, one real pull, wait for Fortuna,
assert settlement. This is the acceptance test.

## 9. Deployment runbook (testnet)

```sh
cd contracts && pnpm add -D @pythnetwork/entropy-sdk-solidity@2.2.1   # + remapping
forge test

# 1. confirm the deployment (see §2)
cast call $ENTROPY "getFeeV2()(uint128)" --rpc-url arc_testnet
cast call $ENTROPY "getDefaultProvider()(address)" --rpc-url arc_testnet

# 2. deploy + fund the adapter (registers gacha and collection as consumers)
ENTROPY_ADDRESS=$ENTROPY ENTROPY_FUND_USDC=2 forge script script/DeployEntropySource.s.sol "${DEPLOY_FLAGS[@]}"

# 3. switch both consumers (owner calls, in the same script behind SWITCH=1)
#    gacha.setSource(adapter); collection.setRandomnessSource(adapter)

# 4. acceptance: one pull on /gacha and one claim on /swap; both settle without the keeper running
```

Then stop the operator keeper. Keep `OperatorCommitReveal` deployed; it is the rollback (§10).
Verify the adapter on Arcscan with the standard-JSON route we used for the others.

Registry: add `entropySource` to `deployments/5042002.json` and to `DeployBase.Deployments`.

## 10. Rollback

`gacha.setSource(operatorSource)` and `collection.setRandomnessSource(operatorSource)`, restart the
keeper, recommit seeds. Requests already opened on Entropy still settle through Entropy. Nothing is
lost either way; that is the point of recording the source per request.

## 11. UI

- Gacha page: the "Randomness queue" stat reads the operator source's commitment count. Replace with
  a "Randomness" stat that shows `Pyth Entropy` and the adapter's fee balance in USDC, or the
  operator queue depth when the operator source is active. Read `gacha.source()` to decide.
- Home page: update the randomness paragraph to say Pyth Entropy on Arc, with the operator
  commit-reveal described as the fallback.
- No change to the pull or reveal flows; the pending state simply ends when Fortuna calls back
  instead of when the keeper does. Expect one to three seconds.

## 12. Monitoring (replaces the keeper)

A small script or cron that reads the adapter balance and alerts below a threshold, and that
counts `CallbackFailed` events. Fortuna's own explorer at https://entropy-explorer.pyth.network
shows request and fulfilment status per chain once Arc is listed.

## 13. Open questions for the Pyth conversation

1. Reveal delay on Arc, and whether Fortuna's testnet provider will run with the same delay.
2. Fee quote in native USDC: what `getFeeV2()` returns on Arc and how often it moves.
3. Maximum callback gas limit they will honour on Arc (our large packs want up to 4M).
4. Whether a mainnet deployment is planned for the same window, so the runbook runs twice.

## 14. Effort

About half a day once the address is live: adapter and tests are a few hours, the deploy and
acceptance pull are under an hour, UI stat swap is minutes. Everything except the acceptance test can
be done before the deployment exists, and should be, so the switch itself is one script run.

## Addendum 2026-09-13 (review fixes)

- The adapter forwards `gasleft() - CALLBACK_RESERVE` (60k) to the consumer so its catch block can always record the word; `isPending` returns false once Entropy reports `CALLBACK_FAILED` for the sequence or the provider has revealed a later sequence (hash chain). Zero the consumers' refund deadlines during any migration anyway.
- Fees are budgeted per consumer: `fundConsumer(consumer)` (anyone) or `allocate(consumer, amount)` (owner, from unassigned balance). `withdraw` only touches unassigned balance. Budget each consumer for its own draw rate; the hiring draw is free to players, so its budget is the one to watch.
- Gacha settlement: budget the machine's callback for the settlement itself plus two staged activations (~380k at 1,500 prizes); the keeper activates the rest with `activateStaged`.
