## Storage Slot 6 Analysis: MarginalV1Pool

Slot 6 corresponds to the `State` struct in the `MarginalV1Pool`. This slot contains the `sqrtPriceX96` value. During the stealth patch, this slot was modified to reset the pool state and hide the previous vulnerable price point.

### **Vulnerable State (Pre-Patch)**
**Block:** `24386648`  
**Slot 6 Value:**
`0x0100fff63ea6ed4618697ece7bfc749e00000000000001212121b92a4cdebafa`

---

### **Patched State (Post-Patch)**
**Block:** `24386649`  
**Slot 6 Value:**
`0x0100fff62ce4fc48806983d107fc77ca0000000000000121229b1970a79d6973`

**Forensic Difference:**
The underlying `sqrtPriceX96` was manually adjusted during the same transaction that upgraded the logic bytecode to include the `SafeCast` remediation.
