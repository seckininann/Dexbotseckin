# Forbidden Patterns

These patterns are not automatically wrong everywhere, but they are **red alerts** for this executor and must be justified or removed.

## Hard-forbidden in the final hot path

- `abi.encode(...)` carrying route or callback state
- `abi.decode(...)` for route or callback state
- scalar ABI hot-path arguments for `tokenA`, `amount`, `minProfit`
- explicit canonical UniV3 pool addresses in calldata
- exact-output branches inside the same simple forward exact-input loop
- CL legs without real `sqrtPriceLimitX96` protection
- transient lock values that can be interpreted differently by different branches
- `delete` on transient state
- fabricated router/periphery usage
- fabricated Balancer / Slipstream compatibility

## Usually-forbidden in the hot path

- `SafeERC20` in swap/flash/callback logic
- `bytes memory` for route transport
- generic variable-length trailer walking for every leg
- extcodesize-style existence checks before every blind ERC20 hot-path call

## Allowed with explicit scope and explanation

- `SafeERC20` in owner/admin/recovery paths
- explicit-address override modes for non-canonical pools
- rejection of entire venue families when ABI certainty is missing
- rejecting exact-output entirely in v1
- using an intermediate `execute(bytes)` only during development if the final goal remains raw `fallback()`

## Red-flag grep strings

```text
abi.encode(
abi.decode(
bytes memory
new bytes
SafeERC20
function execute(
tokenA,
minProfit
sqrtPriceLimitX96
delete
exactOutput
```

## Review rule

If a red-flag string appears, inspect whether it is:
- hot path
- cold path
- temporary debug code
- a genuine final-production compromise

If it is a hot-path regression, remove it.
