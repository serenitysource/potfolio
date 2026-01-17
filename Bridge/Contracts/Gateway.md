# MockAxelarGateway.sol Documentation

## Overview

`MockAxelarGateway.sol` is a mock contract simulating the Axelar cross-chain gateway for local development and testing. It allows contracts to simulate cross-chain contract calls, event emission, and relayer actions without requiring a live Axelar network. It is used to test bridge flows between BridgeEmitter and BridgeReceiver contracts.

---

## State Variables
- `contractCallParameters`: Maps commandId to `ContractCallParams` struct, storing cross-chain call details.
- `owner`: Owner of the mock gateway contract.

---

## Structs
- `ContractCallParams`: Stores `commandId`, `sourceChain`, `sourceAddress`, and `payload` for a simulated contract call.

---

## Events
- `ContractCall(sender, destinationChain, destinationContractAddress, payloadHash, payload)`: Emitted when a contract initiates a cross-chain call.
- `TestReceiveCallEntered(commandId, sourceChain, sourceAddress, contractAddress, payload, msgSender)`: Emitted when a simulated receive call is made.
- `TestReceiveCallFirst(data, sender, value)`: Emitted on fallback with data.
- `TestReceiveCallFirstNoData(sender, value)`: Emitted on receive with no data.
- `BeforeTargetExecute(target, commandId)`: Emitted before executing the target contract.
- `AfterTargetExecute(target, commandId)`: Emitted after executing the target contract.
- `DebugRelayBridgeResultToEmitterCaller(msgSender, emitter, user, symbol, amount, success)`: Debug event for relaying bridge results.

---

## Modifiers
- `onlyOwner`: Restricts function to contract owner.

---

## Key Functions

### Ownership
- `releaseOwnership(newOwner, emitter)`: Owner-only. Transfers ownership of a BridgeEmitter contract to a new address.

### Cross-Chain Simulation
- `callContract(destinationChain, destinationContractAddress, payload)`: Simulates a contract call from the source chain, emits event.
- `testReceiveCall(commandId, sourceChain, sourceAddress, contractAddress, payload)`: Simulates a relayer delivering a cross-chain call to the destination contract (e.g., BridgeReceiver), stores parameters, and calls `execute` on the target contract.
- `relayBridgeResultToEmitter(emitter, user, symbol, amount, success)`: Simulates relayer calling back to BridgeEmitter on the source chain with the bridge result.
- `getContractCallParameters(commandId)`: Returns stored contract call parameters for a given commandId.

### Internal
- `_storeContractCallParameters(commandId, sourceChain, sourceAddress, payloadToStore)`: Internal helper to store contract call parameters.

### Fallback/Receive
- `fallback()`: Emits event with calldata and value.
- `receive()`: Emits event with sender and value.

---

## Logic and Purpose
- **Simulates Axelar Gateway**: Allows local testing of cross-chain flows without a real Axelar network.
- **Event Emission**: Emits events for all major actions to aid in debugging and tracing.
- **Parameter Storage**: Stores and exposes contract call parameters for use by destination contracts.
- **Ownership Management**: Allows emergency transfer of BridgeEmitter ownership.
- **Relayer Simulation**: Provides functions for relayers to simulate cross-chain message delivery and bridge result callbacks.

---

## Summary Table
| Function                    | Access      | Purpose                                         |
|-----------------------------|-------------|-------------------------------------------------|
| releaseOwnership            | onlyOwner   | Transfer BridgeEmitter ownership                |
| callContract                | public      | Simulate contract call from source chain        |
| testReceiveCall             | public      | Simulate relayer delivering cross-chain call    |
| relayBridgeResultToEmitter  | public      | Simulate relayer callback to BridgeEmitter      |
| getContractCallParameters   | public      | Retrieve stored contract call parameters        |
| fallback/receive            | public      | Emit events for fallback/receive                |

---


