# 🚀 SerenitySource Portfolio

Welcome to my portfolio repository! This collection showcases blockchain, IoT, and cross-chain infrastructure projects I've worked on. Each project demonstrates expertise in Web3, smart contracts, zero-knowledge proofs, and real-time IoT systems.

---

## 📂 Projects

### 🌉 Cross-Chain Bridge System
**A secure token bridge between private and public EVM networks using zero-knowledge proofs**

This project enables cross-chain token transfers with ZK-proof verification, simulating Axelar network functionality for local development and testing environments.

#### Key Features:
- 🔐 Zero-knowledge proof verification (Groth16)
- 🔗 Cross-chain messaging simulation
- 🎯 Token locking/burning mechanism
- 📡 Off-chain relayer service
- ⚡ Mock Axelar Gateway implementation

#### Documentation:
- [📖 Complete Bridge Flow](./Bridge/CrossChainBridge.md) - End-to-end system architecture
- [🔧 Bridge Relayer](./Bridge/BridgeRelayer.md) - Off-chain relayer script documentation
- [📝 Smart Contracts](./Bridge/Contracts/):
  - [Gateway Contract](./Bridge/Contracts/Gateway.md) - Mock Axelar Gateway
  - [Bridge Emitter](./Bridge/Contracts/BridgeEmitter) - Source chain contract
  - [Bridge Receiver](./Bridge/Contracts/BridgeReceiver.md) - Destination chain contract

**Tech Stack:** Solidity, Web3.js, Node.js, Ethereum, Zero-Knowledge Proofs (zkSNARKs)

---

### ⚡ Hepek Web3 MQTT Master
**All-in-one blockchain-enabled smart meter platform for real-time energy monitoring**

A Dockerized platform integrating MQTT, Web3, and IoT protocols to deliver real-time energy data with blockchain transaction capabilities.

#### Key Features:
- 🔗 Web3 blockchain integration for energy tokens
- 📡 Real-time MQTT data streaming
- 🐳 Fully Dockerized deployment
- 🔒 Encrypted secrets management
- ⚡ Modbus industrial protocol support
- 🛠️ Auto-deploy capabilities

#### Documentation: 
- [📖 Full Documentation](./Hepek/HepekWeb3MQTTDevice.md) - Complete platform overview and setup guide

**Tech Stack:** Python 3.10, FastAPI, Web3.py, MQTT (paho-mqtt), Docker, Modbus

---

### 🌱 Carbon Credit Marketplace
**Decentralized carbon credit trading platform with NFT-based certificates**

A blockchain-based marketplace for trading carbon credits as NFTs, featuring auction functionality, admin controls, and comprehensive provenance tracking.

#### Key Features:
- 💰 Fixed price listings and auctions
- 🎨 NFT-based carbon credit certificates
- 🔒 Admin controls and fee management
- 📊 Real-time tracking and transparency
- 🛡️ Pausable contract for emergency stops
- 💳 ERC20 payment token integration

#### Documentation:
- [📖 Contract Documentation](./Carbon/docs/contracts/CarbonCreditMarketplace.md) - Complete smart contract reference
- [🔌 API Reference](./Carbon/docs/api/CarbonCreditMarketplace.md) - Function signatures and events
- [🎨 UI Implementation](./Carbon/docs/UI/) - Admin dashboard and integration guides

**Tech Stack:** Solidity, ERC721 (NFTs), ERC20 (Payments), Smart Contract Security

---

### 🔐 Provenance System
**End-to-end provenance implementation for energy token integrity**

An audit-ready, event-centric provenance system that delivers fast user UX with strong cryptographic anchoring. The system tracks energy token provenance from device readings through bridging to NFT certificate minting.

#### Key Features:
- ⚡ Real-time UX with single-transaction bridging
- 🔒 Audit-grade provenance with deterministic hashing
- 📊 Minimal on-chain load with off-chain detail retention
- 🔗 Device-to-NFT traceability
- 🌳 Merkle tree commitments for period aggregation
- 📢 Event-driven architecture

#### Documentation:
- [📖 Complete Implementation Guide](./Provenance/README.md) - Full system design and architecture
- [🔄 Provenance Flow](./Provenance/Provenance.md) - Cryptographic proof chain details

**Tech Stack:** Solidity, Event Sourcing, Merkle Trees, IPFS, Keccak256 Hashing

---

## 🛠️ Technical Expertise

<table>
<tr>
<td>

**Blockchain & Web3**
- Solidity Smart Contracts
- Web3.js/Web3.py
- Cross-chain Bridges
- Zero-Knowledge Proofs
- Ethereum/EVM Networks

</td>
<td>

**Backend & Infrastructure**
- Node.js & Python
- Docker & Containerization
- MQTT & IoT Protocols
- REST APIs (FastAPI)
- Event-Driven Architecture

</td>
</tr>
</table>

---

## 🎯 What This Portfolio Demonstrates

✅ **Smart Contract Development** - Complex multi-contract systems with security best practices  
✅ **Cross-Chain Infrastructure** - Bridge design, relayer services, and cross-chain messaging  
✅ **Zero-Knowledge Proofs** - ZK-SNARK integration for privacy and verification  
✅ **IoT & Real-Time Systems** - MQTT, Modbus, and real-time data streaming  
✅ **DevOps & Deployment** - Docker, CI/CD, and automated deployment pipelines  
✅ **System Architecture** - End-to-end system design and documentation  

---

## 📊 Repository Structure

```
potfolio/
├── Bridge/                          # Cross-chain bridge system
│   ├── CrossChainBridge.md         # Complete flow documentation
│   ├── BridgeRelayer.md            # Relayer script documentation
│   └── Contracts/                  # Smart contract documentation
│       ├── Gateway.md              # Mock Axelar Gateway
│       ├── BridgeEmitter           # Source chain contract
│       └── BridgeReceiver.md       # Destination chain contract
├── Carbon/                          # Carbon credit marketplace
│   └── docs/                       # Documentation
│       ├── contracts/              # Smart contract reference
│       ├── api/                    # API documentation
│       └── UI/                     # UI implementation guides
├── Hepek/                          # Web3 MQTT Master platform
│   └── HepekWeb3MQTTDevice.md      # Platform documentation
├── Provenance/                     # Provenance system
│   ├── README.md                   # Implementation guide
│   └── Provenance.md               # Cryptographic proof chain
└── README.md                       # This file
```

---

## 🌐 Connect & Explore

Each project includes detailed documentation covering:
- System architecture and design decisions
- Implementation details and code walkthrough
- Setup instructions and deployment guides
- Testing strategies and troubleshooting

Navigate to the specific project folders to dive deeper into each implementation! 

---

## 📝 About

This portfolio represents real-world projects combining blockchain technology, smart contracts, IoT systems, and infrastructure automation. All documentation is maintained to provide clear insights into the technical implementations and architectural decisions. 

---

<div align="center">

**Built with 🔥 by SerenitySource**

*Blockchain • Web3 • IoT • Smart Contracts*

</div>