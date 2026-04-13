# Stealth Patch Evidence

## Table of Contents
1. [Overview](#overview)
2. [The Patch Transaction](#the-patch-transaction)
3. [EIP-1967 Slot Obfuscation](#eip-1967-slot-obfuscation)
4. [Bytecode Forensic Proof: The SafeCast String](#bytecode-forensic-proof-the-safecast-string)
5. [Storage Slot 6 Diff: sqrtPriceX96 Reset](#storage-slot-6-diff-sqrtpricex96-reset)
6. [Summary of Evidence Chain](#summary-of-evidence-chain)

---

## Overview

On February 4, 2026 — four days after the vulnerability was reported to Cantina and while the official Cantina triage team had **dismissed** the report as impossible — the Marginal V1 team silently upgraded the `MarginalV1Pool` proxy to a new implementation that contained the exact `SafeCast` fix recommended in the disclosure.

This page documents every piece of on-chain evidence proving:

1. **The upgrade happened** — the proxy implementation pointer was changed.
2. **The fix matches the recommendation** — the patched bytecode contains OpenZeppelin's `SafeCast` error string.
3. **The state was reset** — storage slot 6 (`sqrtPriceX96`) was manually adjusted in the same transaction to erase evidence of the manipulated price.
4. **The upgrade was disguised** — the EIP-1967 implementation slots were zeroed out to hide the change from standard blockchain explorers.

---

## The Patch Transaction

| Field | Value |
|---|---|
| **Transaction Hash** | [`0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a`](https://etherscan.io/tx/0xe021842bc2fe89865e41ef20aa84a8f649efe82d515fe3980b6dd160b564189a) |
| **Block Number** | `24,386,649` |
| **Date** | February 4, 2026 |
| **Proxy Contract** | [`0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7`](https://etherscan.io/address/0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7) |
| **New Implementation** | [`0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8`](https://etherscan.io/address/0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8) |
| **Classified as by Cantina** | "MEV swap" (incorrect) |

Source: [`tx-proof/stealth_patch_tx.md`](../tx-proof/stealth_patch_tx.md)

---

## EIP-1967 Slot Obfuscation

EIP-1967 defines a standard storage slot (`0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`) where proxy contracts store the address of their current implementation. Standard tools like Etherscan's proxy detector, Tenderly, and block explorers check this slot to surface the implementation address.

The Marginal V1 team **zeroed out this slot** in the same transaction, preventing standard tools from detecting that an implementation upgrade had occurred. The slot was deliberately made to appear as `0x0000...0000` while the actual logic was routed to the new `SafeCast`-patched implementation.

This is not a standard operational practice. Zeroing the EIP-1967 implementation slot serves no purpose other than to obscure the upgrade path from public view.

**Consequence:** Cantina's triage team, when later asked to review the patch evidence, examined the proxy slot, saw `0x0`, and classified the transaction as a routine "MEV swap" rather than an implementation upgrade. This classification was used to justify the report rejection.

---

## Bytecode Forensic Proof: The SafeCast String

Source: [`bytecode-safe-cast/bytecode_proof.md`](../bytecode-safe-cast/bytecode_proof.md)

### The Smoking Gun

The OpenZeppelin `SafeCast` library, when it reverts, emits this exact human-readable error string:

```
SafeCast: value doesn't fit in 160 bits
```

This string **did not exist** in the original pre-patch bytecode of `MarginalV1Pool`. It **does exist** in the post-patch implementation `0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8`.

### Hex Encoding

```
ASCII:  SafeCast: value doesn't fit in 160 bits
Hex:    53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473
```

### On-Chain Verification

You can verify this string is present in the live patched implementation using [cast](https://book.getfoundry.sh/reference/cast/cast-code) (part of Foundry):

```bash
cast code 0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8 \
  | grep -o "53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473"
```

A non-empty result confirms the `SafeCast` library was injected into the post-patch bytecode.

### Why This Matters

The initial disclosure (January 31, 2026) explicitly recommended:

```solidity
// Use OpenZeppelin's SafeCast lib.
uint160 price = SafeCast.toUint160(sqrtPriceX96);
```

The patched contract contains exactly this fix. The probability of the Marginal V1 team independently and simultaneously arriving at this identical remediation — while officially claiming the vulnerability was impossible — is effectively zero. The evidence establishes that the team used the report to silently fix the vulnerability without compensating the researcher.

---

## Storage Slot 6 Diff: sqrtPriceX96 Reset

Source: [`storage-slot-diff/slot_6_diff.md`](../storage-slot-diff/slot_6_diff.md)

Storage slot 6 of `MarginalV1Pool` contains the `State` struct, which includes `sqrtPriceX96` — the exact value that must be manipulated above `type(uint160).max` to trigger the vulnerability.

### Pre-Patch (Block 24,386,648)

```
Slot 6 Value: 0x0100fff63ea6ed4618697ece7bfc749e00000000000001212121b92a4cdebafa
```

### Post-Patch (Block 24,386,649)

```
Slot 6 Value: 0x0100fff62ce4fc48806983d107fc77ca0000000000000121229b1970a79d6973
```

### Forensic Significance

The `sqrtPriceX96` portion of slot 6 was **manually adjusted** in the same block and transaction that upgraded the logic bytecode. This indicates:

1. The team was aware of the vulnerable price state and deliberately reset it.
2. The storage manipulation was coordinated with the bytecode upgrade, suggesting a planned remediation rather than an incidental swap.
3. The price reset removed the most visible on-chain indicator that the overflow condition had been approached.

---

## Summary of Evidence Chain

The following table summarizes all pieces of on-chain evidence and what each one proves:

| Evidence | Location | What It Proves |
|---|---|---|
| Patch transaction hash | [`tx-proof/stealth_patch_tx.md`](../tx-proof/stealth_patch_tx.md) | The upgrade occurred 4 days after the report |
| EIP-1967 slot zeroed | On-chain at proxy address | The upgrade was deliberately hidden from block explorers |
| `SafeCast` error string in bytecode | [`bytecode-safe-cast/bytecode_proof.md`](../bytecode-safe-cast/bytecode_proof.md) | The exact fix recommended in the disclosure was used |
| Storage slot 6 value changed | [`storage-slot-diff/slot_6_diff.md`](../storage-slot-diff/slot_6_diff.md) | The manipulated price state was manually reset in the same tx |
| Cantina classification as "MEV swap" | [`cantina-rejection/cantina_failure_summary.md`](../cantina-rejection/cantina_failure_summary.md) | Cantina's review of this tx was incorrect, enabling the rejection |
| SHA-256 hash of original submission | `bb111bfc...` | The PoC predates the patch — establishing priority |
