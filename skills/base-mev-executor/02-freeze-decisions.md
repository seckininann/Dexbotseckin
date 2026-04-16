# Freeze Decisions Before Code

Fill this table before patching the executor. No critical row may remain undecided.

| Decision | Allowed values | Frozen value | Status | Why |
|---|---|---|---|---|
| Final hot-path entrypoint | `fallback()` / `execute(bytes)` / other |  |  |  |
| Packed amount/minProfit location | first 32-byte header / other |  |  |  |
| Hot-path route parsing | raw calldata offsets / other |  |  |  |
| Callback payload transport | direct calldata math / route pointer discipline / other |  |  |  |
| Transient lock enum | explicit numeric table |  |  |  |
| Transient clear method | explicit `tstore(slot,0)` / other |  |  |  |
| Compiler floor | `0.8.34+` / other |  |  |  |
| EVM target when using TSTORE | `cancun` / other |  |  |  |
| Canonical UniV3 pool resolution | CREATE2 derive / explicit-address mode / mixed |  |  |  |
| Aerodrome V2 pool resolution | exact factory semantics / other |  |  |  |
| Slipstream support | reject / explicit exact ABI support |  |  |  |
| Balancer-style flash support | reject / exact verified lender ABI |  |  |  |
| Exact-output support | reject / separate reverse mode |  |  |  |
| CL price limit source | packed route guard / other |  |  |  |
| Hot-path ERC20 transfer mode | blind low-level call / mixed / SafeERC20 |  |  |  |
| V2 surplus harvesting mode | pre/post balance delta / other |  |  |  |
| Profitability truth | realized balance delta / other |  |  |  |

## Hard rule

If a row cannot be frozen honestly, reduce scope and reject the corresponding feature. Do not leave it ambiguous and code anyway.
