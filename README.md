# CVE-2026-4931: Marginal V1 Critical Integer Truncation

![CVSS 9.1 Critical](https://img.shields.io/badge/CVSS%20v3.1-9.1%20Critical-red)
![CVE-2026-4931](https://img.shields.io/badge/CVE-2026--4931-critical)
![CERT/CC VU#643748](https://img.shields.io/badge/CERT%2FCC-VU%23643748-blue)

| Field | Value |
|---|---|
| **CVE** | CVE-2026-4931 |
| **CVSS v3.1 Score** | **9.1 (Critical)** |
| **CVSS v3.1 Vector** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N` |
| **CERT/CC Case** | VU#643748 |
| **Author** | Corrin Clark (donnyoregon / arch) |
| **Report Date** | January 31, 2026 |
| **Target Protocol** | Marginal V1 |
| **Vulnerable Contract** | `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` (Ethereum Mainnet) |

### CVSS v3.1 Metric Breakdown

| Metric | Value | Rationale |
|---|---|---|
| Attack Vector (AV) | **Network** | Exploitable remotely via flash loan |
| Attack Complexity (AC) | **Low** | Standard DeFi primitives, no special conditions |
| Privileges Required (PR) | **None** | No account or on-chain permissions needed |
| User Interaction (UI) | **None** | Fully autonomous attack |
| Scope (S) | **Changed** | Attack impacts pool lenders beyond the attacker's own component |
| Confidentiality (C) | **High** | Complete extraction of pooled funds |
| Integrity (I) | **High** | Corrupts all protocol accounting |
| Availability (A) | **None** | N/A (financial drain, not service disruption) |

---

## Executive Summary
This repository contains the forensic disclosure for a critical integer truncation vulnerability in Marginal V1. The vulnerability allowed an attacker to bypass protocol accounting via an unsafe downcast, enabling the settlement of multi-million dollar debt positions for practically zero cost. 

Despite the protocol pausing operations and deploying the exact mitigation recommended in the initial report, the mediating bug bounty platform, Cantina, rejected the finding.

Following this rejection, the vulnerability and accompanying on-chain evidence were submitted to CERT/CC. After independent verification, CERT/CC confirmed the vulnerability and issued CVE-2026-4931.

## Official CERT/CC Validation
> "Smart contract Marginal v1 performs unsafe downcast, allowing attackers to settle a large debt position for a negligible asset cost." 
> — *CERT/CC Vulnerability Note VU#643748*

## Vulnerability Details

### Root Cause
The pool uses Q96 fixed-point (256 bits) internally but downcasts to `uint160` without overflow checks during price calculations:

```solidity
uint160 price = uint160(sqrtPriceX96);  // NO BOUNDS CHECK
At the bytecode level, this functions as a simple bitmask (AND 0xfff...fff). When sqrtPriceX96 > type(uint160).max, the high bits are silently dropped. The transaction does not revert.
Attack Vector
Attacker utilizes standard flash loans to manipulate pool liquidity.
The manipulation pushes the sqrtPrice past the uint160 limit.
The price overflows and wraps to a near-zero integer.
The attacker settles a massive debt position for a fractional cost, draining the pool.
Proof of Concept Output
To ensure proper testing for Web3 ethical bug bounty hunting, the execution was recorded on a local fork of the Ethereum Mainnet.
Execution Trace Logs:═══════════════════════════════════════════════════
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
The Timeline of Events
January 31, 2026: Vulnerability reported to Cantina with full proof-of-concept.
February 02, 2026: Marginal V1 pauses the protocol.
February 04, 2026: Fix deployed to Ethereum Mainnet using the recommended SafeCast library in transaction 0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a.
February 10, 2026: Cantina rejects the finding.
February 17, 2026: Cantina Support officially classifies the proxy upgrade as an "MEV swap." Forensic evidence submitted to CERT/CC.
February 19, 2026: CERT/CC validates the vulnerability.
March 28, 2026: CVE-2026-4931 is officially assigned.
Repository Contents
This repository serves as the public record for the vulnerability and the subsequent governance failure by the mediating platform.
run_exploit.sh - Automated bash script containing the Foundry proof-of-concept demonstrating the complete extraction of pool liquidity on a mainnet fork.
CANTINA_TRIAGE_LOG.md - Raw transcript of the Cantina triage process demonstrating their contradictory administrative classifications.
CERT_VALIDATION.md - Documentation of the federal vulnerability assignment.
/bytecode-analysis/ - Decompiled bytecode proving the addition of the SafeCast library in the patched implementation 0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8.
Evidence Hash (SHA256)
The following hash can be verified against the original Cantina submission file to establish the cryptographic timeline of the initial report:
bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368
Statement on Bug Bounty Governance
Bug bounty platforms exist to act as neutral arbiters between researchers and protocols. When a platform dismisses on-chain cryptographic proof of a remediation and rejects a federally validated CVE, the integrity of the ecosystem is compromised. This disclosure is published to ensure transparency and accountability in Web3 security practices.