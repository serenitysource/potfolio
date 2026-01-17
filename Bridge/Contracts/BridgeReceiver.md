# BridgeReceiver.sol Documentation

## Overview

`BridgeReceiver.sol` is a cross-chain bridge contract that receives payloads from the Axelar network, verifies zero-knowledge proofs, and mints tokens on the destination chain. It manages token registration, proof verification, and event logging for cross-chain minting operations.

---

## State Variables
- `tokenMap`: Maps token symbol (string) to token contract address.
- `verifier`: Address of the zk-proof verifier contract.
- `owner`: Owner of the bridge receiver contract.

---

## Events
- `BridgeResult(device, user, symbol, amount, success)`: Emitted after a bridge operation.
- `MintError(user, symbol, amount, lowLevelData)`: Emitted on minting errors.
- `ExecuteCalled(device, user, symbol, amount, token, msgSender)`: Emitted when execute is called.
- `ExecuteEntered(payload, msgSender)`: Emitted when _execute is entered.
- `AxelarExecuteEntered(commandId, sourceChain, sourceAddress, payload, msgSender)`: Emitted when execute is called by Axelar.
- `Debug(message, device, addr, symbol, amount)`: Debugging event.
- `BridgeMinted(device, recipient, symbol, amount, v1TxHash, v1BlockNumber)`: Emitted when tokens are minted after bridge.

---

## Modifiers
- `onlyOwner`: Restricts function to contract owner.

---

## Key Functions

### Ownership and Setup
- `setVerifier(_verifier)`: Owner-only. Sets the zk-proof verifier contract.
- `setToken(symbol, token)`: Owner-only. Registers a token contract for a symbol.
- `registerToken(symbol, token)`: Registers a new token (if not already registered).
- `updateToken(symbol, token)`: Updates token contract for a symbol.

### Minting and Verification
- `verifyAndMint(device, user, symbol, amount, v1TxHash, v1BlockNumber, a, b, c, input)`: Verifies zk-proof and mints tokens to user. Emits events for debugging and result.

### Axelar Integration
- `execute(commandId, sourceChain, sourceAddress, payload)`: Entry point for Axelar. Calls internal `_execute`.
- `_execute(...)`: Decodes payload, verifies proof, mints tokens, and emits events.

### Testing Utilities
- `testReceiveCall(payload)`: For testing, decodes payload and calls `verifyAndMint`.
- `emitTestBridgeResult(device, user, symbol, amount)`: Emits a test bridge result event.

---

## Logic and Purpose
- **Cross-Chain Minting**: Receives payloads from Axelar and mints tokens on the destination chain.
- **ZK-Proof Verification**: Ensures only valid proofs result in minting.
- **Token Registration**: Supports dynamic registration and updating of bridged tokens.
- **Event Logging**: Emits detailed events for debugging and tracking bridge operations.
- **Security**: Only owner can set verifier and register tokens. Only Axelar can call execute.

---

## Summary Table
| Function                | Access      | Purpose                                      |
|-------------------------|-------------|----------------------------------------------|
| setVerifier             | onlyOwner   | Set zk-proof verifier contract               |
| setToken                | onlyOwner   | Register token contract for symbol           |
| registerToken           | public      | Register new token (if not registered)       |
| updateToken             | public      | Update token contract for symbol             |
| verifyAndMint           | public      | Verify proof and mint tokens                 |
| execute                 | Axelar      | Entry point for Axelar payloads              |
| _execute                | internal    | Decode payload, verify, mint, emit events    |
| testReceiveCall         | public      | Test utility for minting                     |
| emitTestBridgeResult    | public      | Emit test bridge result event                |

---


