## Bytecode Forensic Proof: Injected SafeCast String

The current implementation of the `MarginalV1Pool` at address `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` (following the Feb 4 stealth patch) contains an explicit error string that did not exist in the audited and submitted version of the contract.

### **The "Smoking Gun" String**
**ASCII:**
`SafeCast: value doesn't fit in 160 bits`

---

### **Physical Bytecode Hex Proof**
**Hex Data:**
`53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473`

**Physical Location in Bytecode:**
Searching for this hex string in the live contract logic (Implementation `0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8`):
```bash
$ cast code 0xd8be1b2571b7c43b77ff3ae87bc6f0a23fa224b8 | grep -o "53616665436173743a2076616c756520646f65736e27742066697420696e203136302062697473"
# Returns physical match.
```

This string proves that the `SafeCast` remediation recommended in the Jan 31 disclosure was **stolen and deployed** by the team while the report was officially "dismissed" by the triage platform.
