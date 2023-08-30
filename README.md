# EIP-XXXX: Cryptographic Proof of Exchange Solvency

## Preamble

```
EIP: XXXX
Title: Cryptographic Proof of Exchange Solvency
Author: [ADD NAMES HERE]
Type: Standards Track (ERC)
Category: ERC
Status: Draft
Created: 2023-08-30
```

## Abstract

This EIP proposes a standard for centralized exchanges to cryptographically prove their solvency without revealing information about individual user balances and aggregated information related to user balances. This is done using cryptographic techniques like zero-knowledge proofs.

## Motivation

The lack of a standardized, cryptographic verification process for exchange solvency puts users at risk and lacks transparency. This standard aims to provide a reliable, auditable, and decentralized mechanism for verifying exchange solvency.

## Specification

### Solidity Interface

```solidity
// [Sample Interface showing methods for proof of solvency]
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
