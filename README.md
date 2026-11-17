# CVE-2026-4931: Unsafe uint160 Downcast in Marginal V1

**Researcher:** Corrin Clark ([@donnyoregon](https://github.com/donnyoregon)) · clarkcorrin@gmail.com

| Field | Value |
|---|---|
| CVE | [CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931) |
| NVD score | CVSS v3.1 8.6 High (`AV:N/AC:L/PR:N/UI:N/S:C/C:N/I:H/A:N`) |
| CERT/CC | [VU#643748](https://kb.cert.org/vuls/id/643748) |
| Weakness | CWE-681, Incorrect Conversion between Numeric Types |
| Target | Marginal V1 pool, [`0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7`](https://etherscan.io/address/0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7) (Ethereum mainnet) |
| Fixed implementation | [`0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8`](https://etherscan.io/address/0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8), block 24,386,649 |
| Status | Patched |

## Summary

Marginal V1 downcast a 256-bit Q96 price value to `uint160` without a bounds check. Values above `type(uint160).max` were silently truncated instead of reverting, which let a debt position be settled for a negligible amount of the asset.
CERT/CC's description: *"Smart contract Marginal v1 performs unsafe downcast, allowing attackers to settle a large debt position for a negligible asset cost."*

## Root cause

```solidity
uint160 price = uint160(sqrtPriceX96); // no bounds check
```

In bytecode the cast compiles to a mask (`AND 0xff…ff`), so the high bits are dropped and execution continues.

## Attack path

1. Move pool liquidity with standard flash-loan primitives.
2. Push `sqrtPriceX96` past the `uint160` limit.
3. The value wraps to a near-zero integer.
4. Settle a large debt position at a fraction of its cost.

## Proof of concept

Reproduced in Foundry against a 1:1 fork of live Ethereum mainnet state at the vulnerable block, using the deployed contracts, not mocks. See [`full_exploit_code.t.sol`](full_exploit_code.t.sol) and [`marginal_poc_output.txt`](marginal_poc_output.txt).

```
Original value:     1461501637330902918203684832716283019655932599981
Truncated value:   57005
Precision loss:     99.999999%
```

## Fix

Use OpenZeppelin `SafeCast.toUint160()` (or an explicit bounds check) so out-of-range values revert. The deployed fix includes the `SafeCast` revert string; see [`bytecode-safe-cast/`](bytecode-safe-cast/) and the [storage slot diff](storage-slot-diff/slot_6_diff.md).

## Timeline

| Date (2026) | Event |
|---|---|
| Jan 31 | Reported with PoC and SafeCast recommendation |
| Feb 2 | Protocol paused |
| Feb 4 | Fixed implementation deployed ([tx](https://etherscan.io/tx/0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a)) |
| Feb 10 | Bounty submission closed without reward |
| Feb 17 | Submitted to CERT/CC |
| Apr 7 | CVE-2026-4931 published |

## Method

Found by reading deployed EVM bytecode directly and diffing storage across the upgrade block, then confirming on a 1:1 mainnet-state fork of the deployed contracts.

## References

- [NVD: CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931)
- [CERT/CC VU#643748](https://kb.cert.org/vuls/id/643748)
- [MarginalProtocol/v1-core#10](https://github.com/MarginalProtocol/v1-core/issues/10)

---

Available for smart contract security reviews (EVM and Move). Contact: clarkcorrin@gmail.com
