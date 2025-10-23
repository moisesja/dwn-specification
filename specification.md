# 🧠 DWN-X Product Requirements Document (PRD)

### Version

**Draft 0.2**

### Author

Moises Jaramillo (CTO)

### Purpose

DWN-X defines the runtime specification and behaviors for **decentralized personal data nodes** capable of:

- Local-first data ownership
- Sharded and selective CRDT-based synchronization
- Resource-addressable storage
- Queryable and type-enforced persistence
- Delegated governance via ZCAP
- Encrypted data-at-rest and data-in-transit
- Pluggable interfaces (REST, WebSocket, gRPC)
- Extensible infrastructure layer enabling decentralized hosting (DePIN)

---

## 1. Scope

DWN-X is **not designed for full compatibility** with the public DWN specification; it borrows architectural principles but prioritizes:

- Full control by the user (node owner)
- Deterministic data locality and selective sync
- Typed persistence and query support
- ZCAP-governed access and compute delegation
- Deployability across edge, browser, cloud, and embedded hardware environments

---

## 2. Network & Topology

### 2.1 Concepts

| Term                    | Description                                                                                    |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **ShardGroup**          | Logical grouping of DWNs sharing a portion of a user’s data (e.g., Health, Finance, Identity). |
| **Selective Sync**      | Fine-grained control over which shard groups synchronize with others.                          |
| **Localization Policy** | Geo-boundary constraint enforcing where physical data may reside or sync.                      |
| **Peer DWN**            | Any DWN participating in sync or data exchange.                                                |
| **Master DWN**          | The user’s authoritative root node; holds metadata and shard policies.                         |

### 2.2 Example Topology JSON

```json
{
  "masterNode": "did:dwnx:alice:root",
  "shardGroups": [
    {
      "id": "health",
      "location": "us-east-1",
      "syncPolicy": {
        "allowPeers": ["did:dwnx:alice:laptop", "did:dwnx:alice:mobile"],
        "denyRegions": ["non-us"]
      }
    },
    {
      "id": "finance",
      "location": "us-west-2",
      "syncPolicy": {
        "allowPeers": ["did:dwnx:alice:desktop"],
        "denyRegions": []
      }
    }
  ]
}
```

---

## 3. CRDT Synchronization

### 3.1 Goals

- Eventual consistency across all DWNs within a shard group.
- Deterministic conflict resolution via CRDTs (using Automerge or equivalent).
- Support for **partial replication** and **field-level conflict resolution**.

### 3.2 Sync Transport

- **Push-based model** (publish/subscribe via WebSocket or gRPC).
- CRDT deltas are transmitted as JSON-patch style diffs.

### 3.3 Example CRDT Document

```json
{
  "type": "profile",
  "version": "cid:bafy123...",
  "data": {
    "name": "Alice",
    "email": "alice@example.com",
    "metadata": {
      "updatedAt": "2025-10-23T14:00:00Z"
    }
  },
  "crdt": {
    "opId": "op12345",
    "delta": { "email": "alice@newmail.com" },
    "actor": "did:dwnx:alice:mobile"
  }
}
```

---

## 4. Persistence Layer

### 4.1 Storage Classes

| Type          | Description                                   | Example Backend     |
| ------------- | --------------------------------------------- | ------------------- |
| **Standard**  | Default storage for JSON and binary payloads. | S3-compatible       |
| **Ephemeral** | Cache-like, time-bounded data.                | Local disk / memory |
| **Immutable** | Version-locked (e.g., content-addressed).     | IPFS, IPLD          |
| **Encrypted** | Uses key envelope encryption.                 | Any                 |

### 4.2 JSON Schema for Stored Record

```json
{
  "record": {
    "id": "cid:bafyabc...",
    "type": "com.dwnx.record",
    "schema": "https://schemas.dwnx.dev/record/v1",
    "owner": "did:dwnx:alice",
    "dataType": "application/json",
    "storageClass": "encrypted",
    "createdAt": "2025-10-23T00:00:00Z",
    "history": [
      { "version": 1, "cid": "cid:bafy111", "timestamp": "2025-10-20" },
      { "version": 2, "cid": "cid:bafy222", "timestamp": "2025-10-22" }
    ]
  }
}
```

---

## 5. Query Layer

### 5.1 Requirements

- Built-in embedded indexer
- Support for field-level querying (`.data.field=value`)
- Queries expressed in JSON-based DSL

### 5.2 Example Query

```json
{
  "select": ["id", "owner", "data.name"],
  "from": "records",
  "where": {
    "dataType": "application/json",
    "data.category": "health"
  },
  "limit": 50
}
```

### 5.3 Response Example

```json
{
  "results": [
    {
      "id": "cid:bafy123",
      "owner": "did:dwnx:alice",
      "data.name": "Blood Test"
    }
  ]
}
```

---

## 6. Encryption & Data Protection

### 6.1 Threat Model

Commercially hosted DWNs must not access plaintext user data.

### 6.2 Approach

- **Envelope Encryption**:

  - Data encrypted with a symmetric key (AES-256-GCM).
  - Symmetric key encrypted with the owner’s DID public key.

- **Zero-knowledge storage**: Node operator only sees ciphertext.
- **Per-record encryption context** defined as:

```json
{
  "encryption": {
    "algorithm": "AES-256-GCM",
    "encryptedKey": "base64encoded(...)",
    "iv": "base64encoded(...)",
    "authTag": "base64encoded(...)"
  }
}
```

### 6.3 In Transit

- All peer-to-peer sync traffic encrypted (mTLS or DIDComm-like encrypted envelopes).

---

## 7. Authentication & Authorization

### 7.1 Authentication

- Public/private key challenge-response using the DID Document’s verification method.

### 7.2 Authorization

- Implemented **only through ZCAPs**.
- Capabilities define who can:

  - Read/write records
  - Manage shard policies
  - Initiate CRDT sync
  - Host data (for DePIN nodes)

### 7.3 Example ZCAP Document

```json
{
  "id": "zcap:bafy777",
  "controller": "did:dwnx:bob",
  "invoker": "did:dwnx:ai-agent-1",
  "capability": "write:health",
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2025-10-23T15:00:00Z",
    "proofPurpose": "capabilityDelegation",
    "verificationMethod": "did:dwnx:bob#key-1",
    "jws": "..."
  }
}
```

---

## 8. Facade Interfaces

| Interface     | Transport       | Description                          |
| ------------- | --------------- | ------------------------------------ |
| **REST**      | HTTP            | Simplified developer access          |
| **WebSocket** | Bidirectional   | Real-time CRDT delta streaming       |
| **gRPC**      | Binary protocol | Low-latency inter-node communication |
| **CLI SDK**   | Optional        | Local developer interface            |

---

## 9. Governance & Data Control

- Every DWN maintains **local node governance**:

  - Owner-defined ACL and ZCAPs
  - Rule sets defining shard policies and sync regions

- Governance records stored under `/system/governance` namespace.

Example:

```json
{
  "governance": {
    "nodeOwner": "did:dwnx:alice",
    "rules": [
      {
        "scope": "health",
        "allow": ["did:dwnx:alice:mobile"],
        "deny": ["non-us"]
      }
    ]
  }
}
```

---

## 10. DePIN Integration

### 10.1 Objective

Enable **any participant** to host DWN-X storage, compute, or relay nodes and earn rewards.

### 10.2 Components

- **Node Registry Contract** (on-chain): Registers public nodes, their region, and SLA score.
- **Proof of Storage & Availability**: Nodes provide signed attestations periodically.
- **Payment Layer (optional)**: Integration with ERC-20 tokens or DAO-based incentives.

### 10.3 Example Node Metadata

```json
{
  "node": {
    "id": "did:dwnx:provider:xyz",
    "region": "eu-central",
    "capabilities": ["storage", "relay"],
    "uptimeScore": 0.998,
    "stake": "1000.00 ROOT",
    "encryption": "zero-knowledge",
    "proofs": ["attestation:bafy789"]
  }
}
```

---

## 11. Record History & IPLD DAG

Each record version links to the previous one using IPLD content identifiers forming a verifiable DAG.

```json
{
  "recordHistory": {
    "root": "cid:bafy111",
    "versions": [
      { "cid": "cid:bafy222", "prev": "cid:bafy111" },
      { "cid": "cid:bafy333", "prev": "cid:bafy222" }
    ]
  }
}
```

---

## 12. Deferred Topics (Separate Specs)

| Feature                         | Status   | Notes                                                        |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| Edge Compute on Data            | Deferred | Will define model execution permissions and ZCAP integration |
| Federation with other DWN impls | Optional | Not prioritized                                              |
| Data Provenance / Trust         | Optional | Might use VC or DIDComm proofs later                         |
