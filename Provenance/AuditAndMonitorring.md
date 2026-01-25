# Provenance Audit Specification

## Table of contents
- [Overview](#overview)
- [1. Monitored Events](#1-monitored-events)
  - [ErgonEnergy Events](#ergonenergy-events)
  - [Carbon Events](#carbon-events)
  - [BridgeEmitter Events](#bridgeemitter-events)
- [2. Role Assignment and Event Visibility](#2-role-assignment-and-event-visibility)
- [3. Event Dialog Actions and Results](#3-event-dialog-actions-and-results)
- [4. Search and Query Process](#4-search-and-query-process)
- [5. User Experience Summary](#5-user-experience-summary)
- [6. References](#6-references)

---

## Overview
This document describes the monitored events, their arguments and data types, role-based access, and the user experience in the Provenance Explorer UI. It also details the process of querying, searching, and backend communication for audit and provenance information.

---

## 1. Monitored Events

### ErgonEnergy Events

<table>
  <thead>
    <tr>
      <th align="left">Event Name</th>
      <th align="left">Arguments (Type)</th>
      <th align="left">Description</th>
      <th align="left">Role Access</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>PowerOutAudit</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>device</code> (<code>string</code>)<br>
        <code>readingHash</code> (<code>bytes32</code>)<br>
        <code>provenanceHash</code> (<code>bytes32</code>)<br>
        <em>…</em>
      </td>
      <td>Outgoing power reading, hash chain integrity</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>PowerInAudit</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>device</code> (<code>string</code>)<br>
        <code>readingHash</code> (<code>bytes32</code>)<br>
        <code>provenanceHash</code> (<code>bytes32</code>)<br>
        <em>…</em>
      </td>
      <td>Incoming power reading, hash chain integrity</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>DemandExportBreached</code></td>
      <td>
        <code>device</code> (<code>string</code>)<br>
        <code>exportPower</code> (<code>uint256</code>)<br>
        <code>exportLimit</code> (<code>uint256</code>)<br>
        <em>…</em>
      </td>
      <td>Export limit breach event</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>RedeemCarbon</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>device</code> (<code>string</code>)<br>
        <code>amount</code> (<code>uint256</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon token redemption</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>OracleUpdated</code></td>
      <td>
        <code>ergonPrice</code> (<code>uint256</code>)<br>
        <code>carbonPrice</code> (<code>uint256</code>)
      </td>
      <td>Price oracle update</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>DemandResponseUpdated</code></td>
      <td><em>…</em></td>
      <td>Demand response policy update</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>UserAdded</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>New user added</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>UserUpdated</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>User info updated</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>UserGeoUpdated</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>geo</code> (<code>string</code>)<br>
        <em>…</em>
      </td>
      <td>User geolocation updated</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>UserStatusUpdated</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>status</code> (<code>uint8</code>)<br>
        <em>…</em>
      </td>
      <td>User status changed</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>SubmitterApproved</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Submitter approval</td>
      <td><code>auditor</code></td>
    </tr>
  </tbody>
</table>

---

### Carbon Events

<table>
  <thead>
    <tr>
      <th align="left">Event Name</th>
      <th align="left">Arguments (Type)</th>
      <th align="left">Description</th>
      <th align="left">Role Access</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>CarbonMintLotDelta</code></td>
      <td>
        <code>holder</code> (<code>address</code>)<br>
        <code>deltaAmount</code> (<code>uint256</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon lot minted</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonLotTransfer</code></td>
      <td>
        <code>from</code> (<code>address</code>)<br>
        <code>to</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon lot transferred</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonLotBurn</code></td>
      <td>
        <code>from</code> (<code>address</code>)<br>
        <code>operator</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon lot burned</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonLotFrozen</code></td>
      <td>
        <code>holder</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon lot frozen</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonLotUnfrozen</code></td>
      <td>
        <code>holder</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon lot unfrozen</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonMint</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>device</code> (<code>string</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon tokens minted</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>CarbonBurn</code></td>
      <td>
        <code>user</code> (<code>address</code>)<br>
        <code>device</code> (<code>string</code>)<br>
        <em>…</em>
      </td>
      <td>Carbon tokens burned</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>PeriodClosed</code></td>
      <td><em>…</em></td>
      <td>Period closed (anchoring)</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>DevicePeriodCommitted</code></td>
      <td><em>…</em></td>
      <td>Device period committed</td>
      <td><code>auditor</code></td>
    </tr>
    <tr>
      <td><code>PeriodGlobalCommitted</code></td>
      <td><em>…</em></td>
      <td>Global period committed</td>
      <td><code>auditor</code></td>
    </tr>
  </tbody>
</table>

---

### BridgeEmitter Events

<table>
  <thead>
    <tr>
      <th align="left">Event Name</th>
      <th align="left">Arguments (Type)</th>
      <th align="left">Description</th>
      <th align="left">Role Access</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>BridgeInitiatedV2</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Bridge transfer initiated (v2)</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>BridgeInitiatedV3</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <em>…</em>
      </td>
      <td>Bridge transfer initiated (v3)</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>BridgeInitiatedLots</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>lotsHash</code> (<code>bytes32</code>)<br>
        <em>…</em>
      </td>
      <td>Bridge lots initiated</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>TokensLocked</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Tokens locked for bridge</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>TokensUnlocked</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Tokens unlocked</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>TokensBurned</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Tokens burned</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>BridgeFinalized</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Bridge finalized</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
    <tr>
      <td><code>AutoRefundProcessed</code></td>
      <td>
        <code>messageId</code> (<code>bytes32</code>)<br>
        <code>user</code> (<code>address</code>)<br>
        <em>…</em>
      </td>
      <td>Auto refund processed</td>
      <td><code>user</code>, <code>auditor</code></td>
    </tr>
  </tbody>
</table>

---

## 2. Role Assignment and Event Visibility

### Roles
- **user**: Default for wallet holders. Can view only their own device/user events.
- **auditor**: Assigned via configuration (address allowlist). Can view all events, including policy/admin events.
- **admin**: Highest privilege, can view all events and switch roles.
- **public_auditor**: Optional, allows public audit access if enabled.

### Role Assignment
- Role is determined by wallet address and configuration.
- Admin and auditor addresses are set in environment variables.
- Users can switch roles if their address is in the admin/auditor list.

### Event Visibility by Role
- **user**: Can view only user-selectable events (e.g., <code>PowerOutAudit</code>, <code>PowerInAudit</code>, <code>DemandExportBreached</code>, <code>RedeemCarbon</code>, Carbon lot/token events, bridge events) scoped to their address/devices.
- **auditor/admin**: Can view all events, including policy, governance, and user management events (e.g., <code>OracleUpdated</code>, <code>UserGeoUpdated</code>, etc).

---

## 3. Event Dialog Actions and Results

When a user opens an event dialog:
- **Event Details**: All decoded arguments are shown, with copy-to-clipboard shortcuts.
- **Hash Verification**: For <code>PowerOutAudit</code>/<code>PowerInAudit</code>, a button fetches the reading payload and verifies the <code>readingHash</code>.
- **Audit Chain**: For reading events, the audit chain can be traversed to verify hash chain integrity.
- **Export**: Users can export event details as JSON.
- **Status Chips**: Visual indicators show hash match/mismatch and verification status.

---

## 4. Search and Query Process

### Search
- Users can search by transaction hash, event name, device, user, or any argument value.
- Search is case-insensitive and matches across all event fields.

### Event Selection
- Users select event types from a role-filtered list.
- Presets are available for common event groups (core, lots, bridge, policy).

### Querying and Backend Communication
The UI communicates with the backend (and optionally an MQTT broker) as follows:

1. **Provider Setup**: Connects to Ethereum node via WebSocket or RPC (configured in environment).
2. **Event Query**: For selected event types and block range, the UI queries the blockchain for matching events using <code>ethers.js</code>.
3. **Timestamp Resolution**: For each event, the UI fetches the block timestamp for accurate audit trails.
4. **MQTT Integration (optional)**: The UI may subscribe to MQTT topics for real-time event updates or device status.
5. **Audit Chain / Reading Fetch**: For hash chain verification, the UI fetches reading payloads from a REST API (not MQTT) and recalculates hashes.

### Audit Information Gathering
- The process is fully client-driven: the UI queries the blockchain directly for events, then augments with timestamp and (optionally) reading payloads for hash verification.
- All filtering, searching, and analytics are performed client-side for privacy and transparency.

---

## 5. User Experience Summary
- **Event Table**: Paginated, sortable, filterable by event type, time, and search.
- **Event Dialog**: Rich details, hash verification, audit chain, and export.
- **Role Awareness**: UI adapts to user role, showing only permitted events and actions.
- **Audit Dashboard**: Analytics and integrity checks for loaded events.
- **Export**: CSV and PDF export of filtered events.

---

## 6. References
- See <code>ProvenanceHome.js</code> for implementation details.
- See contract ABIs for full event argument lists and types.
- Environment variables control provider URLs, role allowlists, and feature toggles.

---

*Document generated: 2026-01-26*
