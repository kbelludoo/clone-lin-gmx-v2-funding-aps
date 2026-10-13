# clone-lin-gmx-v2-funding-aps

Experimental i64-scale LIN clone of GMX V2 `MarketUtils.getNextFundingAmountPerSize` / `getFundingAmountPerSizeDelta` / `getPoolDivisor` / `Precision.applyFactor`.

**Proofs and evidence live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open)** (`src/lin_gmx_v2_funding_aps.lin`, `test/prove_gmx_v2_funding_aps_external.py`, `examples/gmx_v2_funding_aps/`). This repository is the separate LIN copy only.

## Kernel (EXPERIMENTAL, not mainnet uint256, not FLOAT=1e30)

```
FLOAT' = 1e6
SQRT'  = 1e3
STORE' = FLOAT' * SQRT' = 1e9
if longOI==0 || shortOI==0: fundingUsd = 0
size = longsPayShorts ? longOI : shortOI
fundingUsd = size * duration * fps / FLOAT' / divisor
divisor = sameToken ? 2 : 1
split = fundingUsd * colOI / payingOI
aps = (fundingUsd * STORE' / OI) / price   # round-up on fee path
```

This is **not** V1 `getNextFundingRate` (factor*reserved*intervals/pool). DataStore duration/fps are arguments.

Named contrasts (recomputed in lin-open against C11 `__int128`, limb muldiv, LIN Compiler 0 VM, Python bigint, and libtcc `roundtrip-jit`):

| Path | Result |
|---|---|
| paying long 2e8, dur 4000, fps 10, two-token | fundingUsd `8e6` |
| same, single-token divisor 2 | `4e6` |
| ignore divisor | `8e6` (2x charge) |
| long OI 0 | `0` |
| wrong still-charge shorts | `4e6` |
| forgotten fps | `8e5` (10x) |
| split $500 on 1000/4000 of 5000 | `100` / `400` |
| fee aps $100 / $2000 / $10k | `5000` |
| claimable on $5k | `10000` |
| naive i64 `7e9*2e9/1e6` | wraps; 128-bit `14000000000000` |

## Upstream

- https://github.com/gmx-io/gmx-synthetics `a85ea3491c19c93bb4b5a002d9b358fb769b7849`
- `contracts/market/MarketUtils.sol` blob `ad7a0b2b5a516ec86ce0afed89ff84ecba11038c`
- `contracts/utils/Precision.sol` blob `047143722beaf6adceac3fc2ef1817f054dfe00a`
- License: GPL-2.0-or-later after BUSL Change Date 2026-08-31

## Files

- `src/lin_gmx_v2_funding_aps.lin` — LIN kernel (`g2f_*`)
- `gmx_v2_funding_aps_c11.c` — independent C11 oracle (selftest)
