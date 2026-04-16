---
trigger: glob
description: Apply these low-level Solidity rules when reading or editing contract files for the Base MEV executor.
globs:
  - "src/**/*.sol"
  - "contracts/**/*.sol"
  - "**/*Executor*.sol"
---

# Solidity Hot-Path Rules for the Base MEV Executor

## Hot-path memory policy

Treat any hidden calldata->memory movement as suspicious.

Forbidden in the hot path unless explicitly justified in writing:
- `abi.encode(...)` for route or callback payloads
- `abi.decode(...)` for route or callback payloads
- copying the full route blob into memory
- rebuilding paths as memory arrays
- helper abstractions that smuggle dynamic bytes into memory

Preferred pattern:
- parse `msg.data` or calldata slices with explicit offset math
- keep callback context in transient storage or compact storage/context words
- read only the fields needed at each step

## Callback rules

Every callback must verify all three:
- active execution context exists
- callback kind matches the expected enum value
- `msg.sender` equals the exact expected pool/lender

Do not use a generic nonReentrant pattern that blocks legitimate callbacks.

## Concentrated-liquidity swap rules

- do not default `sqrtPriceLimitX96` to the universal min/max sentinels in final production code
- use a real guard sourced from compact route metadata
- do not claim early abort exists if the price limit is effectively disabled

## Exact-output rules

Exact-output is not allowed to piggyback on the forward exact-input loop.

Allowed options:
- reject exact-output entirely
- implement a separate reverse-oriented mode / dispatcher / entrypoint

Forbidden option:
- pretending exact-output can be “just another branch” inside the same simple forward loop

## V2 exact-in surplus harvesting

For V2-style legs:
- do not assume the requested `amountOut` equals all value that can be harvested
- use balance deltas or equally exact realized-accounting methods to capture surplus/dust

## Transfer helper rules

In the hot path, a blind low-level ERC20 transfer helper is acceptable if:
- the token universe is pre-vetted off-chain
- return-data handling is correct
- the tradeoff is documented

In the cold path:
- `SafeERC20` is acceptable

## Red-flag constructs to review immediately

If you see any of these in a hot-path patch, stop and re-check the invariants:

- `abi.encode(`
- `abi.decode(`
- `new bytes`
- `bytes memory`
- `SafeERC20` inside the swap/flash/callback path
- `sqrtPriceLimitX96 = MIN_`
- `sqrtPriceLimitX96 = MAX_`
- scalar hot-path function parameters for `tokenA`, `amount`, `minProfit`
- explicit canonical UniV3 pool address fields in calldata structs
- `delete` on transient-like state
- exact-output branches inside the forward loop

## Preferred implementation order

1. Entry-point and raw parser
2. Lock truth table
3. Pool derivation
4. Flash path
5. CL swap callback
6. V2 path and dust harvesting
7. Profit firewall
8. Cold/admin surface
9. Self-audit
