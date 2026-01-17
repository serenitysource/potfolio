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
**Enterprise-grade NFT marketplace for carbon credit trading with blockchain-backed provenance**

A comprehensive decentralized marketplace platform enabling transparent carbon credit tokenization, trading, and retirement through NFT certificates. Features include fixed-price sales, auction systems, admin analytics dashboard, and full provenance tracking.

#### Key Features:
- 🏪 **NFT Marketplace** - Fixed-price listings and time-based auctions
- 💰 **ERC20 Payment System** - Flexible token-based transactions
- 📊 **Admin Dashboard** - Real-time analytics, user management, and platform controls
- 🔐 **User Verification** - KYC-ready user validation system
- 📈 **Financial Analytics** - Revenue tracking, trading volume analysis, and reporting
- 🛡️ **Security Controls** - Pausable contracts, fee management, and emergency protocols
- 🚫 **User Management** - Blocking, verification, and risk assessment tools

#### Documentation:
- [📖 Marketplace Contract](./Carbon/docs/contracts/CarbonCreditMarketplace.md) - Complete smart contract documentation
- [🔧 API Reference](./Carbon/docs/api/CarbonCreditMarketplace.md) - Quick function lookup and usage
- [📊 Admin Dashboard Master Plan](./Carbon/docs/UI/ADMIN_DASHBOARD_MASTER_PLAN.md) - Enterprise dashboard vision
- [✅ Admin Dashboard Implementation](./Carbon/docs/UI/ADMIN_DASHBOARD_IMPLEMENTATION_PLAN.md) - Implementation status
- [🔌 Dashboard Architecture](./Carbon/docs/UI/ADMIN_DASHBOARD_CONNECTION_ARCHITECTURE.md) - Technical architecture
- [🛑 Blocked Users Integration](./Carbon/docs/UI/BLOCKED_USERS_INTEGRATION.md) - User restriction system

**Tech Stack:** Solidity, React, Material-UI, ethers.js, Node.js, WebSockets

---

### 📜 Provenance System
**End-to-end blockchain provenance design for energy token ecosystems**

A comprehensive audit-ready provenance system connecting IoT energy devices to carbon credit NFTs through deterministic hashing, event-centric architecture, and cross-chain bridging.

- <a>📖 Full Documentation</a> - Complete provenance architecture and flow

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
- NFT Marketplaces & Auctions

</td>
<td>

**Backend & Infrastructure**
- Node.js & Python
- Docker & Containerization
- MQTT & IoT Protocols
- REST APIs (FastAPI)
- Event-Driven Architecture
- Real-time Analytics & Dashboards

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
✅ **NFT Marketplace Development** - Enterprise-grade trading platforms with admin dashboards  

---

## 📊 Repository Structure

```
potfolio/
├── Bridge/                          # Cross-chain bridge system
│   ├── CrossChainBridge.md         # Complete flow documentation
│   ├── BridgeRelayer.md             # Relayer script documentation
│   └── Contracts/                   # Smart contract documentation
│       ├── Gateway.md               # Mock Axelar Gateway
│       ├── BridgeEmitter            # Source chain contract
│       └── BridgeReceiver.md        # Destination chain contract
├── Carbon/                          # Carbon credit marketplace
│   └── docs/
│       ├── api/                     # API references
│       │   └── CarbonCreditMarketplace.md
│       ├── contracts/               # Contract documentation
│       │   └── CarbonCreditMarketplace.md
│       └── UI/                      # UI integration guides
│           ├── ADMIN_DASHBOARD_MASTER_PLAN.md
│           ├── ADMIN_DASHBOARD_IMPLEMENTATION_PLAN.md
│           ├── ADMIN_DASHBOARD_INTEGRATION_COMPLETE.md
│           ├── ADMIN_DASHBOARD_CONNECTION_ARCHITECTURE.md
│           └── BLOCKED_USERS_INTEGRATION.md
├── Hepek/                           # Web3 MQTT Master platform
│   └── HepekWeb3MQTTDevice.md       # Platform documentation
├── Provenance/                      # Provenance system design
│   └── README.md                    # End-to-end provenance architecture
└── README.md                        # This file
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