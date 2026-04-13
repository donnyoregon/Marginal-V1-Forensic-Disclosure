# Security Policy

## CVE-2026-4931 — Critical Integer Truncation in Marginal V1

### Severity

| Attribute | Value |
|---|---|
| **CVSS v3.1 Score** | **9.1 (Critical)** |
| **CVSS v3.1 Vector** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N` |
| **CVE ID** | CVE-2026-4931 |
| **CERT/CC Case** | VU#643748 |
| **CWE** | CWE-197: Numeric Truncation Error |
| **Affected Contract** | `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` (Ethereum Mainnet) |
| **Status** | **Patched** (block 24,386,649 — Feb 04, 2026) |

### CVSS v3.1 Metric Details

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N
```

- **AV:N** — Network: Attacker exploits remotely using universally available flash loans
- **AC:L** — Low Complexity: Standard DeFi flash loan primitives; no special conditions
- **PR:N** — No Privileges Required: Any external account can execute the attack
- **UI:N** — No User Interaction: Fully autonomous, single-transaction attack
- **S:C** — Scope Changed: Impact extends to all pool liquidity providers and lenders
- **C:H** — High Confidentiality: Complete extraction of deposited funds
- **I:H** — High Integrity: Full corruption of debt accounting (99.999999% precision loss)
- **A:N** — Availability: Not applicable (economic drain, not service outage)

### Vulnerability Description

The `MarginalV1Pool` contract uses Q96 fixed-point arithmetic (256-bit) internally
but downcasts to `uint160` without overflow protection during settlement calculations:

```solidity
uint160 price = uint160(sqrtPriceX96);  // NO BOUNDS CHECK
```

At the EVM bytecode level, this is a bitmask operation (`AND 0xfff...fff` for 160 bits).
When `sqrtPriceX96` exceeds `type(uint160).max`, the high bits are silently dropped and
the transaction does not revert.

An attacker uses flash loans to push `sqrtPrice` past the `uint160` overflow threshold,
causing the protocol to calculate a $100,000,000 debt as worth `0.000000000000057005 ETH`.

### Proof of Concept Result

```
Original Value:    1461501637330902918203684832716283019655932599981
Truncated Value:   57005
Precision Loss:    99.999999%

Actual Debt:       $100,000,000 USDC
Settlement Cost:   0.000000000000057005 ETH
Attacker Profit:   $99,999,999.99
```

### Remediation

The Marginal V1 team deployed a patch at block 24,386,649 (Feb 04, 2026) using
OpenZeppelin's `SafeCast` library (`SafeCast: value doesn't fit in 160 bits`),
confirmed on-chain in transaction:
`0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a`

The correct fix is:

```solidity
// Use SafeCast instead of direct downcast:
uint160 price = SafeCast.toUint160(sqrtPriceX96);
// or:
require(sqrtPriceX96 <= type(uint160).max, "Price overflow");
uint160 price = uint160(sqrtPriceX96);
```

### References

- [CERT/CC VU#643748](https://kb.cert.org/vuls/id/643748)
- [NVD CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931)
- [GitHub Issue MarginalProtocol/v1-core#10](https://github.com/MarginalProtocol/v1-core/issues/10)
- [Forensic Disclosure Repository](https://github.com/donnyoregon/Marginal-V1-Forensic-Disclosure)
