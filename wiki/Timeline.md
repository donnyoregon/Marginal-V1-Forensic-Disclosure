# Timeline of Events

This page provides the complete, chronological record of all events related to CVE-2026-4931, from initial discovery through CVE assignment.

---

## Full Timeline

| Date | Event | Details |
|---|---|---|
| **Jan 31, 2026** | Vulnerability discovered and reported | Researcher Corrin Clark (donnyoregon) files a full report with Cantina, including Foundry PoC, video walkthrough, and explicit fix recommendation. Submission hash: `bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368` |
| **Feb 02, 2026** | Marginal V1 pauses protocol operations | The Marginal V1 team pauses the live protocol. Timing is consistent with receiving the disclosure through Cantina. |
| **Feb 04, 2026** | Stealth patch deployed to Ethereum Mainnet | Transaction [`0xe021842b...`](https://etherscan.io/tx/0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a) at block 24,386,649 upgrades the `MarginalV1Pool` proxy to implementation `0xd8be1b...`, injecting the exact `SafeCast` fix from the report. EIP-1967 slots are zeroed to hide the upgrade from block explorers. Storage slot 6 (`sqrtPriceX96`) is manually reset in the same transaction. |
| **Feb 10, 2026** | Cantina rejects the finding | Cantina triage team (including Mike Leffer) formally rejects the vulnerability report, citing "Gnosis Safe limits" and claiming the overflow threshold is unreachable. |
| **Feb 17, 2026** | Cantina Support misclassifies the patch tx | Cantina Support officially classifies the February 4 proxy upgrade transaction as an "MEV swap," providing a false basis for the rejection. Forensic evidence (bytecode proof, storage diff, tx analysis) is compiled and submitted to CERT/CC. |
| **Feb 19, 2026** | CERT/CC validates the vulnerability | After independent technical analysis, CERT/CC confirms the unsafe `uint160` downcast and opens Vulnerability Note VU#643748. |
| **Mar 28, 2026** | CVE-2026-4931 officially assigned | CERT/CC coordinates with MITRE to assign CVE-2026-4931 (CVSS 9.1 Critical, CWE-197). The NVD entry is published. |
| **Apr 13, 2026** | Public forensic disclosure repository published | This repository is made public as the permanent record of the vulnerability, the stealth patch, and the Cantina platform failure. |

---

## Annotated Narrative

### January 31, 2026 — Initial Report

The researcher submitted a complete, production-ready vulnerability report to Cantina covering:
- The exact vulnerable code pattern: `uint160 price = uint160(sqrtPriceX96);`
- EVM mechanics explaining why the cast silently truncates
- A working Foundry test demonstrating a 99.999999% precision loss
- A video walkthrough of the exploit
- Explicit remediation guidance (SafeCast or require-based check)

The submission archive was hashed before submission to establish cryptographic priority.

### February 2, 2026 — Protocol Pause

Two days after the report was filed, the Marginal V1 team paused the live protocol. This rapid pause is consistent with having received and understood the disclosure. Under standard coordinated disclosure practice, this would typically be followed by a patch, acknowledgment, and researcher compensation.

### February 4, 2026 — The Stealth Patch

Rather than working through Cantina's standard disclosure process, the Marginal V1 team executed a covert upgrade at block 24,386,649. The key forensic indicators that distinguish this from a normal operational upgrade:

1. **The EIP-1967 slots were zeroed** — standard upgrades do not erase the implementation pointer.
2. **The SafeCast string appears in the new bytecode** — identical to the fix recommended 4 days earlier.
3. **Storage slot 6 was reset in the same block** — erasing the visible record of the manipulated `sqrtPriceX96` state.

### February 10, 2026 — Cantina Rejection

Without conducting adequate on-chain forensic review, Cantina dismissed the report. The triage team cited the Gnosis Safe management structure and internal swap limits as reasons the overflow was "impossible" — neither of which is relevant to the publicly-accessible swap function attack path.

### February 17, 2026 — The "MEV Swap" Misclassification

When the researcher presented the patch transaction as evidence that the vulnerability was real and had been used, Cantina Support reviewed it and concluded it was a routine MEV swap. This conclusion is forensically incorrect. MEV swaps do not change proxy implementation bytecode, inject SafeCast error strings, reset storage slots, or zero EIP-1967 implementation pointers.

On the same date, the full forensic evidence package was submitted to CERT/CC.

### February 19, 2026 — CERT/CC Validates

CERT/CC's independent review confirmed the vulnerability within two days of receiving the submission. Their validation directly contradicts Cantina's "impossible" classification.

### March 28, 2026 — CVE Assigned

CVE-2026-4931 is a permanent, immutable public record. Its assignment constitutes the authoritative resolution of the validity dispute — the vulnerability was real, critical, and reported by the researcher before it was patched.

---

## Evidence Anchored to the Timeline

| Date | Evidence | File |
|---|---|---|
| Jan 31, 2026 | SHA-256 hash of original submission | `bb111bfc...` (in `README.md`) |
| Jan 31, 2026 | Original Cantina submission text | [`cantina_submission.txt`](../cantina_submission.txt) |
| Jan 31, 2026 | Original PoC archive | [`poc/bounty_submission.tar.gz`](../poc/bounty_submission.tar.gz) |
| Feb 04, 2026 | Patch transaction | [`tx-proof/stealth_patch_tx.md`](../tx-proof/stealth_patch_tx.md) |
| Feb 04, 2026 | Bytecode SafeCast string | [`bytecode-safe-cast/bytecode_proof.md`](../bytecode-safe-cast/bytecode_proof.md) |
| Feb 04, 2026 | Storage slot 6 diff | [`storage-slot-diff/slot_6_diff.md`](../storage-slot-diff/slot_6_diff.md) |
| Feb 10–17, 2026 | Cantina rejection & misclassification | [`cantina-rejection/cantina_failure_summary.md`](../cantina-rejection/cantina_failure_summary.md) |
| Feb 19, 2026 | CERT/CC validation | [VU#643748](https://kb.cert.org/vuls/id/643748) |
| Mar 28, 2026 | CVE assignment | [CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931) |
