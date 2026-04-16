@base-mev-executor
/build-base-mev-executor

Goal: build or refactor the Base Mainnet arbitrage executor as a single-contract, direct-pool, calldata-minimized MEV executor.

Before writing code:
1. Read the repo `AGENTS.md`.
2. Read the full `@base-mev-executor` Skill files.
3. Inspect the current executor and produce a defect map.
4. Fill and freeze the decision table from `02-freeze-decisions.md`.
5. Show the lock truth table from `03-lock-matrix.md`.

Non-negotiable invariants:
- no hot-path `abi.encode(...)` or `abi.decode(...)` for route/callback state
- no scalar ABI hot-path args for `tokenA`, `amount`, `minProfit`
- prefer raw `fallback()` parsing `msg.data`
- canonical UniV3 pools must be derived on-chain instead of carried as 20-byte calldata addresses
- Aerodrome V2 classic must use its own family semantics, not fake UniV3 assumptions
- exact-output must be rejected or moved to a separate reverse mode
- every CL leg must use a real `sqrtPriceLimitX96` guard
- V2 exact-in legs must harvest surplus via realized balance deltas
- transient lock values must be set and checked with an exact one-to-one mapping
- clear transient state with explicit `tstore(slot, 0)`, not `delete`
- hot-path ERC20 transfers may use blind low-level calls for pre-vetted tokens
- fail closed on unsupported ABI uncertainty

Work in phases and self-audit after each phase.
Do not jump straight into code.
Do not invent compatibility.
If any invariant conflicts with the current code, preserve the invariant and rewrite the code.
