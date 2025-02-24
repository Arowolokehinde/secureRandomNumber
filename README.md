# Secure Random Number Generator for Stacks

A Clarity utility that generates verifiably random numbers by combining multiple entropy sources, designed for games, lotteries, NFT distributions, and fair selection processes.

## Overview

Generating true randomness in a deterministic blockchain environment is challenging but essential for many applications. This utility solves this problem by combining multiple sources of entropy to create unpredictable but verifiable random numbers on the Stacks blockchain.

## Features

- **Multiple Entropy Sources**: Combines Stacks blocks, Bitcoin blocks, transaction data, and state variables
- **Same-Block Protection**: Prevents multiple calls within the same block to resist manipulation
- **Evolving State**: Each generation affects future randomness through an entropy accumulator
- **Flexible Ranges**: Generate numbers within any uint range
- **Gas Efficient**: Optimized for minimal resource usage
- **Verification Mechanism**: Provides access to entropy state for verification

## Installation

1. Copy the `secure-random-generator.clar` contract to your project
2. Deploy to the Stacks blockchain using Clarinet or another deployment method

## Usage

### Basic Random Number

Generate a random number between 0 and a maximum value:

```clarity
;; Get a random number between 0 and 100
(contract-call? .secure-random-generator get-random 
  0x123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0 
  u100)

;; Returns (ok u42) - A number between 0 and 100
```

### Custom Range

Generate a random number within a specific range:

```clarity
;; Get a random number between 5 and 10
(contract-call? .secure-random-generator get-random-in-range
  0x123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0
  u5
  u10)

;; Returns (ok u7) - A number between 5 and 10
```

### With Dynamic Seed

Create transaction-specific seeds for even more unpredictability:

```clarity
;; Create a transaction-specific seed from block info
(let
  (
    (block-data (unwrap-panic (get-stacks-block-info? header-hash stacks-block-height)))
    (dynamic-seed (sha256 block-data))
  )
  (contract-call? .secure-random-generator get-random dynamic-seed u1000)
)
```

### Verify Entropy State

Access the current entropy state for verification:

```clarity
(contract-call? .secure-random-generator get-entropy-state)
;; Returns the current entropy accumulator state
```

## How It Works

1. **Entropy Collection**: The generator collects entropy from:
   - Recent Stacks block header hashes
   - Bitcoin burn block hash
   - Transaction sender information
   - Evolving entropy accumulator
   - Nonce counter
   - User-provided seed

2. **Entropy Combination**: These sources are combined using SHA-256 hashing to create a single 32-byte random value.

3. **Number Extraction**: The 32-byte hash is converted to a uint and scaled to the requested range.

4. **State Evolution**: Each generation updates the entropy accumulator and nonce, ensuring each random number affects future generations.

5. **Block Protection**: A check prevents multiple calls within the same block to protect against manipulation.

## Security Considerations

While no blockchain random number generator can be perfectly unpredictable to miners/validators, this implementation provides strong protection against common attack vectors:

- **Multiple Data Sources**: Uses a combination of unpredictable values
- **Block Dependency**: Uses data from already mined blocks
- **State Evolution**: Maintains state that changes with each call
- **Same-Block Protection**: Prevents multiple calls in the same block

## Use Cases

- **NFT Distributions**: Randomly select traits or mint IDs
- **Gaming**: Generate unpredictable results for card draws, dice rolls, etc.
- **Selection Processes**: Fair committee member selection, task assignment
- **Randomized Testing**: Statistical sampling for on-chain processes

## License

MIT License

## Author

Your Name

---

*This recipe is part of the Hiro Cookbook, a collection of reusable patterns for building on Stacks.*