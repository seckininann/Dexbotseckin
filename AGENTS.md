# Base Mainnet MEV Executor — Repo Laws

This repository exists to build a **single-contract, direct-pool, calldata-minimized, Base Mainnet arbitrage executor**. Optimize for searcher competitiveness, not tutorial clarity.

## Always-on non-negotiables

- **Base cost model:** calldata/L1 security fee is usually more important than raw L2 compute. Spend compute if it removes bytes.
- **No routers or periphery.** Direct pool/pair interaction only.
- **Final hot-path entrypoint:** prefer raw calldata through `fallback()`. Do not finalize a hot path built around scalar ABI arguments or `execute(bytes)` unless you explicitly document why the extra ABI framing is acceptable.
- **No route memory copies in hot path.** No `abi.encode(...)` / `abi.decode(...)` for route or callback payloads inside hot execution or callbacks.
- **Canonical CL pool resolution:** for Uniswap V3-style canonical pools, derive pool addresses on-chain from sorted tokens + fee + factory + init-code-hash. Do not pass a 20-byte pool address in calldata unless the venue family is explicitly non-canonical.
- **Aerodrome V2 classic is not UniV3.** Use exact factory/pool semantics. Do not fake generic CREATE2 assumptions for Aerodrome V2 classic pools.
- **Exact-output is separate.** Do not force exact-output math into the forward exact-input loop. Reject it unless a separate reverse mode is intentionally implemented.
- **Real early-abort guards:** concentrated-liquidity legs must use real `sqrtPriceLimitX96` values, not permanent min/max sentinels.
- **V2 dust harvesting:** exact-in V2 style legs must capture surplus with realized balance deltas.
- **Hot-path transfers:** blind low-level `call` without `extcodesize` is acceptable only for pre-vetted tokens/pools in the hot path. Cold/admin paths may use `SafeERC20`.
- **Profitability truth:** realized token balance deltas only. No quote truth.

## ABI certainty rule

Fail closed on ABI uncertainty. Never invent:
- Balancer compatibility
- generic Slipstream compatibility
- router semantics
- callback surfaces you cannot justify from exact interfaces

## Transient-storage law

If transient storage is used:
- compile with Solidity `>= 0.8.34`
- target a Cancun-compatible EVM
- define lock enum values once
- the value written before an external call must be the exact value required in the callback
- clear transient locks with explicit `tstore(slot, 0)`
- do not use `delete` on transient variables

## Session discipline

Before editing code, freeze architecture decisions.
After each patch, run the self-audit.
If a self-audit fails, fix the failure before doing anything else.
