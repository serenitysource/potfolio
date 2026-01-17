# BridgeEmitter.sol Documentation

## Overview

`BridgeEmitter.sol` is a cross-chain bridge contract that locks tokens on the source chain and initiates a minting process on the destination chain using the Axelar network. It manages token locking, gas payments, and communication with the Axelar gateway, and supports multiple tokens via a symbol-to-address mapping.

---

## State Variables
- `gateway`: Axelar gateway contract for cross-chain calls.
- `gasService`: Axelar gas service contract for gas payments.
- `destinationChain`: Name of the destination chain.
- `destinationContract`: Address of the contract on the destination chain.
- `tokenMap`: Maps token symbol (string) to token contract address.
- `owner`: Owner of the bridge contract.
- `locked`: Mapping of user => symbol => amount, tracking locked tokens per user and token.

---

## Events
- `BridgeInitiated(sender, amount, token)`: Emitted when a bridge operation is initiated.
- `DebugBurnAttempt(token, amount, balanceBefore, success)`: Emitted on burn attempts for debugging.
- `DebugBurnError(token, amount, message)`: Emitted on burn errors for debugging.
- `DebugHandleBridgeResultCaller(msgSender, gatewayAddress)`: Debug event for bridge result handler.

---

## Modifiers
- `onlyOwner`: Restricts function to contract owner.

---

## Key Functions

### Ownership
- `transferOwnership(newOwner)`: Transfers contract ownership.

### Token Management
- `setToken(symbol, token)`: Owner-only. Registers a token contract for a symbol.

### Bridging
- `bridgeMint(symbol, amount, user, device, v1TxHash, v1BlockNumber, a, b, c, input)`: Locks tokens, pays gas, and calls the destination contract via Axelar. Requires proof and device/user info.

### Bridge Result Handling
- `handleBridgeResult(user, symbol, amount, success)`: Called by Axelar gateway after destination minting. If successful, burns locked tokens; otherwise, unlocks and returns tokens to user.
- `unlockTokens(symbol)`: Allows users to unlock and withdraw their tokens if bridging fails.

### Utility
- `emitBridgeEvent(amount, token)`: Emits a bridge event (for testing or logging).

---

## Logic and Purpose
- **Token Locking**: Tokens are locked in the contract before bridging to prevent double-spending.
- **Cross-Chain Messaging**: Uses Axelar to send payloads to the destination chain.
- **Gas Payment**: Requires native gas payment for Axelar contract call.
- **Burn on Success**: Burns tokens on successful bridge, or returns them on failure.
- **Multi-Token Support**: Can bridge multiple tokens by symbol.
- **Security**: Only owner can register tokens and transfer ownership. Only Axelar gateway can call bridge result handler.

---

## Summary Table
| Function                | Access      | Purpose                                      |
|-------------------------|-------------|----------------------------------------------|
| transferOwnership       | onlyOwner   | Transfer contract ownership                  |
| setToken                | onlyOwner   | Register token contract for symbol           |
| bridgeMint              | public      | Lock tokens and initiate bridge              |
| handleBridgeResult      | gateway     | Burn or unlock tokens after bridge           |
| unlockTokens            | public      | User unlocks tokens if bridge fails          |
| emitBridgeEvent         | public      | Emit bridge event (testing/logging)          |

---



