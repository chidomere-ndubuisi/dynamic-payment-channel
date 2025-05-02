# Dynamic Payment Channel Network (DPCN)

A **Clarity smart contract** enabling secure, scalable, and dynamic off-chain payments between two parties on the **Stacks blockchain**. This contract provides an efficient layer-2 solution for trustless microtransactions through the use of bi-directional payment channels, supporting deposits, updates, disputes, and cooperative or unilateral closures.

## Overview

The **Dynamic Payment Channel Network** (DPCN) allows participants to:

* Open and fund payment channels.
* Exchange payments off-chain with minimal on-chain interaction.
* Cooperatively or unilaterally close channels.
* Resolve disputes through a built-in timeout mechanism.

This smart contract improves scalability and transaction efficiency while preserving security through signature verification and dispute resolution.

## Features

* **Channel Creation**: Users can create unique channels with specified participants and initial deposits.
* **Funding**: Channels can be topped up by the creator at any time.
* **Cooperative Closure**: Channels can be closed when both parties sign off on final balances.
* **Unilateral Closure**: One party can initiate closure with a dispute period to challenge balances.
* ⏱**Dispute Resolution**: Automatic settlement after a 7-day window (based on block height).
* **Security Checks**: Input validation, signature verification, nonce usage, and replay protection.
* **Emergency Withdrawals**: Contract owner can recover funds in emergencies.

## Contract Structure

### Constants

| Name             | Description                                      |
| ---------------- | ------------------------------------------------ |
| `CONTRACT-OWNER` | Owner set to the contract deployer               |
| Error Codes      | Range from unauthorized access to invalid inputs |

### Key Functions

#### Public

* `create-channel(channel-id, participant-b, initial-deposit)`
* `fund-channel(channel-id, participant-b, additional-funds)`
* `close-channel-cooperative(channel-id, participant-b, balance-a, balance-b, signature-a, signature-b)`
* `initiate-unilateral-close(channel-id, participant-b, proposed-balance-a, proposed-balance-b, signature)`
* `resolve-unilateral-close(channel-id, participant-b)`
* `emergency-withdraw()`

#### Read-only

* `get-channel-info(channel-id, participant-a, participant-b)`

---

## Usage

### Creating a Channel

```lisp
(create-channel 0x01 'SP...B  u100000)
```

* Requires a unique 32-byte channel ID.
* The sender becomes `participant-a`.

### Funding a Channel

```lisp
(fund-channel 0x01 'SP...B u50000)
```

* Only the channel creator can fund it.

### Closing a Channel (Cooperatively)

```lisp
(close-channel-cooperative 
  0x01 
  'SP...B 
  u70000 
  u30000 
  0x...sigA 
  0x...sigB)
```

* Requires signatures from both participants verifying final balances.

### Unilateral Closure + Dispute Resolution

```lisp
(initiate-unilateral-close 
  0x01 
  'SP...B 
  u80000 
  u20000 
  0x...sig)
```

* After 7 days, call:

```lisp
(resolve-unilateral-close 0x01 'SP...B)
```

## Security and Validation

* Input and signature validation.
* Replay protection using `nonce`.
* `stx-transfer?` wrapped in `as-contract` to enforce safe asset handling.
* Emergency recovery for contract owner with access control.

## Technical Notes

* **Dispute window**: \~7 days, calculated as `1008` Stacks blocks (\~10 minutes per block).
* **Signature validation** is simplified for compatibility with Clarinet; integrate full cryptographic verification for production.
* **Channel ID** is expected to be a buffer of up to 32 bytes.

## Emergency Recovery

The `emergency-withdraw` function allows the contract owner to retrieve remaining funds from the contract after validation.

```lisp
(emergency-withdraw)
```

## Testing & Simulation

Compatible with [Clarinet](https://docs.stacks.co/docs/clarinet/overview/). Test scenarios should include:

* Valid/invalid channel creation
* Multiple funding rounds
* Signature validation
* Dispute period enforcement
* Re-entrancy and edge case coverage

## Contributing

Pull requests and issue reports are welcome. Focus areas:

* Signature cryptography improvements
* Multi-party channel expansion
* Off-chain coordination tooling

## Contact

For questions, collaborations, or support, open an issue or contact the maintainer.
