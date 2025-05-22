# RGB v0.12 Second Phase Testing: Progress and Contributions

## Introduction
As contributors to the Bitcoin RGB ecosystem, BitlightLabs has been focused on testing and enhancing functionality for RGB v0.12. This post summarizes our latest test results, key issue fixes, and code contributions to the RGB ecosystem.

---

## RGB v0.12 Test Execution Overview

| Test Type     | Total  | Passed | Failed | Ignored |
| ------------- | ------ | ------ | ------ | ------- |
| `issuance.rs` | 18     | 16     | 0      | 2       |
| `transfer.rs` | 44     | 22     | 8      | 14      |
| `stress.rs`   | 10     | 4      | 4      | 2       |
| **Total**     | **72** | **42** | **12** | **18**  |

### Overall Success Rate: 58% (42/72)

---

## Detailed Test Status

### Asset Issuance Tests (100% Pass Rate)

All asset issuance related tests passed successfully, including:

- **FUA (Fractional Unique Asset) issuance**: Single and multiple UTXO allocation scenarios with various parameters
- **NIA (Non-Inflatable Asset) issuance**: Single and multiple UTXO allocation scenarios with various parameters

### Transfer Tests (50% Pass Rate)

#### Passed Tests
- Accepting zero-confirmation transactions
- Sending assets to the same wallet
- Checking fungible asset history
- Basic transfer functionality with different parameters
- Genesis transaction rollback
- Invoice reuse scenarios
- Specific tapret/opret seal scenarios
- Transfer loops with various wallet and asset combinations

#### Failed Tests
1. **rbf_transfer**: Fee replacement (RBF) functionality fails with `Fulfill(StateInsufficient)` error
2. **same_transfer_twice_no_update_witnesses**: Invoice reuse without updating witnesses fails
3. **blank_tapret_opret::case_2..3**: Transaction conflicts when run as part of the full test suite
4. **tapret_wlt_receiving_opret**: "Transaction already in block chain" error
5. **receive_from_unbroadcasted_transfer_to_blinded**: Assertion failure
6. **transfer_loop::case_03**: Assertion failure in specific transfer loop scenario

### Stress Tests (50% Pass Rate)

- **Passed**: back_and_forth::case_1..4 (back and forth transfer tests)
- **Failed**: back_and_forth::case_5..8, all failing with `Transfer(PsbtPrepare(ChangeRequired))` error

---

## Key Issues Analysis and Contributions


### RGB Smart Contract Enhancements

We contributed several key PRs to `rgb-std` for RGB smart contracts, implementing import, export, and destruction functionality:

- [RGB-WG/rgb-std#299](https://github.com/RGB-WG/rgb-std/pull/299)
- [RGB-WG/rgb-std#301](https://github.com/RGB-WG/rgb-std/pull/301)
- [RGB-WG/rgb-std#302](https://github.com/RGB-WG/rgb-std/pull/302)

These PRs restored the Purge, Import, Export, and Backup methods in the RGB CLI, with relevant code path: [RGB-WG/rgb/blob/v0.12-tests/cli/src/exec.rs#L149C1-L166C14](https://github.com/RGB-WG/rgb/blob/v0.12-tests/cli/src/exec.rs#L149C1-L166C14)


### RGB21 Asset Issuance Bug Fixes

We identified and fixed three critical bugs affecting RGB21 asset issuance [FAC, UAC, UDA](https://github.com/pandora-prime/rgb-issuers/issues/2):

1. **TypeAbsent Error in RGB21 Issuers**
   - **Symptom**: `TypeAbsent` errors when issuing RGB21 assets
   - **Root Cause**: RGB21 issuers were using `CommonTypes` instead of `Rgb21Types`
   - **Fix**: PR [pandora-prime/rgb-issuers#3](https://github.com/pandora-prime/rgb-issuers/pull/3)

2. **UnknownType Error in RGB21 Type System**
   - **Symptom**: `UnknownType` error
   - **Root Cause**: `Rgb21Types::type_system` method fails to recognize `ReservedBytes<26>` used in NFT's `_reserved` field
   - **Fix**: PR [LNP-BP/client_side_validation#190](https://github.com/LNP-BP/client_side_validation/pull/190)

3. **Encoding Error Due to Field Order Mismatch**
   - **Symptom**: `bug in business logic of type system` error when trying to encode struct values
   - **Root Cause**: `strict_write_ty` method assumes value and type field orders match exactly
   - **Fix**: PR [strict-types/strict-types#58](https://github.com/strict-types/strict-types/pull/58)

---

## Remaining Issues and Next Steps

1. **Main Issues to Fix**:
   - `StateInsufficient` errors in RBF and invoice reuse tests
   - Blockchain reorganization handling: API lacks stable mechanism for handling blockchain reorganizations
   - Transaction conflicts in test suite execution
   - `ChangeRequired` errors in stress tests
   - Assertion failures in complex transfer scenarios

2. **Next Steps**:
   
   **Feedback to RGB Core Team**:
   - Fix `StateInsufficient` errors in RBF and invoice reuse tests
   - Implement contract architecture support for blockchain reorganization scenarios
   - Provide documentation on payment scripts and internal state transition APIs
   
   **BitlightLabs Team Focus**:
   - Address `ChangeRequired` errors in stress tests
   - Implement test isolation to prevent transaction conflicts
   - Improve RGB21 asset issuance types
   - Continue implementation of coinjoin and Lightning Network tests

---

## Repository Links

- Main Branch: [RGB-WG/rgb-tests](https://github.com/RGB-WG/rgb-tests)  
- Active Dev Branch: [feat/rgb-v12-enhanced-tests](https://github.com/bitlightlabs/rgb-tests/tree/feat/rgb-v12-enhanced-tests)  

Follow our progress and contribute via GitHub! 🚀 