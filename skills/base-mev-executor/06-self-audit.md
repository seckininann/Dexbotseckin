# Self-Audit Checklist

Run this audit after every subsystem patch and again at the end.

## A. Entry and calldata

- [ ] Final hot path prefers raw `fallback()` or has a written reason for not doing so
- [ ] `amount` and `minProfit` are packed into the route header, not separate ABI scalars
- [ ] Route parsing uses direct calldata/math rather than `abi.decode`
- [ ] No hidden route copy to memory exists in callbacks

## B. Canonical pool compression

- [ ] Canonical UniV3-style pools are derived on-chain rather than passed as 20-byte calldata addresses
- [ ] Any explicit-address override mode is clearly marked and justified
- [ ] Aerodrome V2 classic uses the right family-specific semantics instead of fake generic CREATE2 assumptions

## C. Lock and callback safety

- [ ] Every external call that can callback writes the correct lock value first
- [ ] Every callback requires the exact matching lock value
- [ ] Every callback verifies exact `msg.sender`
- [ ] Every callback verifies the active execution phase/context
- [ ] Locks are cleared explicitly with `tstore(slot, 0)` only after the callback completes

## D. Execution semantics

- [ ] Exact-output is rejected or moved to a separate reverse mode
- [ ] V2 exact-in legs harvest surplus with realized balance deltas
- [ ] CL legs use real `sqrtPriceLimitX96` guards
- [ ] Profitability uses realized token balances and exact repayment obligations

## E. Hot-path gas posture

- [ ] No unnecessary `SafeERC20` remains in the hot path
- [ ] No habitual extcodesize-style check remains in blind hot-path transfer helpers unless explicitly justified
- [ ] No generic trailer-walking complexity remains unless it clearly wins overall

## F. ABI honesty

- [ ] No fabricated Balancer compatibility is claimed
- [ ] No fabricated Slipstream compatibility is claimed
- [ ] No router/periphery dependency exists

## Suggested grep checks

Adapt paths as needed.

```bash
rg -n "abi\.encode\(" src contracts .
rg -n "abi\.decode\(" src contracts .
rg -n "bytes memory|new bytes" src contracts .
rg -n "SafeERC20" src contracts .
rg -n "function\s+execute\s*\(" src contracts .
rg -n "tokenA|minProfit" src contracts .
rg -n "sqrtPriceLimitX96" src contracts .
rg -n "\bdelete\b" src contracts .
rg -n "exactOutput|reverse" src contracts .
rg -n "router|SwapRouter|UniversalRouter|IAerodromeRouter" src contracts .
```

## Final audit question

“If I strip away all comments and optimism, does the resulting contract still obey the lock matrix, avoid hidden memory copies, keep calldata compressed, and fail closed on unsupported ABI assumptions?”
