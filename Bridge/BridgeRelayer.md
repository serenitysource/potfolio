# bridge-relayer.js Documentation

## Overview

`scripts/bridge-relayer.js` is an off-chain Node.js script that acts as a relayer for the cross-chain bridge. It listens for `ContractCall` events on the source `MockAxelarGateway` contract and relays them to the destination `MockAxelarGateway` by calling `testReceiveCall`. It also handles proof injection, payload decoding, and result callbacks to complete the cross-chain bridging process in a local/test environment.

---

## Key Responsibilities
- Listens for `ContractCall` events on the source chain.
- Decodes and re-encodes payloads for cross-chain delivery.
- Injects zk-SNARK proof data (from `proof.json` and `public.json`).
- Calls `testReceiveCall` on the destination `MockAxelarGateway` to simulate Axelar relayer delivery.
- After successful minting, calls `relayBridgeResultToEmitter` on the source gateway to notify the bridge emitter of the result.
- Provides extensive logging and error handling for debugging and monitoring.

---

## Environment Variables
- `SRC_RPC_URL`, `SRC_WSS_URL`: Source chain RPC/WebSocket URLs.
- `DEST_RPC_URL`, `DEST_WSS_URL`: Destination chain RPC/WebSocket URLs.
- `RELAYER_PK`: Private key for relayer account.
- `SRC_MOCK_GATEWAY_ADDRESS`, `DEST_MOCK_GATEWAY_ADDRESS`: Addresses of source/destination mock gateways.
- `DEST_BRIDGE_RECEIVER_ADDRESS`: Address of destination bridge receiver contract.
- `DEST_ETHERGON_CONTRACT_ADDRESS`, `DEST_ETHCARBON_CONTRACT_ADDRESS`: Token contract addresses on destination.
- `SRC_BRIDGE_EMITTER_ADDRESS`: Source bridge emitter address.

---

## Main Components

### 1. Setup and Configuration
- Loads environment variables and contract ABIs.
- Connects to source and destination blockchains using Web3.
- Prints contract and environment info for verification.

### 2. Event Listening
- Subscribes to `ContractCall` events on the source gateway.
- Decodes event payloads (device, user, symbol, amount, proof, etc.).
- Loads zk-SNARK proof and public signals from local files.

### 3. Relaying
- Encodes the payload for the destination chain.
- Calls `testReceiveCall` on the destination gateway, simulating Axelar relayer delivery.
- Handles transaction gas estimation, sending, and receipt logging.

### 4. Result Callback
- After successful minting, calls `relayBridgeResultToEmitter` on the source gateway to notify the bridge emitter of the result.
- Handles errors and logs them to `relay_errors.log`.

### 5. Utilities
- Prints contract info and checks token/bridge mappings for correctness.
- Provides helper functions for proof formatting and payload encoding.

---

## Logic and Purpose
- **Simulates Axelar Relayer**: Enables local/test bridging without a real Axelar network.
- **Proof Injection**: Supports static zk-SNARK proof injection for testing.
- **Cross-Chain Event Flow**: Ensures events and state changes propagate between source and destination chains.
- **Robust Logging**: Provides detailed logs for debugging and monitoring the bridge process.
- **Error Handling**: Catches and logs errors at every step, including event decoding, proof loading, and transaction sending.

---

## Summary Table
| Function/Section                | Purpose                                                      |
|---------------------------------|--------------------------------------------------------------|
| printContractInfo               | Prints environment and contract setup info                   |
| main                            | Main entry point, sets up listeners and relaying logic       |
| srcGateway.events.ContractCall  | Listens for bridge events on source chain                    |
| decode/encode payload           | Handles ABI decoding/encoding for cross-chain payloads       |
| callHandleBridgeResultOnSource  | Calls back to source to notify bridge result                 |
| Proof loading                   | Loads zk-SNARK proof and public signals from files           |
| Error handling                  | Logs errors and writes to relay_errors.log                   |

---


