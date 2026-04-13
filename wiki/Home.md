# CVE-2026-4931: Marginal V1 Forensic Disclosure — Wiki Home

> **Status: Patched** | **CVSS 9.1 Critical** | **CVE-2026-4931** | **CERT/CC VU#643748**

This wiki is the complete reference for the forensic disclosure of a critical integer truncation vulnerability in Marginal V1. It covers the vulnerability itself, the on-chain proof of exploitation, the stealth patch evidence, the Cantina bug bounty platform's failure, and the subsequent federal validation by CERT/CC.

---

## At a Glance

| Field | Value |
|---|---|
| **CVE** | CVE-2026-4931 |
| **CWE** | CWE-197: Numeric Truncation Error |
| **CVSS v3.1 Score** | **9.1 (Critical)** |
| **CVSS v3.1 Vector** | `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N` |
| **CERT/CC Case** | VU#643748 |
| **Author / Researcher** | Corrin Clark (donnyoregon / arch) |
| **Report Date** | January 31, 2026 |
| **Protocol** | Marginal V1 |
| **Vulnerable Contract** | [`0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7`](https://etherscan.io/address/0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7) |
| **Patched Contract (impl)** | [`0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8`](https://etherscan.io/address/0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8) |
| **Patch Block** | 24,386,649 (Feb 04, 2026) |
| **Patch Tx** | [`0xe021842b...`](https://etherscan.io/tx/0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a) |
| **Submission Hash (SHA-256)** | `bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368` |

---

## One-Paragraph Summary

The `MarginalV1Pool` contract on Ethereum Mainnet used Q96 fixed-point arithmetic (256-bit integers) internally but cast the value to `uint160` without an overflow check. An attacker using universally available flash loans could push the pool's `sqrtPrice` above the `uint160` maximum, causing the EVM to silently drop the high bits. The result: a $100,000,000 debt position could be settled for `0.000000000000057005 ETH` — a 99.999999% precision loss. This vulnerability was reported to Cantina on January 31, 2026. Despite the Cantina triage team dismissing the report as impossible, the Marginal V1 team silently deployed the exact `SafeCast` fix recommended in the disclosure three days later. Cantina subsequently rejected the finding. After forensic evidence was submitted to CERT/CC, the U.S. federal vulnerability coordination body independently verified the bug and assigned CVE-2026-4931.

---

## Wiki Pages

| Page | Description |
|---|---|
| [Vulnerability Technical Details](Vulnerability-Technical-Details) | Root cause, EVM mechanics, vulnerable code path, impact analysis |
| [Proof of Concept](Proof-of-Concept) | Full PoC walkthrough, Foundry test output, how to reproduce |
| [Stealth Patch Evidence](Stealth-Patch-Evidence) | On-chain transaction proof, bytecode diff, storage slot analysis |
| [Cantina Triage Failure](Cantina-Triage-Failure) | Rejection rationale, forensic rebuttal, dispute claim |
| [CERT/CC and CVE](CERT-CC-and-CVE) | Federal validation process, CVE assignment, official statement |
| [Timeline](Timeline) | Complete chronological record of all events |
| [Glossary](Glossary) | Definitions of all technical terms used in this disclosure |

---

## Repository Contents

| File / Folder | Description |
|---|---|
| `README.md` | Top-level summary with CVE badge and executive overview |
| `SECURITY.md` | Formal security advisory with CVSS breakdown and remediation |
| `report-summary.md` | Extended narrative including the Cantina failure and stealth patch |
| `cantina_submission.txt` | Original verbatim vulnerability report submitted to Cantina |
| `full_exploit_code.t.sol` | Foundry Solidity test demonstrating the arithmetic truncation |
| `marginal_poc_output.txt` | Raw `forge test` terminal output showing passing exploit |
| `poc/` | `bounty_submission.tar.gz` (original submission archive) + `evidence.txt` |
| `bytecode-safe-cast/bytecode_proof.md` | Forensic proof of the `SafeCast` string injected in the patched bytecode |
| `storage-slot-diff/slot_6_diff.md` | Before/after storage slot 6 (`sqrtPriceX96`) values across the patch block |
| `tx-proof/stealth_patch_tx.md` | Details and Etherscan link for the stealth-patch transaction |
| `cantina-rejection/cantina_failure_summary.md` | Summary of Cantina's contradictory triage reasoning |
| `cantina-rejection/DISPUTE_CLAIM_25000.md` | Formal $25,000 bounty dispute documentation |
| `cantina-rejection/cantina_rejection_thread.pdf` | Redacted full Cantina email chain |
| `screenshots/` | Debugger and log screenshots from the PoC runs |
| `videos/` | Screen-recorded walkthroughs of the exploit |

---

## Key External Links

- [CERT/CC VU#643748](https://kb.cert.org/vuls/id/643748)
- [NVD CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931)
- [Patch Transaction on Etherscan](https://etherscan.io/tx/0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a)
- [Patched Implementation on Etherscan](https://etherscan.io/address/0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8)
- [MarginalProtocol/v1-core#10](https://github.com/MarginalProtocol/v1-core/issues/10)
