---
title: Proof of Solvency for Centralized Exchanges
description: A standard for centralized exchanges to cryptographically prove their solvency without revealing information about individual user balances and aggregated information related to user balances.
author: [ADD NAMES HERE]
discussions-to: https://ethereum-magicians.org/t/eip-draft-proof-of-solvency-standard-for-centralized-exchanges/15963
status: draft
type: Standards Track
category: ERC
created: 2023-08-30
---

## Abstract

This EIP proposes a standard for centralized exchanges to cryptographically prove their solvency without revealing information about individual user balances and aggregated information related to user balances. This is done using cryptographic techniques like zero-knowledge proofs.

## Motivation

The lack of a standardized, cryptographic verification process for exchange solvency puts users at risk and lacks transparency. This standard aims to provide a reliable, auditable, and decentralized mechanism for verifying exchange solvency.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

### Solidity Interface

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity ^0.8.18;

/**
    @title Cryptographically Auditable Exchange standard
    @dev See https://eips.ethereum.org/EIPS/eip-xxxx
 */
interface ICryptographicallyAuditable {
    /**
     * @dev Struct representing an address ownership proof submitted by the CEX
     * @param cexAddress MUST be the address owned by the CEX (submitted as a string, as it can be a non-EVM address)
     * @param chain MUST be the name of the chain name where the address belongs (e.g., ETH, BTC)
     * @param signature MUST be the signature of the `message` signed by the address public key
     * @param message MUST be the message signed by the address public key. The message MAY be arbitrary.
     */
    struct AddressOwnershipProof {
        string cexAddress;
        string chain;
        bytes signature;
        bytes message;
    }

    /**
     * @dev Struct representing an asset owned by the CEX
     * @param assetName MUST be a unique name of the asset
     * @param chain MUST be the name of the chain where the asset lives (e.g., ETH, BTC). The name MUST correspond to the name used in the `AddressOwnershipProof` struct
     * @param amount MUST be the total amount of the asset that the CEX holds on a given chain
     */
    struct Asset {
        string assetName;
        string chain;
        uint256 amount;
    }

    /**
     * @dev MUST emit when the CEX successfully submits an address ownership proof
     * @param addressOwnershipProofs MUST be the list of address ownership proofs
     */
    event AddressOwnershipProofSubmitted(
        AddressOwnershipProof[] addressOwnershipProofs
    );
    /**
     * @dev MUST emit when the CEX successfully submits a proof of solvency
     * @param timestamp MUST be the timestamp at which the CEX took the snapshot of its assets and liabilities
     * @param mstRoot MUST be the Merkle sum tree root of the CEX's liabilities
     * @param assets MUST be the list of assets owned by the CEX
     */
    event SolvencyProofSubmitted(
        uint256 indexed timestamp,
        uint256 mstRoot,
        Asset[] assets
    );

    /**
     * @dev Submit an optimistic proof of address ownership for a CEX. The proof is subject to an off-chain verification as it's not feasible to verify the signatures of non-EVM chains in an Ethereum smart contract.
     * @param _addressOwnershipProofs MUST be the list of address ownership proofs
     * MUST revert if `_addressOwnershipProofs` array is empty.
     * MUST revert if any `AddressOwnershipProof` has an empty `cexAddress`, `chain`, `signature`, or `message`.
     * MUST revert if the `cexAddress` has already been verified.
     * MUST revert if called by anyone other than the contract owner.
     * MUST emit `AddressOwnershipProofSubmitted` event upon successful submission.
     */
    function submitProofOfAddressOwnership(
        AddressOwnershipProof[] memory _addressOwnershipProofs
    ) external;

    /**
     * @dev Submit proof of solvency of a CEX.
     * @param mstRoot MUST be the Merkle sum tree root of the CEX's liabilities
     * @param assets MUST be the list of assets owned by the CEX
     * @param proof MUST be the ZK proof generated from the specified proof of solvency circuit
     * @param timestamp MUST be the timestamp at which the CEX took the snapshot of its assets and liabilities
     * The proof MUST be accepted if it satisfies the corresponding verifier generated from the described circuit. The inputs of the solvency proof (`mstRoot` and `assets`) are optimistically considered valid. The validity of the CEX asset balances is subject to off-chain verification, and the validity of the liabilities can be challenged by users at the user inclusion verification stage.
     * In case of successful proof submission `mstRoot` MUST be stored in the contract state against the given `timestamp` because it MAY be used by `verifyProofOfInclusion` function to verify the proof of user inclusion into the Merkle sum tree of the CEX's liabilities. `verifyProofOfInclusion` MUST be able to look up the stored `mstRoot` by a timestamp.
     * MUST revert if `mstRoot` is zero.
     * MUST revert if `timestamp` is zero.
     * MUST revert if `assets` array is empty.
     * MUST revert if any `Asset` has an empty `chain` or `assetName`.
     * MUST revert if any `Asset` has an `amount` of zero.
     * MUST revert if the ZK proof verification fails.
     * MUST revert if called by anyone other than the contract owner.
     * MUST emit `AddressOwnershipProofSubmitted` event upon successful submission.
     */
    function submitProofOfSolvency(
        uint256 mstRoot,
        Asset[] memory assets,
        bytes memory proof,
        uint256 timestamp
    ) external;

    /**
     * @dev Verify a ZK proof of user inclusion in the Merkle sum tree of the CEX's liabilities
     * @param proof MUST be a ZK proof of inclusion generated from the specified proof of inclusion circuit
     * @param timestamp MUST be a timestamp at which the CEX took the snapshot of its assets and liabilities
     * MUST return `true` if the proof is valid, `false` otherwise
     * MUST revert if `timestamp` is zero.
     * MUST revert if the ZK proof verification fails.
     * MUST be able to look up the stored MST root by a `timestamp` for verification. MUST revert if the lookup fails.
     * MUST NOT modify the contract state. MUST NOT reject any calls based on caller address.
     */
    function verifyProofOfInclusion(
        bytes memory proof,
        uint256 timestamp
    ) external view returns (bool);
}
```

### Core Circuits

- **Proof of Inclusion Circuit**: Ensures a user's assets are included in the total calculation of the exchange's assets.
- **Proof of Solvency Circuit**: Verifies the solvency of the exchange.

### Data Structures

- **Liability Tree**: Data structure used to maintain and compute liabilities.

## Rationale

This standard is designed to achieve a balance between privacy and verifiability. It allows users and independent actors to confirm an exchange's solvency without exposing individual account balances, adding an extra layer of security and trust to the digital assets ecosystem.

## Backwards Compatibility

The implementation of this standard is expected to be fully backward compatible.

[Look at some other EIPs and see what's appropriate for this section]

## Test Cases

[Describe test cases]

## Reference Implementation

The [Summa](https://github.com/summa-dev) team (supported by the Privacy and Scaling Explorations team) has created a reference implementation [here](https://github.com/summa-dev/summa-solvency).

## Security Considerations

Given that the standard utilizes advanced cryptographic techniques, it's crucial to ensure that the circuits used are secure and that the smart contracts have undergone rigorous auditing.

[Describe security considerations]

## Copyright

Copyright and related rights waived via [CC0](/LICENSE).
