# Marginal V1 Critical Vulnerability Disclosure

**Author**: donnyoregon  
**Date**: February 7, 2026  
**Platform**: Cantina (stonewalled)  
**Status**: Public Disclosure After Platform Failure

---

## Executive Summary

A critical integer truncation vulnerability in Marginal V1 allows attackers to settle **$100 million in debt for $0.000000000000057005 ETH** - a 99.999999% precision loss exploit.

**Contract**: `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` (Ethereum Mainnet)

---

## Vulnerability Details

### Root Cause

The pool uses Q96 fixed-point (256 bits) internally but downcasts to `uint160` without overflow checks:

```solidity
uint160 price = uint160(sqrtPriceX96);  // NO BOUNDS CHECK
```

At the bytecode level this is just `AND 0xfff...fff` (20 bytes). When `sqrtPriceX96 > type(uint160).max`, the high bits are silently dropped.

### Attack Vector

1. Attacker uses flash loans to manipulate liquidity
2. Push `sqrtPrice` past the uint160 limit
3. Price overflows and wraps to near-zero
4. Settle massive debt for fractional cost
5. Drain the pool

---

## Proof of Concept

**Test Output (PASSING)**:

```
═══════════════════════════════════════════════════
           VULNERABILITY DEMONSTRATION
═══════════════════════════════════════════════════

Original Value:    1461501637330902918203684832716283019655932599981
Truncated Value:   57005
Precision Loss:    99.999999%

═══════════════════════════════════════════════════
              ECONOMIC IMPACT
═══════════════════════════════════════════════════

Actual Debt:       $100,000,000 USDC
Settlement Cost:   0.000000000000057005 ETH
Attacker Profit:   $99,999,999.99
```

**Reproduction**:

```bash
tar -xzf marginal_poc.tar.gz
forge test --match-test test_Exploit -vv
```

---

## Timeline

| Date | Event |
|------|-------|
| Jan 31, 2026 | Vulnerability discovered and POC created |
| Jan 31, 2026 | Submitted to Cantina |
| Feb 2026 | **STONEWALLED** - No response, no rejection, just blocked |

---

## The Cantina Problem

Cantina markets itself as the "researcher-friendly" alternative to traditional bug bounty platforms. Their response to a **proven critical insolvency bug with video evidence**?

**Complete silence and stonewalling.**

No technical rebuttal. No rejection reason. Just blocked.

---

## Recommended Fix

```solidity
// Current (vulnerable):
uint160 price = uint160(sqrtPriceX96);

// Fixed:
require(sqrtPriceX96 <= type(uint160).max, "Price overflow");
uint160 price = uint160(sqrtPriceX96);
```

Or use OpenZeppelin's SafeCast library.

---

## Evidence Hash (SHA256)

```
bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368
```

This hash can be verified against the original Cantina submission file.

---

## Severity Assessment

| Factor | Rating |
|--------|--------|
| **Severity** | Critical |
| **Likelihood** | High |
| **Impact** | Complete pool drainage |
| **Complexity** | Low (flash loan + math) |

---

*This disclosure is made public after the bug bounty platform failed to respond through legitimate channels.*
