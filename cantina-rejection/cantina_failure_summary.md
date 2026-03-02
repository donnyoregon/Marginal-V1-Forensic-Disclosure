## Cantina Platform Failure: Marginal V1 Case

The following is a summary of the triage correspondence regarding the Marginal V1 truncation report.

### **The "Gnosis Safe" Rejection**
The Cantina triage team (including Mike Leffer) dismissed the critical insolvency vulnerability using the following rationale:
> "The vulnerability is impossible because the protocol uses a Gnosis Safe for management, and the internal swap limits prevent the price from reaching the truncation threshold."

---

### **The Forensic Rebuttal**
1. **Misunderstanding of the Vector:** The vulnerability did not require management access; it was triggered via a **publicly accessible swap function** using flash loan liquidity.
2. **Economic Proof:** The "Gnosis Safe" provides no protection against arithmetic truncation in the core logic contract.
3. **The Theft:** While publicly maintaining this dismissal, the Marginal V1 team privately used the report to deploy a mainnet patch on Feb 4, 2026.

**Attached Evidence:**
- `cantina_rejection_thread.pdf` (Complete redacted email chain)
