---
name: base-mev-executor
description: Design, review, or refactor a Base Mainnet direct-pool arbitrage executor under extreme calldata, callback, and hot-path constraints. Use for route encoding, fallback entrypoints, CREATE2 pool derivation, transient lock design, callback safety, V2 dust harvesting, and final self-audit.
---

# Base MEV Executor Skill

This Skill exists because the task is too large and too low-level for a single conversational prompt.

Use this Skill whenever the task involves any of the following:
- building the executor from scratch
- refactoring an existing executor
- freezing the route format
- designing the callback state machine
- removing calldata bloat
- replacing hot-path safety overhead with pre-vetted blind-call helpers
- deciding which venue families are truly supported
- auditing whether the current code has drifted back into default Solidity patterns

## Mandatory operating sequence

Read these files in order before editing code:

1. `01-constitution.md`
2. `02-freeze-decisions.md`
3. `03-lock-matrix.md`
4. `04-route-and-entry-policy.md`
5. `05-forbidden-patterns.md`
6. `06-self-audit.md`

## Required outputs in every serious session

Before code:
- defect map
- frozen decision table
- lock truth table
- venue-family support matrix

After code:
- forbidden-pattern report
- self-audit report
- residual-gap summary

## Fail-closed law

If exact ABI certainty is missing for any protocol family, reject the feature instead of inventing support.

## Forbidden drift

Stop immediately if the code drifts into any of the following:
- `abi.encode(...)` / `abi.decode(...)` carrying route or callback state
- a scalar-ABI hot path instead of packed/raw bytes
- canonical UniV3 pool addresses still in calldata
- exact-output forced into the forward exact-input loop
- missing `sqrtPriceLimitX96` guards on CL legs
- mismatched transient lock set/check values
- `delete` on transient state
- fabricated Balancer / Slipstream compatibility

## Final standard

The correct answer is not “whatever compiles”.
The correct answer is the strongest design that preserves the invariants, fails closed on uncertainty, and remains honest about residual competitive gaps.
