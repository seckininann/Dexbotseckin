# Constitution: What This Executor Is

## Mission

Build a **single-contract**, **direct-pool**, **Base Mainnet** arbitrage executor for highly competitive L2 MEV.

## Ranked design goals

1. Exact callback and repayment correctness
2. Minimal calldata / L1-fee pressure
3. Minimal hot-path compute overhead
4. Honest venue-family boundaries
5. Realized-profit gating
6. Clear separation of off-chain and on-chain roles

## Non-goals

Do not optimize for:
- tutorial readability
- broad protocol universality
- router convenience
- easy ABI surfaces for frontends
- “safe but slow” default Solidity patterns in hot paths

## Architectural commandments

- The hot path should not move route metadata into memory.
- The contract should prefer raw `msg.data` parsing over dynamic ABI wrappers.
- Canonical concentrated-liquidity pools should be derived on-chain when deterministic derivation is available.
- Exact-output is a separate paradigm, not a boolean branch inside the same simple forward loop.
- Realized balances decide profitability.
- Unsupported families are rejected, not guessed.

## Common failure modes to guard against

- using `execute(bytes)` plus `abi.decode` because it “feels cleaner”
- using `abi.encode` to smuggle route data into callbacks
- writing one transient lock value and checking for another
- passing 20-byte canonical pool addresses instead of deriving them
- faking universal pool compatibility
- omitting CL price limits and hoping the final min-profit check is enough
- using `SafeERC20` everywhere out of habit
