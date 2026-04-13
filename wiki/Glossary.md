# Glossary

This page defines all technical and contextual terms used throughout this disclosure. Terms are organized alphabetically. If you are new to smart contract security or DeFi, start with the [Background Concepts](#background-concepts) section first.

---

## Background Concepts

### Smart Contract
A program stored and executed directly on a blockchain. Once deployed, its code runs exactly as written — there is no way to stop a transaction mid-execution (unless the code explicitly reverts). Smart contracts in DeFi handle real money with no human intermediary.

### Ethereum Virtual Machine (EVM)
The computation engine that executes Ethereum smart contracts. The EVM operates on 256-bit (32-byte) words natively. All Solidity integer arithmetic happens in this environment.

### Solidity
The primary programming language used to write Ethereum smart contracts. Solidity compiles down to EVM bytecode.

### DeFi (Decentralized Finance)
Financial applications built on smart contracts that operate without banks or brokers. Examples include lending protocols, decentralized exchanges (DEXs), and derivative platforms like Marginal V1.

---

## Technical Terms

### AND Mask (Bitwise AND)
An operation that compares two binary numbers bit by bit and outputs `1` only where both inputs are `1`. In the EVM, casting a 256-bit integer to a 160-bit integer is equivalent to applying an AND mask of `0x00...00FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF` (160 ones), which drops the upper 96 bits silently.

### Bytecode
The compiled, binary representation of a smart contract that is stored on the blockchain and executed by the EVM. Unlike Solidity source code (which is human-readable), bytecode is a sequence of low-level operation codes (opcodes).

### CWE-197: Numeric Truncation Error
A weakness in the MITRE CWE (Common Weakness Enumeration) taxonomy. It describes the class of bugs where a numeric value is implicitly or explicitly narrowed to a smaller type, causing the loss of significant bits. CVE-2026-4931 is classified as CWE-197.

### CVE (Common Vulnerabilities and Exposures)
A standardized identifier for publicly known software vulnerabilities, maintained by MITRE under the CVE Program. Each CVE has a unique ID (e.g., CVE-2026-4931) and is listed in the National Vulnerability Database (NVD).

### CVSS (Common Vulnerability Scoring System)
A numeric framework for rating the severity of vulnerabilities. Scores range from 0.0 to 10.0: 
- 0.0: None  
- 0.1–3.9: Low  
- 4.0–6.9: Medium  
- 7.0–8.9: High  
- 9.0–10.0: Critical

CVE-2026-4931 scores **9.1 (Critical)**.

### EIP-1967 (Transparent Proxy Storage Slots)
An Ethereum Improvement Proposal that standardizes the storage slots used by proxy contracts to store their implementation address. Block explorers and tools check these slots to surface the current logic contract address. Zeroing these slots hides the implementation pointer from standard tooling.

### EOA (Externally Owned Account)
A standard Ethereum wallet controlled by a private key — as opposed to a contract account. Any EOA can call public smart contract functions. The attacker in this disclosure requires no special privileges beyond being an EOA.

### Fixed-Point Arithmetic
A method of representing fractional numbers using integers, by pre-multiplying by a fixed scaling factor. Uniswap V3 and Marginal V1 use Q96 fixed-point: values are stored multiplied by `2^96` to preserve sub-integer precision within 256-bit integers.

### Flash Loan
A DeFi primitive that allows borrowing any amount of tokens for a single transaction with no collateral, as long as the borrowed amount (plus fees) is repaid before the transaction ends. If repayment fails, the entire transaction reverts — making flash loans risk-free for the lender. They are universally available on Aave, Uniswap V3, Balancer, and other protocols, making them the standard tool for price manipulation attacks.

### Foundry
An open-source smart contract development framework (written in Rust) used for writing and running Ethereum tests. The `forge test` command executes Solidity test files. The PoC for this vulnerability is implemented as a Foundry test (`full_exploit_code.t.sol`).

### Integer Overflow / Underflow
In older Solidity versions (pre-0.8.0), arithmetic could silently wrap around: adding 1 to the maximum `uint256` value would produce 0. Solidity 0.8.0+ added automatic revert-on-overflow for addition, subtraction, and multiplication — but **not** for explicit type casts. The vulnerability in CVE-2026-4931 is a cast-based truncation, not a standard overflow, and Solidity 0.8.20 does not protect against it.

### Integer Truncation
The loss of bits that occurs when a larger integer type is cast to a smaller one. In this disclosure, a 256-bit value is cast to 160 bits, dropping the upper 96 bits. This is the root cause of CVE-2026-4931. See also: CWE-197.

### Liquidity Provider (LP)
A user who deposits tokens into a DeFi pool in exchange for a share of trading fees. In a successful exploitation of CVE-2026-4931, all funds deposited by liquidity providers and lenders would be drained from the pool.

### MEV (Maximal Extractable Value)
Profit extracted by reordering, inserting, or censoring transactions within a block. MEV bots execute "MEV swaps" — automated trades that capture price discrepancies. Cantina incorrectly classified the February 4 stealth patch transaction as an MEV swap.

### NVD (National Vulnerability Database)
The U.S. government's (NIST) official repository of vulnerability data, enriched with CVSS scores and CWE classifications. CVE-2026-4931 is listed at [https://nvd.nist.gov/vuln/detail/CVE-2026-4931](https://nvd.nist.gov/vuln/detail/CVE-2026-4931).

### OpenZeppelin
A widely used library of audited, reusable smart contract components for Solidity. The `SafeCast` library from OpenZeppelin provides safe integer downcasting functions that revert instead of silently truncating. The patch for CVE-2026-4931 uses `SafeCast.toUint160()`.

### PoC (Proof of Concept)
A minimal, working demonstration of a vulnerability. In smart contract security, PoCs are typically Foundry or Hardhat test files that reproduce the vulnerable condition and prove the impact.

### Proxy Contract (Upgradeable Proxy)
A design pattern where a "proxy" contract delegates all calls to a separate "implementation" contract. The proxy stores the state; the implementation holds the logic. Upgrading the protocol means pointing the proxy at a new implementation address. Marginal V1 uses this pattern — the vulnerable proxy is `0x3A6C55Ce7...` and the patched implementation is `0xd8be1b25...`.

### Q96 Fixed-Point
A specific fixed-point format where values are represented as their true value multiplied by `2^96`. Used by Uniswap V3 and Marginal V1 to represent `sqrtPriceX96` — the square root of the price ratio of two tokens, scaled by `2^96`. Values can reach up to `type(uint256).max`, well above the `type(uint160).max` limit.

### SafeCast
An OpenZeppelin Solidity library that provides safe downcasting functions. Instead of silently truncating a value, `SafeCast.toUint160(x)` reverts with the message `"SafeCast: value doesn't fit in 160 bits"` if `x > type(uint160).max`. The presence of this error string in the patched bytecode is the forensic "smoking gun" confirming the vulnerability was patched using the exact recommendation from the disclosure.

### SHA-256
A cryptographic hash function that produces a 256-bit (64 hex character) digest of any input. Used in this disclosure to establish the timestamp and integrity of the original submission: `bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368`.

### sqrtPriceX96
The state variable in `MarginalV1Pool` (and Uniswap V3) that represents the square root of the current pool price, stored in Q96 fixed-point format. It is a `uint256` internally but is cast to `uint160` during settlement calculations in the vulnerable code.

### Storage Slot
Ethereum contracts store persistent state in a key-value store where keys are 256-bit slot indices. Slot 6 in `MarginalV1Pool` contains the `State` struct, which includes `sqrtPriceX96`. This slot was modified in the stealth patch transaction.

### uint160 / uint256
Solidity unsigned integer types of 160 and 256 bits, respectively. `uint256` is the native EVM word size. `uint160` is commonly used for Ethereum addresses (which are 20 bytes = 160 bits). Casting a `uint256` value to `uint160` without checking that the value fits is the root cause of CVE-2026-4931.

---

## Organizations and Platforms

### Cantina
A smart contract bug bounty platform that mediates between security researchers and DeFi protocols. Cantina was the platform through which the initial vulnerability report was submitted on January 31, 2026. Their triage process failed to correctly validate the finding. See [Cantina Triage Failure](Cantina-Triage-Failure).

### CERT/CC
The CERT Coordination Center at Carnegie Mellon University's Software Engineering Institute. A federally funded vulnerability coordination body that independently validated CVE-2026-4931. See [CERT/CC and CVE](CERT-CC-and-CVE).

### Marginal V1
A DeFi lending and options protocol deployed on Ethereum Mainnet. The `MarginalV1Pool` contract at `0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7` is the vulnerable contract.

### MITRE
The non-profit organization that manages the CVE Program and assigns CVE identifiers. CVE-2026-4931 was assigned by CERT/CC through MITRE's CVE Program.

### NIST
The U.S. National Institute of Standards and Technology, which maintains the National Vulnerability Database (NVD) and publishes CVSS scoring for CVEs.
