# Route and Entry Policy

This file defines how the executor should think about packed calldata, entrypoints, and address compression.

## Preferred entrypoint

**Preferred final production shape:** raw `fallback()` parsing `msg.data`.

Why:
- avoids the 4-byte selector + dynamic-bytes offset/length framing of `execute(bytes)`
- avoids scalar ABI padding for `tokenA`, `amount`, `minProfit`
- keeps the external hot path maximally compact

Acceptable only with explicit written justification:
- `execute(bytes calldata routeData)` during an intermediate/debugging stage

## Header policy

`amount` and `minProfit` belong in the packed header, not as separate 32-byte scalar ABI parameters.

Minimum requirement:
- first 32-byte word packs routing mode + leg count + flags + amount + minProfit
- any extra global fields should be justified against calldata cost

## Canonical CL address policy

For canonical UniV3-style pools:
- do **not** carry the 20-byte pool address in calldata
- derive on-chain from:
  - token continuity / current input-output pair
  - sorted tokens
  - factory
  - fee tier or equivalent compact selector
  - init-code-hash

Allowed exception:
- a clearly marked non-canonical override mode for pools that cannot be derived deterministically

## Aerodrome V2 classic policy

Aerodrome V2 classic should not be forced into the same canonical CREATE2 pattern. Use the family’s exact factory/pool verification semantics instead.

## Trailer policy

Avoid generic per-leg variable-length trailer walking in the final hot path unless the win is overwhelming.

Preferred alternatives:
- fixed-width leg layouts per family
- mode-specific parsers
- branch-local offsets that stay deterministic
- compact lookup/selector fields instead of arbitrary trailing blobs

The problem with a generic trailer is not correctness; it is hot-path pointer arithmetic complexity.

## Exact-output policy

Default stance:
- reject exact-output in the first contract

Acceptable extension:
- a separate reverse dispatcher / mode with explicitly different state-machine logic

Forbidden:
- pretending exact-output can be made safe and simple inside the same forward loop used for exact-input

## Price-limit policy

Every CL leg must carry enough information to build a real `sqrtPriceLimitX96` guard.

Forbidden final pattern:
- using permanent global min/max sentinels that effectively disable early abort

## Dust-harvest policy

V2-style exact-in legs must account for realized deltas so surplus is not stranded by a too-conservative output specification.
