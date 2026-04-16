# Build Base MEV Executor

Use this workflow when creating or refactoring the Base Mainnet arbitrage executor.

## Step 0 — Load the right context

Read in this order:
1. `AGENTS.md`
2. `@base-mev-executor`
3. the current target contract file
4. the exact ABI/interface files for any supported external protocol family

Do **not** start patching until you have read the Skill resources.

## Step 1 — Produce a defect map

Inspect the current code and write a defect map under these headings:

- calldata bloat
- hidden memory copies
- broken callback state machine
- venue-family ABI uncertainty
- missing early-abort logic
- hot-path transfer waste
- V2 dust loss
- exact-output / forward-loop contamination
- remaining bounded compromises

## Step 2 — Freeze the architecture

Fill the decision table from:
- `.windsurf/skills/base-mev-executor/02-freeze-decisions.md`

Every row must end as one of:
- `FROZEN`
- `REJECTED`
- `OUT OF SCOPE`

Do not write code while any critical row is undecided.

## Step 3 — Patch in this order

1. final entrypoint strategy (`fallback()` preferred)
2. raw packed-header parsing
3. lock truth table / transient context
4. canonical UniV3 pool derivation
5. flash-source entry
6. UniV3 swap callback settlement
7. Aerodrome V2 exact-in path with surplus harvesting
8. profitability firewall
9. owner/executor + cold recovery
10. residual honesty comments / limitations

## Step 4 — Self-audit after each subsystem

Use:
- `.windsurf/skills/base-mev-executor/06-self-audit.md`

If any forbidden pattern remains, fix it before moving on.

## Step 5 — Final output package

Before you call the contract finished, produce:

- defect map
- frozen decisions table
- final contract
- final self-audit report
- explicit list of bounded compromises that still remain

## Suggested grep checks

Adapt the paths if your repo layout differs.

```bash
rg -n "abi\.encode\(" src contracts .
rg -n "abi\.decode\(" src contracts .
rg -n "function\s+execute\s*\(" src contracts .
rg -n "SafeERC20" src contracts .
rg -n "sqrtPriceLimitX96" src contracts .
rg -n "\bdelete\b" src contracts .
rg -n "exactOutput|reverse" src contracts .
```

## Workflow stop conditions

Stop and reconsider the architecture immediately if any of these are still true:
- the route still enters callbacks through `bytes memory`
- the lock enum can be mis-set or mis-checked
- canonical CL pool addresses are still carried in calldata
- exact-output is still inside the same forward loop
- CL legs still use effectively disabled price limits
- hot-path transfers still burn unnecessary safety overhead
