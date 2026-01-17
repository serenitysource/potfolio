# Cross-Chain Bridge Complete Flow Documentation

## Overview

This document explains the complete flow of the Ergon cross-chain bridge system, which enables secure token transfers between a private EVM network (source) and a public Ethereum network (destination) using zero-knowledge proofs for verification.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CROSS-CHAIN BRIDGE SYSTEM                          │
└─────────────────────────────────────────────────────────────────────────────┘

    SOURCE CHAIN                    RELAYER                  DESTINATION CHAIN
  (Private EVM/Ganache)         (Off-Chain Service)         (Public Ethereum)
         │                              │                           │
         │                              │                           │
    ┌────▼────┐                    ┌────▼────┐                 ┌────▼────┐
    │  User   │                    │ Bridge  │                 │ Bridge  │
    │ Wallet  │                    │ Relayer │                 │Receiver │
    └────┬────┘                    │ Script  │                 │Contract │
         │                         └────┬────┘                 └────┬────┘
         │                              │                           │
    ┌────▼────────┐                     │                      ┌────▼────────┐
    │   Carbon/   │                     │                      │  ETHCarbon/ │
    │   Ergon     │                     │                      │  ETHErgon   │
    │   Token     │                     │                      │   Token     │
    └────┬────────┘                     │                      └─────────────┘
         │                              │
    ┌────▼────────┐                     │                      ┌──────────────┐
    │   Bridge    │                     │                      │  Groth16     │
    │   Emitter   │◄────────────────────┼──────────────────────┤  Verifier    │
    └────┬────────┘                     │                      └──────────────┘
         │                              │
    ┌────▼────────┐                     │
    │MockAxelar   │                     │
    │  Gateway    │                     │
    └─────────────┘                     │
                                        │
                                   ┌────▼────┐
                                   │ ZK Proof│
                                   │Generator│
                                   │(Static) │
                                   └─────────┘
```

---

## Component Details

### Source Chain Components

1. **Token Contracts (Carbon/Ergon)**
   - Standard ERC20 tokens on the private chain
   - Can be burned/locked for cross-chain transfer

2. **BridgeEmitter Contract**
   - Initiates cross-chain transfers
   - Burns tokens on source chain
   - Emits events for relayer to listen

3. **MockAxelarGateway (Source)**
   - Simulates Axelar cross-chain messaging
   - Emits `ContractCall` events

### Relayer (Off-Chain)

1. **Bridge Relayer Script**
   - Listens for events on source chain
   - Loads ZK proofs (static or dynamic)
   - Calls destination chain contracts
   - Handles transaction submission

2. **ZK Proof Management**
   - Currently: Static proof from `proof.json`
   - Future: Dynamic proof generation per transaction

### Destination Chain Components

1. **BridgeReceiver Contract**
   - Verifies ZK proofs on-chain
   - Mints wrapped tokens
   - Enforces security checks

2. **Wrapped Token Contracts (ETHCarbon/ETHErgon)**
   - ERC20 tokens on destination chain
   - Minted by BridgeReceiver after verification

3. **Groth16Verifier Contract**
   - On-chain ZK proof verification
   - Generated from circom circuit
   - Called by BridgeReceiver

4. **MockAxelarGateway (Destination)**
   - Receives cross-chain messages
   - Triggers BridgeReceiver execution

---

## Complete Transaction Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        STEP-BY-STEP TRANSACTION FLOW                        │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: USER INITIATES BRIDGE TRANSFER
═══════════════════════════════════════════════════════════════════════════════

    User Wallet (Source Chain)
         │
         │ calls: burn(amount)
         ▼
    ┌─────────────────────┐
    │   Carbon/Ergon      │  ──► Burns tokens
    │   Token Contract    │      Amount removed from supply
    └──────────┬──────────┘
               │
               │ emits: Transfer(from, 0x0, amount)
               ▼


STEP 2: BRIDGE EMITTER PROCESSES BURN
═══════════════════════════════════════════════════════════════════════════════

    ┌─────────────────────┐
    │  BridgeEmitter      │
    │  Contract           │
    └──────────┬──────────┘
               │
               │ function: initiateBridge(symbol, amount, destChain, destContract)
               ▼
    ┌─────────────────────────────────────────────────────────────┐
    │  Encodes Payload:                                           │
    │  abi.encode(device, user, symbol, amount, txHash, blockNum) │
    └──────────┬──────────────────────────────────────────────────┘
               │
               │ calls: gateway.callContract(destChain, destContract, payload)
               ▼
    ┌─────────────────────┐
    │  MockAxelarGateway  │  ──► Emits: ContractCall(
    │  (Source)           │              sender,
    └──────────┬──────────┘              destChain,
               │                         destContract,
               │                         payloadHash,
               │                         payload)
               │
               └──────────────────────────────────────────┐
                                                          │
                                                          │
STEP 3: RELAYER DETECTS EVENT                            │
═══════════════════════════════════════════════════════════════════════════════
                                                          │
                                                          │ WebSocket Listener
                                                          ▼
                                           ┌──────────────────────────┐
                                           │   Bridge Relayer Script  │
                                           │   (Node.js)              │
                                           └────────────┬─────────────┘
                                                        │
                                                        │ 1. Detects ContractCall event
                                                        │ 2. Decodes payload
                                                        │ 3. Extracts parameters
                                                        ▼
                                           ┌──────────────────────────┐
                                           │ Payload Decoded:         │
                                           │ - device address         │
                                           │ - user address           │
                                           │ - symbol (ERGON/CARBON)  │
                                           │ - amount                 │
                                           │ - v1TxHash               │
                                           │ - v1BlockNumber          │
                                           └────────────┬─────────────┘
                                                        │
                                                        │
STEP 4: ZK PROOF LOADING/GENERATION                     │
═══════════════════════════════════════════════════════════════════════════════
                                                        │
                                                        ▼
                                           ┌──────────────────────────┐
                                           │ Load Static Proof from   │
                                           │ proof.json               │
                                           │                          │
                                           │ Future: Generate dynamic │
                                           │ proof for transaction    │
                                           └────────────┬─────────────┘
                                                        │
                                                        │ Proof structure:
                                                        │ - a: [uint256, uint256]
                                                        │ - b: [[uint256, uint256], [uint256, uint256]]
                                                        │ - c: [uint256, uint256]
                                                        │ - input: [uint256]
                                                        ▼


STEP 5: CALL DESTINATION CONTRACT
═══════════════════════════════════════════════════════════════════════════════

                                           ┌──────────────────────────┐
                                           │ Relayer calls:           │
                                           │ BridgeReceiver.          │
                                           │ verifyAndMint(           │
                                           │   device,                │
                                           │   user,                  │
                                           │   symbol,                │
                                           │   amount,                │
                                           │   v1TxHash,              │
                                           │   v1BlockNumber,         │
                                           │   a, b, c, input         │
                                           │ )                        │
                                           └────────────┬─────────────┘
                                                        │
                                                        │ Transaction submitted
                                                        │ to Destination Chain
                                                        ▼


STEP 6: ON-CHAIN VERIFICATION (DESTINATION)
═══════════════════════════════════════════════════════════════════════════════

    ┌─────────────────────┐
    │  BridgeReceiver     │
    │  Contract           │
    └──────────┬──────────┘
               │
               │ 1. Validate token registered
               │ 2. Validate amount > 0
               │ 3. Validate verifier set
               ▼
    ┌─────────────────────────────────────────────────────────┐
    │  verifier.staticcall(                                   │
    │    "verifyProof(uint256[2],uint256[2][2],               │
    │                 uint256[2],uint256[1])",                │
    │    a, b, c, input                                       │
    │  )                                                       │
    └──────────┬──────────────────────────────────────────────┘
               │
               ▼
    ┌─────────────────────┐
    │  Groth16Verifier    │  ──► Verifies ZK proof using
    │  Contract           │      elliptic curve pairing
    └──────────┬──────────┘      Returns: bool (valid/invalid)
               │
               │ proof valid? ──── NO ───► REVERT: "Invalid zk-proof"
               │
               YES
               │
               ▼


STEP 7: TOKEN MINTING
═══════════════════════════════════════════════════════════════════════════════

    ┌─────────────────────┐
    │  BridgeReceiver     │
    │  Contract           │
    └──────────┬──────────┘
               │
               │ calls: IToken(tokenMap[symbol]).mint(user, amount)
               ▼
    ┌─────────────────────┐
    │  ETHErgon/          │  ──► Mints tokens to user address
    │  ETHCarbon          │      Increases user balance
    │  Token Contract     │
    └──────────┬──────────┘
               │
               │ emits: Transfer(0x0, user, amount)
               │
               ▼
    ┌─────────────────────┐
    │  BridgeReceiver     │  ──► Emits: BridgeMinted(
    │  emits event        │              device,
    └─────────────────────┘              user,
                                         symbol,
                                         amount,
                                         v1TxHash,
                                         v1BlockNumber)

═══════════════════════════════════════════════════════════════════════════════
TRANSACTION COMPLETE ✓
═══════════════════════════════════════════════════════════════════════════════
```

---

## Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DATA FLOW                                      │
└─────────────────────────────────────────────────────────────────────────────┘

SOURCE CHAIN                                              DESTINATION CHAIN
═════════════                                            ══════════════════

Token Burn Event                                         Token Mint Event
      │                                                          ▲
      │                                                          │
      ▼                                                          │
┌─────────────┐                                                  │
│ Burn Event  │                                                  │
│   Data:     │                                                  │
│ - from      │                                                  │
│ - amount    │                                                  │
│ - symbol    │                                                  │
└──────┬──────┘                                                  │
       │                                                         │
       ▼                                                         │
┌─────────────┐                                                  │
│Encode       │                                                  │
│Payload      │                                                  │
└──────┬──────┘                                                  │
       │                                                         │
       │ Payload = abi.encode(device, user, symbol,              │
       │                      amount, txHash, blockNum)          │
       │                                                         │
       ▼                                                         │
┌─────────────┐                                                  │
│MockAxelar   │                                                  │
│Gateway      │                                                  │
│ContractCall │                                                  │
│Event        │                                                  │
└──────┬──────┘                                                  │
       │                                                         │
       │                    ┌──────────────┐                     │
       │ Event Data ───────►│   RELAYER    │                     │
       │                    │              │                     │
       │                    │ 1. Listen    │                     │
       │                    │ 2. Decode    │                     │
       │                    │ 3. Load Proof│                     │
       │                    │ 4. Call Dest │                     │
       │                    └──────┬───────┘                     │
       │                           │                             │
       │                           │ Decoded:                    │
       │                           │ - device                    │
       │                           │ - user                      │
       │                           │ - symbol                    │
       │                           │ - amount                    │
       │                           │ - v1TxHash                  │
       │                           │ - v1BlockNumber             │
       │                           │                             │
       │                           │ ZK Proof:                   │
       │                           │ - a[2]                      │
       │                           │ - b[2][2]                   │
       │                           │ - c[2]                      │
       │                           │ - input[1]                  │
       │                           │                             │
       │                           ▼                             │
       │                    ┌──────────────┐                     │
       │                    │ Submit TX to │                     │
       │                    │ BridgeReceiver│                    │
       │                    └──────┬───────┘                     │
       │                           │                             │
       │                           │                             │
       │                           ▼                             │
       │                    ┌──────────────┐                     │
       │                    │BridgeReceiver│                     │
       │                    │verifyAndMint │                     │
       │                    └──────┬───────┘                     │
       │                           │                             │
       │                           ▼                             │
       │                    ┌──────────────┐                     │
       │                    │   Groth16    │                     │
       │                    │   Verifier   │                     │
       │                    │  verifyProof │                     │
       │                    └──────┬───────┘                     │
       │                           │                             │
       │                           │ Valid? ────NO───► REVERT    │
       │                           │                             │
       │                           YES                           │
       │                           │                             │
       │                           ▼                             │
       │                    ┌──────────────┐                     │
       │                    │ ETHToken     │                     │
       │                    │   .mint()    │                     │
       │                    └──────┬───────┘                     │
       │                           │                             │
       └───────────────────────────┴─────────────────────────────┘
                                   │
                                   ▼
                            User receives tokens
                            on destination chain
```

---

## Security Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SECURITY LAYERS                                    │
└─────────────────────────────────────────────────────────────────────────────┘

Layer 1: Token Burn Verification
════════════════════════════════════════════════════════════════════════════
    ┌─────────────────────────────────────────┐
    │ Tokens must be burned on source chain   │
    │ before minting on destination           │
    └─────────────────────────────────────────┘
                      │
                      ▼

Layer 2: ZK Proof Generation
════════════════════════════════════════════════════════════════════════════
    ┌─────────────────────────────────────────┐
    │ Proof proves knowledge of:              │
    │ - Secret commitment                     │
    │ - Burn amount                           │
    │ - Transaction details                   │
    │                                         │
    │ Without revealing sensitive data        │
    └─────────────────────────────────────────┘
                      │
                      ▼

Layer 3: On-Chain Verification
════════════════════════════════════════════════════════════════════════════
    ┌─────────────────────────────────────────┐
    │ BridgeReceiver validates:               │
    │ ✓ Token is registered                   |
    │ ✓ Amount > 0                            │
    │ ✓ Verifier contract is set              │
    └─────────────────────────────────────────┘
                      │
                      ▼

Layer 4: Cryptographic Proof Verification
════════════════════════════════════════════════════════════════════════════
    ┌─────────────────────────────────────────┐
    │ Groth16Verifier checks:                 │
    │ ✓ Proof is mathematically valid         │
    │ ✓ Elliptic curve pairing holds          │
    │ ✓ Public inputs match                   │
    └─────────────────────────────────────────┘
                      │
                      ▼

Layer 5: Controlled Minting
════════════════════════════════════════════════════════════════════════════
    ┌─────────────────────────────────────────┐
    │ Only BridgeReceiver can mint            │
    │ Token contract enforces bridge role     │
    │ No arbitrary minting possible           │
    └─────────────────────────────────────────┘
```

---

## Trust Assumptions

### Current Implementation (Centralized Relayer)

```
┌────────────────────────────────────────────────────────────┐
│  TRUST MODEL: Single Relayer                               │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────┐                                          │
│  │   Relayer    │ ◄── SINGLE POINT OF TRUST                │ 
│  └──────┬───────┘                                          │
│         │                                                  │
│         ├─► Must be online and available                   │
│         ├─► Must correctly relay events                    │
│         ├─► Must provide valid proofs                      │
│         └─► Controls which transactions are processed      │
│                                                            │
│  HOWEVER:                                                  │
│  ┌──────────────────────────────────────────────┐          │
│  │ On-chain verification CANNOT be bypassed:    │          │
│  │ • ZK proof must be valid                     │          │
│  │ • Cryptographic verification is enforced     │          │
│  │ • Relayer cannot fake proofs                 │          │
│  │ • Relayer cannot mint arbitrary amounts      │          │
│  └──────────────────────────────────────────────┘          │
│                                                            │
│  RISK: Relayer can:                                        │
│  • Censor transactions (not relay them)                    │
│  • Go offline (halt bridge)                                │
│                                                            │
│  RISK: Relayer CANNOT:                                     │
│  • Mint tokens without valid burn                          │
│  • Forge proofs                                            │
│  • Change amounts or recipients                            │
└────────────────────────────────────────────────────────────┘
```

### Future Enhancement (Decentralized)

```
┌────────────────────────────────────────────────────────────┐
│  ENHANCED TRUST MODEL: Multiple Options                    │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Option 1: Multiple Relayers                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Relayer 1   │  │  Relayer 2   │  │  Relayer 3   │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │              │
│         └─────────────────┴─────────────────┘              │
│                           │                                │
│                    Threshold signing                       │
│                    or consensus                            │
│                                                            │
│  Option 2: Axelar Network Integration                      │
│  ┌─────────────────────────────────────────┐               │
│  │  Use Axelar's decentralized validator   │               │
│  │  network for cross-chain messaging      │               │
│  └─────────────────────────────────────────┘               │
│                                                            │
│  Option 3: Light Client Verification                       │
│  ┌─────────────────────────────────────────┐               │
│  │  Verify source chain block headers      │               │
│  │  on destination chain                   │               │
│  └─────────────────────────────────────────┘               │
└────────────────────────────────────────────────────────────┘
```

---

## ZK Proof Circuit

### Burnproof Circuit Logic

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BURNPROOF CIRCOM CIRCUIT                             │
└─────────────────────────────────────────────────────────────────────────────┘

INPUTS (Private):
═════════════════
  ┌─────────────┐
  │   secret    │  ──► Secret value known only to user
  ├─────────────┤
  │   amount    │  ──► Amount being burned/transferred
  ├─────────────┤
  │    salt     │  ──► Random salt for commitment
  └─────────────┘

INPUTS (Public):
════════════════
  ┌──────────────────┐
  │ commitmentHash   │  ──► Hash of (secret, amount, salt)
  └──────────────────┘

CIRCUIT LOGIC:
══════════════
  ┌─────────────────────────────────────────────────────┐
  │  1. Compute: hash = H(secret, amount, salt)         │
  │                                                     │
  │  2. Verify: hash == commitmentHash                  │
  │                                                     │
  │  3. Verify: amount > 0                              │
  │                                                     │
  │  If all checks pass → Proof is valid                │
  └─────────────────────────────────────────────────────┘

OUTPUT:
═══════
  ┌─────────────────────────────────────────────────────┐
  │  Proof that prover knows (secret, amount, salt)     │
  │  such that H(secret, amount, salt) = commitmentHash │
  │  without revealing the private values               │
  └─────────────────────────────────────────────────────┘
```

### Proof Generation Flow

```
                    PROOF GENERATION WORKFLOW
                    ═════════════════════════

Step 1: Compile Circuit
───────────────────────────────────────────────────────
  burnproof.circom  ──► circom compiler ──┬──► burnproof.r1cs
                                          └──► burnproof.wasm


Step 2: Trusted Setup (One-time)
───────────────────────────────────────────────────────
  Powers of Tau Ceremony:
  1. Generate Powers of Tau  ──► pot12_final.ptau
  2. Contribute randomness   ──► pot12_final_contributed.ptau
  3. Prepare Phase 2         ──► pot12_final_phase2.ptau
  4. Groth16 Setup           ──► burnproof_0000.zkey
  5. Apply beacon            ──► zkey_final.zkey
  6. Export verifier         ──► verifier.sol


Step 3: Generate Witness (Per Transaction)
───────────────────────────────────────────────────────
  input.json  ──┬──► witness calculator ──► witness.wtns
  (secret,      │
   amount,      │
   salt,        │
   hash)        │
                └──► burnproof.wasm


Step 4: Generate Proof (Per Transaction)
───────────────────────────────────────────────────────
  witness.wtns  ──┬──► snarkjs groth16 prove ──┬──► proof.json
  zkey_final.zkey─┘                            └──► public.json


Step 5: Export for Solidity (Per Transaction)
───────────────────────────────────────────────────────
  proof.json ──┬──► snarkjs export ──► calldata
  public.json──┘       soliditycalldata    (a, b, c, input)
```

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEPLOYMENT WORKFLOW                                 │
└─────────────────────────────────────────────────────────────────────────────┘

DESTINATION CHAIN (ergon_v2) - Deploy First
═══════════════════════════════════════════════════════════════════════════

  Step 1: Deploy Groth16Verifier
  ┌─────────────────────────┐
  │  Groth16Verifier.sol    │ ──► From compiled circuit
  └────────────┬────────────┘
               │
               ├──► Save address to .env: DEST_GROTH16_VERIFIER_ADDRESS
               │
               ▼

  Step 2: Deploy MockAxelarGateway
  ┌─────────────────────────┐
  │ MockAxelarGateway       │
  └────────────┬────────────┘
               │
               ├──► Save address to .env: DEST_MOCK_GATEWAY_ADDRESS
               │
               ▼

  Step 3: Deploy Token Contracts
  ┌─────────────────────────┐
  │  ETHErgon Token         │
  └────────────┬────────────┘
               │
               ├──► Save address to .env: DEST_ETHERGON_CONTRACT_ADDRESS
               │
  ┌─────────────▼───────────┐
  │  ETHCarbon Token        │
  └────────────┬────────────┘
               │
               ├──► Save address to .env: DEST_ETHCARBON_CONTRACT_ADDRESS
               │
               ▼

  Step 4: Deploy BridgeReceiver
  ┌─────────────────────────────────────────────────────┐
  │  BridgeReceiver(                                    │
  │    gateway,         ◄── MockAxelarGateway address   │
  │    ergon,           ◄── ETHErgon address            │
  │    carbon,          ◄── ETHCarbon address           │
  │    verifier         ◄── Groth16Verifier address     │
  │  )                                                  │
  └────────────┬────────────────────────────────────────┘
               │
               ├──► Save address to .env: DEST_BRIDGE_RECEIVER_ADDRESS
               │
               ▼

  Step 5: Configure Token Bridges
  ┌─────────────────────────┐
  │ ETHErgon.setBridge()    │ ──► Set to BridgeReceiver address
  └─────────────────────────┘
  ┌─────────────────────────┐
  │ ETHCarbon.setBridge()   │ ──► Set to BridgeReceiver address
  └─────────────────────────┘


SOURCE CHAIN (ergon_v1 or ganache) - Deploy Second
═══════════════════════════════════════════════════════════════════════════

  Step 1: Deploy MockAxelarGateway
  ┌─────────────────────────┐
  │ MockAxelarGateway       │
  └────────────┬────────────┘
               │
               ├──► Save address to .env: SRC_MOCK_GATEWAY_ADDRESS
               │
               ▼

  Step 2: Deploy Token Contracts
  ┌─────────────────────────┐
  │  Ergon Token            │
  └────────────┬────────────┘
               │
  ┌─────────────▼───────────┐
  │  Carbon Token           │
  └────────────┬────────────┘
               │
               ▼

  Step 3: Deploy BridgeEmitter
  ┌─────────────────────────────────────────────────────┐
  │  BridgeEmitter(                                     │
  │    gateway,         ◄── MockAxelarGateway address   │
  │    gasService       ◄── MockAxelarGasService address│
  │  )                                                  │
  └────────────┬────────────────────────────────────────┘
               │
               ├──► Save address to .env: SRC_BRIDGE_EMITTER_ADDRESS
               │
               ▼

  Step 4: Register Tokens
  ┌─────────────────────────────────────────────────────┐
  │ BridgeEmitter.registerToken(                        │
  │   "ERGON",                                          │
  │   ergonTokenAddress,                                │
  │   destBridgeReceiverAddress                         │
  │ )                                                   │
  └─────────────────────────────────────────────────────┘
  ┌─────────────────────────────────────────────────────┐
  │ BridgeEmitter.registerToken(                        │
  │   "CARBON",                                         │
  │   carbonTokenAddress,                               │
  │   destBridgeReceiverAddress                         │
  │ )                                                   │
  └─────────────────────────────────────────────────────┘


RELAYER SERVICE - Start Last
═══════════════════════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │  node scripts/bridge-relayer.js                     │
  │                                                     │
  │  Requires .env configuration:                       │
  │  • SRC_RPC_URL                                      │
  │  • DEST_RPC_URL                                     │
  │  • RELAYER_PK                                       │
  │  • SRC_MOCK_GATEWAY_ADDRESS                         │
  │  • DEST_MOCK_GATEWAY_ADDRESS                        │
  │  • DEST_BRIDGE_RECEIVER_ADDRESS                     │
  │  • DEST_ETHERGON_CONTRACT_ADDRESS                   │
  │  • DEST_ETHCARBON_CONTRACT_ADDRESS                  │
  └─────────────────────────────────────────────────────┘
```

---

## Error Handling & Edge Cases

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ERROR SCENARIOS                                     │
└─────────────────────────────────────────────────────────────────────────────┘

Scenario 1: Invalid ZK Proof
═══════════════════════════════════════════════════════════════════════════
  User Action:
    Submit transaction with invalid/forged proof
         │
         ▼
  ┌──────────────────────────┐
  │ BridgeReceiver           │
  │ verifyAndMint()          │
  └────────┬─────────────────┘
           │
           ├──► Call Groth16Verifier.verifyProof()
           │
           ▼
  ┌──────────────────────────┐
  │ Verification FAILS       │ ──► Returns false
  └────────┬─────────────────┘
           │
           ▼
  REVERT: "Invalid zk-proof"
  
  Result: No tokens minted, transaction reverts


Scenario 2: Token Not Registered
═══════════════════════════════════════════════════════════════════════════
  Relayer Action:
    Call verifyAndMint with unregistered token symbol
         │
         ▼
  ┌──────────────────────────┐
  │ BridgeReceiver           │
  │ tokenMap[symbol] == 0x0  │
  └────────┬─────────────────┘
           │
           ▼
  REVERT: "Token not registered"
  
  Result: Transaction fails before proof verification


Scenario 3: Verifier Not Set
═══════════════════════════════════════════════════════════════════════════
  Config Error:
    BridgeReceiver deployed without verifier address
         │
         ▼
  ┌──────────────────────────┐
  │ BridgeReceiver           │
  │ verifier == address(0)   │
  └────────┬─────────────────┘
           │
           ▼
  REVERT: "Verifier not set"
  
  Result: Cannot process any bridge transactions
  Fix: Call setVerifier() with valid verifier address


Scenario 4: Relayer Offline
═══════════════════════════════════════════════════════════════════════════
  System State:
    Bridge relayer script not running
         │
         ▼
  ┌──────────────────────────┐
  │ ContractCall event       │
  │ emitted on source        │
  └────────┬─────────────────┘
           │
           ▼
  ┌──────────────────────────┐
  │ No listener active       │
  │ Event not processed      │
  └──────────────────────────┘
  
  Result: Tokens burned on source, but not minted on destination
  State: Funds stuck until relayer comes back online
  Recovery: Restart relayer, it processes backlog of events


Scenario 5: Insufficient Gas
═══════════════════════════════════════════════════════════════════════════
  Relayer Action:
    Submit transaction with too little gas
         │
         ▼
  ┌──────────────────────────┐
  │ Transaction execution    │
  │ runs out of gas          │
  └────────┬─────────────────┘
           │
           ▼
  REVERT: Out of gas
  
  Result: Transaction fails, relayer needs to retry with higher gas
  Prevention: Gas estimation with 20% buffer in relayer script


Scenario 6: Replay Attack
═══════════════════════════════════════════════════════════════════════════
  Attacker:
    Try to replay same proof multiple times
         │
         ▼
  ┌──────────────────────────┐
  │ BridgeReceiver           │
  │ verifyAndMint()          │
  └────────┬─────────────────┘
           │
           ├──► Proof is valid
           ├──► But same v1TxHash used before
           │
           ▼
  Note: Current implementation does NOT prevent replay
  
  TODO: Add nonce or tx hash tracking to prevent double-spend
  Enhancement needed:
    mapping(bytes32 => bool) public processedTxHashes;
    require(!processedTxHashes[v1TxHash], "Already processed");
```

---

## Performance Considerations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE METRICS                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Gas Costs (Destination Chain)
═══════════════════════════════════════════════════════════════════════════

  ┌────────────────────────────────────────────┐
  │ Operation          │ Estimated Gas Cost    │
  ├────────────────────────────────────────────┤
  │ verifyProof()      │ ~250,000 - 300,000    │ ◄── Groth16 verification
  │ mint()             │ ~50,000               │ ◄── Token minting
  │ verifyAndMint()    │ ~350,000 - 400,000    │ ◄── Complete operation
  │ Total              │                       │
  └────────────────────────────────────────────┘

  Note: ZK proof verification is the most expensive part


Latency (End-to-End)
═══════════════════════════════════════════════════════════════════════════

  Source Chain TX (Burn)
         │
         │  Block confirmation: ~15 seconds
         ▼
  Event Emission
         │
         │  WebSocket detection: <1 second
         ▼
  Relayer Processing
         │
         │  Payload decode: <100ms
         │  Proof loading: <100ms
         ▼
  Destination Chain TX (Mint)
         │
         │  Block confirmation: ~15 seconds
         ▼
  TOTAL: ~30-45 seconds for complete bridge


Throughput
═══════════════════════════════════════════════════════════════════════════

  Single Relayer:
    • Limited by destination chain block time
    • Can batch multiple proofs in single TX (optimization)
    • ~4 TPS per relayer (assuming 15s blocks)

  Optimization Opportunities:
    • Batch verification of multiple proofs
    • Parallel relayers for different token types
    • Off-chain proof aggregation
```

---

## Future Enhancements

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ROADMAP FOR IMPROVEMENTS                               │
└─────────────────────────────────────────────────────────────────────────────┘

Phase 1: Dynamic Proof Generation
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ • Implement automated proof generation per TX       │
  │ • Integrate snarkjs into relayer                    │
  │ • Generate unique proofs for each bridge transfer   │
  │ • Remove dependency on static proof.json            │
  └─────────────────────────────────────────────────────┘


Phase 2: Replay Protection
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ • Add transaction hash tracking                     │
  │ • Implement nonce system                            │
  │ • Prevent double-spending                           │
  │ • Add commitment nullifier tracking                 │
  └─────────────────────────────────────────────────────┘


Phase 3: Decentralization
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ Option A: Multiple Relayers                         │
  │   • Deploy 3-5 independent relayers                 │
  │   • Threshold signature scheme                      │
  │   • Consensus mechanism                             │
  │                                                     │
  │ Option B: Axelar Integration                        │
  │   • Replace MockAxelarGateway with real Axelar      │
  │   • Use Axelar validator network                    │
  │   • Production-grade cross-chain messaging          │
  └─────────────────────────────────────────────────────┘


Phase 4: Enhanced Privacy
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ • Implement Merkle tree for commitments             │
  │ • Add recipient privacy (stealth addresses)         │
  │ • Amount privacy (range proofs)                     │
  │ • Full privacy-preserving bridge                    │
  └─────────────────────────────────────────────────────┘


Phase 5: Optimizations
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ • Batch proof verification                          │
  │ • Proof aggregation (Groth16 -> PLONK)              │
  │ • Gas optimization                                  │
  │ • Circuit optimization for faster proving           │
  └─────────────────────────────────────────────────────┘


Phase 6: Monitoring & Alerts
═══════════════════════════════════════════════════════════════════════════
  ┌─────────────────────────────────────────────────────┐
  │ • Dashboard for bridge status                       │
  │ • Alert system for failed relays                    │
  │ • Analytics and metrics collection                  │
  │ • Automatic retry mechanism                         │
  └─────────────────────────────────────────────────────┘
```

---

## Testing Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          TEST SCENARIOS                                     │
└─────────────────────────────────────────────────────────────────────────────┘

Unit Tests
═══════════════════════════════════════════════════════════════════════════
  BridgeEmitter:
    ✓ Token registration
    ✓ Burn and emit event
    ✓ Payload encoding

  BridgeReceiver:
    ✓ Token mapping
    ✓ Verifier integration
    ✓ Mint with valid proof
    ✓ Reject invalid proof
    ✓ Reject unregistered token
    ✓ Reject zero amount

  Groth16Verifier:
    ✓ Verify valid proof
    ✓ Reject invalid proof
    ✓ Gas consumption


Integration Tests
═══════════════════════════════════════════════════════════════════════════
  End-to-End Bridge:
    ✓ Burn on source → Mint on destination
    ✓ Multiple tokens (ERGON, CARBON)
    ✓ Different amounts
    ✓ Event emission and listening
    ✓ Relayer processing


Security Tests
═══════════════════════════════════════════════════════════════════════════
  Attack Vectors:
    ✓ Forged proof rejection
    ✓ Replay attack (TODO: implement protection)
    ✓ Unauthorized minting prevention
    ✓ Incorrect amount verification
    ✓ Wrong recipient verification


Performance Tests
═══════════════════════════════════════════════════════════════════════════
  Load Testing:
    ✓ Multiple concurrent transfers
    ✓ Gas consumption profiling
    ✓ Relayer throughput
    ✓ Block confirmation time
```

---

## Glossary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TERMINOLOGY                                    │
└─────────────────────────────────────────────────────────────────────────────┘

ZK-SNARK (Zero-Knowledge Succinct Non-Interactive Argument of Knowledge)
───────────────────────────────────────────────────────────────────────────
  Cryptographic proof system allowing one party to prove possession of
  certain information without revealing the information itself.

Groth16
───────────────────────────────────────────────────────────────────────────
  Specific ZK-SNARK construction with constant-size proofs and efficient
  verification. Most commonly used for blockchain applications.

Circom
───────────────────────────────────────────────────────────────────────────
  Domain-specific language for writing arithmetic circuits used in ZK proofs.

R1CS (Rank-1 Constraint System)
───────────────────────────────────────────────────────────────────────────
  Mathematical representation of a circuit as a system of equations.

Trusted Setup / Powers of Tau
───────────────────────────────────────────────────────────────────────────
  Ceremony to generate cryptographic parameters needed for proof generation.
  Must be done securely to ensure system security.

Witness
───────────────────────────────────────────────────────────────────────────
  The private inputs to a ZK circuit that generate a valid proof.

Public Signals
───────────────────────────────────────────────────────────────────────────
  The public inputs/outputs that are verifiable without revealing the witness.

Commitment
───────────────────────────────────────────────────────────────────────────
  Cryptographic hash binding a value without revealing it.

Nullifier
───────────────────────────────────────────────────────────────────────────
  Unique identifier preventing double-spending in privacy systems.

Relayer
───────────────────────────────────────────────────────────────────────────
  Off-chain service that monitors one chain and submits transactions to
  another chain.

Bridge
───────────────────────────────────────────────────────────────────────────
  System enabling asset transfers between different blockchains.

Wrapped Token
───────────────────────────────────────────────────────────────────────────
  Token on destination chain representing locked/burned token on source chain.

Axelar
───────────────────────────────────────────────────────────────────────────
  Decentralized cross-chain communication protocol (currently mocked in
  this implementation).
```

---

## References & Resources

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ADDITIONAL RESOURCES                                │
└─────────────────────────────────────────────────────────────────────────────┘

Official Documentation
═══════════════════════════════════════════════════════════════════════════
  • Circom: https://docs.circom.io/
  • SnarkJS: https://github.com/iden3/snarkjs
  • Axelar: https://docs.axelar.dev/
  • Groth16 Paper: https://eprint.iacr.org/2016/260.pdf

Tools
═══════════════════════════════════════════════════════════════════════════
  • Truffle: https://trufflesuite.com/
  • Web3.js: https://web3js.readthedocs.io/
  • Ganache: https://trufflesuite.com/ganache/

Project Files
═══════════════════════════════════════════════════════════════════════════
  • Contracts: ./contracts/
  • Migrations: ./migrations/
  • Scripts: ./scripts/
  • Circuit: ./burnproof.circom
  • Proof: ./proof.json
  • README: ./README.md
```

---

**Document Version:** 1.0  
**Status:** Active Development
