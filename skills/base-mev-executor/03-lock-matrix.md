# Transient Lock Matrix

The lock matrix exists to make callback authorization mechanically impossible to confuse.

## Canonical enum

Use one numeric mapping everywhere. Do not reinterpret values in different branches.

- `0 = UNLOCKED`
- `1 = EXPECT_UNIV3_FLASH_CALLBACK`
- `2 = EXPECT_UNIV3_SWAP_CALLBACK`
- `3 = EXPECT_EXTERNAL_FLASH_CALLBACK`
- `4 = INTERNAL_NO_CALLBACK_EXPECTED` (optional; only if genuinely useful)

## Core law

The value written immediately before an external call must be the exact value required in the corresponding callback.

### Correct examples

- Before `pool.flash(...)`: write `EXPECT_UNIV3_FLASH_CALLBACK`
- Inside `uniswapV3FlashCallback(...)`: require `EXPECT_UNIV3_FLASH_CALLBACK`
- Before `pool.swap(...)` that will callback: write `EXPECT_UNIV3_SWAP_CALLBACK`
- Inside `uniswapV3SwapCallback(...)`: require `EXPECT_UNIV3_SWAP_CALLBACK`

### Incorrect examples

- write `1`, require `2`
- write a lock in one branch and let another branch interpret the same value differently
- clear the lock before repayment/callback settlement is complete

## Required callback checks

Every callback must verify:
1. lock kind matches
2. expected caller matches `msg.sender`
3. active route/execution context exists
4. the call occurs at a valid execution phase

## Clear policy

Clear the lock explicitly with `tstore(slot, 0)` only when the expected callback has fully completed.

Do not:
- use `delete` on transient state
- leave the lock uncleared for “gas savings”
- share one lock value across unrelated callback families without a caller check

## Suggested supporting context words

In addition to lock kind, maintain compact context for:
- expected caller
- active route hash or route pointer discipline
- current leg index / phase
- profit token / repayment token if needed

## Review question to ask after every patch

“For every external call that may callback, can I point to the exact numeric value written before the call and the exact matching require inside the callback?”
