# Windsurf Control Plane for a Base Mainnet MEV Executor

This package is built to stop Windsurf from drifting back to default Solidity patterns while you build a top-tier Base Mainnet arbitrage executor.

The core idea is simple:

1. **Keep always-on instructions short.** Root `AGENTS.md` and always-on Rules are injected frequently, so they should contain only non-negotiable invariants.
2. **Move the heavy spec into a Skill.** This is where the long MEV architecture, lock matrix, binary-layout rules, and self-audits live.
3. **Use a manual Workflow for trajectory control.** This prevents the agent from jumping straight into code before freezing the architecture.
4. **Pin only critical files.** Pin the root `AGENTS.md`, the `.windsurf/skills/base-mev-executor/` folder, the current contract file, and any exact ABI/interface files you want respected. Do **not** pin the whole repo.

## Install

Copy this package into the root of your Windsurf workspace so the final layout looks like:

- `AGENTS.md`
- `.windsurf/rules/*.md`
- `.windsurf/workflows/*.md`
- `.windsurf/skills/base-mev-executor/*`

## How to use it

1. Open the workspace in Windsurf.
2. Pin:
   - `AGENTS.md`
   - `.windsurf/skills/base-mev-executor/`
   - your target executor contract file
   - the exact external ABI/interface files you want the model to obey
3. Invoke the Skill: `@base-mev-executor`
4. Run the manual Workflow: `/build-base-mev-executor`
5. Paste the kickoff prompt from `.windsurf/skills/base-mev-executor/07-kickoff-prompt.md`

## Why this package is structured this way

- `AGENTS.md` keeps durable, always-on repo laws extremely short.
- `.windsurf/rules/00-base-mev-axioms.md` reinforces cross-cutting architectural invariants.
- `.windsurf/rules/10-solidity-hotpath.md` applies only when Windsurf is actually editing Solidity files.
- `.windsurf/skills/base-mev-executor/` holds the long-form MEV specification and audit documents.
- `.windsurf/workflows/build-base-mev-executor.md` forces a deterministic sequence: inspect -> freeze decisions -> patch -> self-audit.

## What this package protects against

- memory-copy regressions (`abi.encode`, `abi.decode`, hidden calldata->memory copies)
- broken transient lock state machines
- calldata bloat from scalar ABI arguments and explicit canonical CL pool addresses
- exact-output hallucinations inside a forward exact-input loop
- hardcoded `sqrtPriceLimitX96` sentinels instead of real early-abort guards
- wasted hot-path gas from `SafeERC20`/`extcodesize` patterns on pre-vetted tokens
- fabricated Balancer / Slipstream / periphery compatibility

## Strong recommended compiler floor

If you use transient storage, pin the compiler to **Solidity 0.8.34 or newer** and target a **Cancun-compatible EVM**. The package assumes explicit `tstore(slot, 0)` style clearing rather than `delete` on transient state.

## Final operating principle

If there is any conflict between current code and the invariant files in this package, Windsurf must preserve the invariant and rewrite the code.
