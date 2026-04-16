---
trigger: always_on
description: Hard cross-cutting invariants for the Base Mainnet MEV executor. Keep these rules above local coding convenience.
---

# Base Mainnet MEV Executor — Hard Axioms

## Objective hierarchy

When making tradeoffs, prioritize in this order:

1. Correctness of callback and repayment mechanics
2. Preservation of calldata/L1-fee efficiency
3. Preservation of hot-path execution efficiency
4. Honest ABI boundaries and fail-closed behavior
5. Readability and convenience

## The contract you are building

The target is a **single-contract, direct-pool arbitrage executor** for **Base Mainnet** under competitive L2 MEV constraints.

This is **not**:
- a tutorial
- a router integration
- a safety-first MVP
- a generic venue abstraction layer
- an off-chain planner

## Must-preserve invariants

- Do not move route metadata into memory in the hot path.
- Do not `abi.encode` callback payloads that contain route data.
- Do not `abi.decode` route payloads inside callbacks.
- Do not pass scalar ABI hot-path arguments if they can be packed into the raw route blob.
- Do not pass 20-byte canonical CL pool addresses when deterministic on-chain derivation is available.
- Do not mix exact-output execution into the forward exact-input loop.
- Do not omit real `sqrtPriceLimitX96` protection on CL legs.
- Do not use `SafeERC20` in the hot execution path unless explicitly justified as a bounded compromise.
- Do not invent protocol compatibility.
- Do not market a bounded compromise as optimal.

## Venue-family discipline

Treat these as different families unless exact verified interfaces prove otherwise:

- Uniswap V3 canonical concentrated-liquidity pools
- Aerodrome V2 classic pools
- Aerodrome Slipstream concentrated-liquidity pools
- Balancer V2-style flash lenders
- Balancer V3 / transient-accounting vault systems

If interface certainty is missing, reject the family instead of inventing support.

## Entry-point discipline

Preferred production entrypoint:
- `fallback()` reading `msg.data` directly

Acceptable only with explicit written justification:
- `execute(bytes calldata routeData)`

Final production hot path should **not** depend on:
- `execute(address tokenA, uint256 amount, uint256 minProfit, bytes calldata routeData)`
- any ABI surface that bloats calldata with scalar arguments and dynamic-offset framing

## Pool-address discipline

### Canonical concentrated-liquidity families
If the venue uses a deterministic CREATE2 pool address (for example a canonical UniV3-style family), then:
- derive the address on-chain
- sort tokens locally
- use factory + init-code-hash + fee/tick-spacing input
- do not carry the 20-byte address in calldata unless it is explicitly a non-canonical override mode

### Aerodrome V2 classic
Aerodrome V2 classic is not the same problem. Do not force it into a fake UniV3 CREATE2 template. Use the exact factory/pool verification semantics intended for that family.

## Transient-lock discipline

Define one lock truth table and reuse it mechanically.

Example pattern:
- `0 = UNLOCKED`
- `1 = EXPECT_UNIV3_FLASH_CALLBACK`
- `2 = EXPECT_UNIV3_SWAP_CALLBACK`
- `3 = EXPECT_EXTERNAL_FLASH_CALLBACK`

If you write `1` before a call, the callback must require `1`. Never set one value and check for another. Never let different branches reinterpret the same value.

## Work sequence discipline

In every serious coding session:
1. Read the Skill support files.
2. Produce a defect map against the current code.
3. Freeze architecture decisions.
4. Patch only one subsystem at a time.
5. Run the self-audit after each subsystem patch.
6. Stop immediately if any invariant breaks.

## Off-chain / on-chain split

Off-chain owns:
- opportunity discovery
- route search
- token/pool vetting
- builder / relay / private-order-flow strategy
- fee forecasting including Base L1 fee
- whether a blind hot-path transfer is safe for a given token universe

On-chain owns:
- exact execution
- callback correctness
- exact repayment
- realized profitability checks
- atomic failure

## Required response style inside this repo

When editing or proposing code, always make the following explicit:

- what was changed
- which invariant it satisfies
- which risk it removes
- which bounded compromises still remain
