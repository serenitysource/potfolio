# End-to-End Provenance Design (HEPEK → ErgonEnergy → Carbon → Bridge → ETHCarbon → Vault → CarbonCreditNFT)

This README consolidates the full design + flow narrative for your **audit-ready, event-centric provenance system** with **fast user UX** (single bridge tx, no per-lot loops) and **strong provenance** anchored on the source chain + database.

---

## Goals

You want:

1. **Real-time UX**
   - ERC20 transfers are instant/fungible.
   - Bridging is **single-transaction** (no “lot loop” prompting MetaMask 50 times).
2. **Audit-grade provenance**
   - Prove **which devices** contributed to a Carbon Credit Certificate NFT mint.
   - Prove approximate **shares per device** (e.g., “20% from device A, remainder split across B/C/D…”).
   - Optionally trace back to **minute-level meter readings** in your DB using deterministic hashes.

This design achieves that by:
- anchoring minute readings via a deterministic `reading_hash`,
- aggregating mints into **one rolling slot per device** (or per device+period),
- using events + optional roots (Merkle/commitments) to preserve auditability without slowing down bridging.

---

## Components & Responsibilities

### A) HEPEK device agent (`contract.py`)
- Measures energy deltas (minute-by-minute).
- Builds a **canonical payload**.
- Computes deterministic `reading_hash = keccak256(canonical_json)`.
- Calls:
  - `ErgonEnergy.SC_Power_OUT_Alloc(produced_power, export_power, reading_hash)`.

**Why this matters**
- `reading_hash` becomes your immutable “audit anchor” across:
  - MQTT payload
  - DB record
  - On-chain tx / event
- Enables on-chain anti-replay: the same hash can’t be accepted twice.

---

### B) `ErgonEnergy.sol` (energy accounting + mint trigger)
Conceptually:
- Validates/guards meter readings (anti-replay via `reading_hash`).
- Computes mint amounts for **ERGON** and **CARBON** based on deltas and your formula rules.
- Calls token contract minting.

This is the “truth boundary” where real-world energy becomes token supply.

---

### C) `Carbon.sol` (fungible token + provenance structure)
Move away from “minute lots” and toward:
- **one slot per device**, or
- **one slot per device per period** (daily/weekly/monthly)

This keeps:
- transfers + bridging fast (single amount),
- provenance verifiable at retirement time.

---

### D) BridgeEmitter / BridgeReceiver / Relayer
Bridge UX should be driven by **messageId correlation**.

**Source chain**
- `BridgeEmitter` emits `BridgeInitiatedV2(...)` which includes a deterministic `messageId`.

**Destination chain**
- `BridgeReceiver` emits:
  - `BridgeMinted(messageId, ...)` on success
  - `BridgeMintRejected(messageId, ...)` on failure

**Relayer**
- Watches source event → calls destination execute → then calls back success/failure to emitter (burn/refund finalization).

This is the correct “event-centric” architecture:
- UI doesn’t guess; it waits for deterministic events.

---

### E) `ETHCarbon.sol` (destination-chain ERC20 mirror)
Destination chain token exists for:
- user holdings on the destination chain
- marketplace / vault / NFT mint cycle

You do **not** need minute-level provenance replicated here.
Provenance remains:
- anchored on source chain and DB
- resolved into a compact proof package at NFT retirement time

---

### F) `PlatformVault` + `CarbonCreditNFT`
Vault is the “retirement gate”:
- tokens get deposited/approved
- vault burns tokens when minting certificate NFT
- NFT mint is the irreversible compliance action

Auditors care most about provenance **here**, at retirement.

---

## Key Design Principle

### Separate the system into two layers

#### 1) Token UX Layer (must be fast)
- ERC20 transfers between users should be standard.
- Bridge should move a **single fungible amount** per tx.
- No per-lot loops in UI.

#### 2) Provenance Layer (must be verifiable)
- Anchored on the **source chain** + DB.
- Uses:
  - deterministic reading hashes
  - period/device commitments (optional)
  - event logs as the public audit trail
- Provenance is *resolved* and *committed* at NFT mint time.

---

## Data Anchoring: `reading_hash`

### Is `reading_hash` enough?
Yes — if you also store it in:
- MQTT payload
- DB record
- on-chain (via tx or events)
- and it is reproducible deterministically

### Recommended canonical hash payload
Minimal + sufficient:

- `device_id`
- `timestamp` (stable bucket, e.g. epoch minute)
- `produced_power`
- `export_power`

Then:
- canonical JSON encoding (sorted keys, stable separators)
- keccak256 hash

### Include in MQTT payload
✅ Recommended:

```python
canonical_reading_hash = make_reading_hash(envelope)

mqtt_payload = {
    **(envelope.get("meter_reading") or {}),
    "device_id": envelope.get("device_id"),
    "timestamp": envelope.get("timestamp"),
    "metadata": envelope.get("metadata"),
    "canonical_reading_hash": canonical_reading_hash,
}
```

This makes correlation instant:
MQTT → DB → chain tx/event.

---

## Events: What to Emit and Why (Audit Log API)

Events are your “public audit trail” and your UI’s correctness anchor.

### 1) Reading accepted (ErgonEnergy)
Recommended events (choose based on privacy needs):

- Minimal:
  - `ReadingAccepted(device, user, readingHash, timestampBucket)`
- Full:
  - `ReadingAccepted(device, user, readingHash, produced, exported, timestampBucket)`

Why:
- Proves chain accepted exact `reading_hash`.
- Enables off-chain indexing and audit reconstruction.

---

### 2) Minting (ErgonEnergy → Carbon)
Recommended:
- `PowerOutAllocated(device, readingHash, ergonMinted, carbonMinted, factor, priceRefHash)`

Why:
- Links reading hash to minted value.

---

### 3) Carbon aggregation (Carbon.sol)
If using one rolling slot per device:
- `DeviceLotUpdated(device, newTotal, delta, periodId)`

If using period commitments:
- `DevicePeriodCommitted(device, periodId, periodSum, deviceRoot)`
- `PeriodGlobalCommitted(periodId, globalSum, globalRoot)`

Why:
- Keeps on-chain state compact
- Preserves tamper-evident commitments auditors can verify

---

### 4) Bridge lifecycle (Emitter/Receiver)
- Source: `BridgeInitiatedV2(messageId, ...)`
- Destination: `BridgeMinted(messageId, ...)` or `BridgeMintRejected(messageId, reason)`

Why:
- UI can reliably show completion/failure without polling or guessing.

---

## “One Slot per Device” Model (Your Plan)

### Meaning
Inside `Carbon.sol` you maintain something like:

- `deviceAggregate[device] += mintedDelta`

ERC20 user balances are still standard.

### Important: ERC20 transfers should NOT “move device slots”
Device slots describe **supply provenance**, not ownership tracking.

Therefore:
- user-to-user transfers remain normal
- provenance is resolved at retirement time

---

## Bridging with Fungibility: Where provenance should be enforced

Because CARBON is fungible:
- tracking perfect provenance through arbitrary user transfers is expensive and often unnecessary.

### Recommended convention
**Provenance is proven at NFT retirement time**, not at every transfer/bridge hop.

Bridging becomes:
- a fast fungible transfer across networks
- provenance remains source-of-truth on source chain + DB

---

## Provenance at NFT Mint (Retirement)

At NFT mint time (e.g., burning ~860 CARBON), generate a **Provenance Package** off-chain:

1. Choose policy:
   - FIFO by period
   - proportional by device balances
   - whitelist-only devices
2. Build:
   - `[{device, amount}, ...]`
3. (Optional) Build Merkle root over the minute readings used
4. Store package:
   - DB / IPFS
5. Commit compact reference:
   - `provenanceRoot = keccak256(devices[], amounts[], periodRange, policyId, …)`
   - store in NFT metadata and/or on-chain storage

This is sufficient for audit and keeps UX fast.

---

## Periodic commitments: Can the contract “auto-run” commits?

No — contracts can’t wake up by themselves.

If you use `commitDevicePeriod()` / `commitPeriodGlobal()`:
- a bot/relayer must call them on a schedule:
  - daily / weekly / monthly

Automation pattern:
- timer-driven service:
  - checks if period rolled over
  - submits commit tx

---

## Flow Diagrams

### 1) Runtime minting flow (minute-level)

```
[HEPEK Device]
  | measures delta (produced/exported)
  | builds canonical payload
  | reading_hash = keccak(canonical_json)
  v
[ErgonEnergy.SC_Power_OUT_Alloc(produced, exported, reading_hash)]
  | anti-replay: usedReading[reading_hash] = true
  | compute ERGON + CARBON mint amounts
  v
[Ergon Token]  mint(user, ergonAmount)
[Carbon Token] mint(user, carbonAmount) + update device aggregate bucket
  |
  +--> Events emitted for audit indexing
```

### 2) Bridge flow (single tx, event-correlated)

```
[Bridge App UI]
  | send tx: BridgeEmitter.bridgeMint(...)
  v
[BridgeEmitter]
  | emits BridgeInitiatedV2(messageId,...payloadHash)
  v
[Relayer]
  | sees BridgeInitiatedV2
  | calls Gateway -> BridgeReceiver.execute(...)
  v
[BridgeReceiver]
  | mints ETHCarbon
  | emits BridgeMinted(messageId,...)
  v
[Bridge App UI]
  | listens BridgeMinted -> DONE
```

### 3) NFT mint (retirement) provenance flow

```
[User / PlatformVault]
  | gather requiredTokens (eligibility rule)
  | build provenance package off-chain (devices + optional roots)
  | store package (DB/IPFS), get provenanceRoot/hash
  v
[PlatformVault.mintNFTIfEligible(tokenURI, user)]
  | burns deposited tokens
  | calls CarbonCreditNFT.mint(... maybe store provenanceRoot)
  v
[CarbonCreditNFT]
  | emits CarbonCreditMinted(tokenId,...)
  v
[Auditor]
  | verifies: burn tx + provenanceRoot + DB reading_hash samples
```

---

## Practical Next Steps

1. Keep your deterministic `reading_hash` exactly as-is and include it in MQTT + DB + chain.
2. Make Carbon mint aggregation default to **one slot per device** (or device+period).
3. Make bridging **lot-agnostic** (single ERC20 amount per bridge tx).
4. Resolve attribution at **NFT mint time**:
   - generate provenance package
   - commit a root/hash into NFT metadata or on-chain
   - retain minute-level evidence in DB keyed by `reading_hash`

---

## Notes for future hardening (optional)
- Add an explicit `policyId` to provenance packages (e.g., FIFO vs proportional).
- Add `periodRange` and `deviceWhitelistRoot` for compliance snapshots.
- Consider storing a compact “proof pointer” (CID / DB id) in NFT metadata.

---

## Repository Structure Suggestion

```
/contracts
  ErgonEnergy.sol
  Carbon.sol
  BridgeEmitter.sol
  BridgeReceiver.sol
  ETHCarbon.sol
  PlatformVault.sol
  CarbonCreditNFT.sol

/device
  contract.py

/backend
  proof-service/
  mqtt-ingest/
  provenance-package/

/frontend
  bridge-app/
  dashboard/
```

---

## License / Disclaimer
Internal engineering & audit documentation for Carbon Bridge / Ergon ecosystem.
