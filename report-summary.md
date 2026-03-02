# Marginal V1 Critical Vulnerability: Stealth Patch & Theft of Work Disclosure

**Author**: donnyoregon  
**Date**: February 7, 2026 (Updated March 1, 2026)  
**Platform**: Cantina (Stonewalled)  

---

## Executive Summary

A critical integer truncation vulnerability was identified in the Marginal V1 Protocol (`MarginalV1Pool` on Ethereum Mainnet) that allowed attackers to settle **$100 million in debt for less than a cent** (a 99.999999% precision loss exploit).

The protocol uses a Q96 fixed-point number internally (256 bits) but downcasts it to `uint160` without overflow checks:
```solidity
uint160 price = uint160(sqrtPriceX96); // NO BOUNDS CHECK
```

When an attacker manipulates the pool to push `sqrtPriceX96` past `type(uint160).max`, the EVM silently drops the high bits (`AND` mask), causing the protocol's debt calculation to wrap around to near-zero. 

## The Stealth Patch & Cantina Failure

This vulnerability was reported to the Marginal V1 team via Cantina on **January 31, 2026**. 

The Cantina triage team (including Mike Leffer) dismissed the report, stating the vulnerability was impossible due to "Gnosis Safe" limits and other extraneous logic that did not actually prevent the exploit path.

Days later (Feb 4), the Marginal V1 team performed a **stealth patch** to their mainnet pool, injecting the exact `SafeCast` bounds-checking logic recommended in the initial report:

```solidity
// Injected into the shadow-patched bytecode:
"SafeCast: value doesn't fit in 160 bits"
```

To cover their tracks, the team obfuscated the standard proxy implementation pointers by zeroing out the EIP-1967 slots, hiding the upgrade path from blockchain explorers while retaining the new, secure logic. 

**This is a formal disclosure of Theft of Work and Security Misconduct by the Marginal V1 team and a catastrophic failure of the Cantina triage process.**