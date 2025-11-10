Perfect! Now I have all the pieces. Let me create a comprehensive, maintainable specification document.

---

# **Decentralized Storage Specification**
## **Version 0.1.0-draft**

**Specification Status:** Draft  
**Latest Draft:** This document  
**Previous Draft:** None  
**Editors:**
- Moises E. Jaramillo

**Participate:**
- GitHub repo: [Specification](https://github.com/moisesja/dwn-specification/blob/main/specification.md)
- File a bug: [TBD]
- Commit history: [TBD]

**Document Conventions:**
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC2119].

---

## **Abstract**

The Hybrid Decentralized Storage (HDS) specification defines a privacy-preserving, decentralized personal data storage system that combines the message-based architecture of Decentralized Web Nodes (DWN) with transport-agnostic design, ZCAP-LD authorization, and advanced synchronization capabilities. The specification enables users to control their data through DID-based identity, distribute storage across multiple nodes for resilience and compliance, and share data through cryptographically-enforced capabilities.

---

## **Status of This Document**

This is a DRAFT specification under active development. It incorporates requirements and design decisions from the Decentralized Web Node specification, Solid Protocol, and W3C standards including DIDs, VCs, and ZCAP-LD. 

**This specification is subject to change.** Implementers should track updates through the version history and participate in the development process.

---

## **Table of Contents**

1. [Introduction](#1-introduction)
2. [Terminology](#2-terminology)
3. [Architecture Overview](#3-architecture-overview)
4. [Identity & Discovery](#4-identity--discovery)
5. [Data Model](#5-data-model)
6. [Protocol System](#6-protocol-system)
7. [Authorization Framework](#7-authorization-framework)
8. [Encryption Framework](#8-encryption-framework)
9. [Synchronization & Replication](#9-synchronization--replication)
10. [Query System](#10-query-system)
11. [Notification System](#11-notification-system)
12. [Transport Layer](#12-transport-layer)
13. [Error Handling](#13-error-handling)
14. [Security Considerations](#14-security-considerations)
15. [Privacy Considerations](#15-privacy-considerations)
16. [Implementation Guidance](#16-implementation-guidance)
17. [References](#17-references)
18. [Appendices](#18-appendices)

---

## **1. Introduction**

### **1.1 Motivation**

Centralized data storage platforms create several critical problems:
- **Loss of Control**: Users cannot control who accesses their data
- **Vendor Lock-in**: Data is trapped in proprietary systems
- **Privacy Risks**: Centralized databases are high-value targets
- **Censorship**: Platforms can deny access arbitrarily
- **Data Loss**: Single points of failure risk data loss

This specification addresses these issues through:
- **Self-Sovereign Identity**: DID-based ownership and control
- **Decentralized Architecture**: Multi-node mesh with no single point of failure
- **Cryptographic Authorization**: Capability-based access control
- **End-to-End Encryption**: Data encrypted at rest and in transit
- **Protocol-Driven Interoperability**: Shared data structures across applications

### **1.2 Design Principles**

1. **DID-First**: Identity and discovery rooted in decentralized identifiers
2. **Transport Agnostic**: Support HTTP, gRPC, DIDComm, WebSocket equally
3. **Capability-Based Security**: Pure ZCAP-LD authorization without origin restrictions
4. **Privacy by Default**: Encryption and selective disclosure built-in
5. **Offline-First**: Eventual consistency with CRDT conflict resolution
6. **Compliance-Ready**: Selective sync for geographic/regulatory requirements
7. **User Sovereignty**: Users control data location, sharing, and deletion

### **1.3 Scope**

**In Scope:**
- DID-based identity and discovery
- Message-based data operations
- Protocol-driven type system
- ZCAP-LD authorization
- Multi-transport support
- Mesh synchronization with CRDTs
- End-to-end encryption
- Searchable encrypted indexes
- Social key recovery

**Out of Scope:**
- DID method specifications (use existing standards)
- Application-level business logic
- User interface requirements
- Specific database implementations
- Payment/billing systems

### **1.4 Relationship to Other Specifications**

This specification builds on:
- **W3C DID Core**: For identity foundation
- **W3C Verifiable Credentials**: For attestations
- **W3C ZCAP-LD**: For authorization capabilities
- **DIF Decentralized Web Node**: For message architecture
- **IPLD**: For content addressing
- **CRDT**: For conflict-free replication

---

## **2. Terminology**

### **2.1 Core Concepts**

**Decentralized Web Node (DWN)**
- A personal data storage node implementing this specification

**Tenant**
- The DID controller who owns and operates one or more DWNs

**Message**
- An atomic operation request with descriptor, data, and authorization

**Record**
- A versioned, content-addressed data object stored in a DWN

**Protocol**
- A declarative schema defining types, structures, and rules for data

**Capability**
- A ZCAP-LD credential granting specific permissions to an invoker

**Context**
- A logical grouping of related records across types

**Shard**
- A subset of data stored on specific nodes for compliance/performance

### **2.2 Key Identifiers**

**DID (Decentralized Identifier)**
- A globally unique identifier for entities (e.g., `did:example:alice`)

**Record ID**
- Content-addressed identifier (CID) of a record's initial entry

**Context ID**
- Deterministic identifier grouping related records

**Capability ID**
- Unique identifier for a ZCAP-LD capability (e.g., `urn:uuid:...`)

### **2.3 Operations**

**Create**
- Insert a new record

**Read**
- Retrieve an existing record

**Update**
- Modify an existing record (creates new version)

**Delete**
- Mark a record as deleted (tombstone or prune)

**Query**
- Search for records matching criteria

**Subscribe**
- Receive notifications on record changes

---

## **3. Architecture Overview**

### **3.1 System Components**

```
┌─────────────────────────────────────────────────────────────┐
│                         TENANT (DID)                         │
│                    did:example:alice                         │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
       ┌───────▼────────┐        ┌───────▼────────┐
       │   DWN Node 1   │◄──────►│   DWN Node 2   │
       │  (US-East)     │  Sync  │   (EU-West)    │
       └───────┬────────┘        └───────┬────────┘
               │                          │
       ┌───────▼──────────────────────────▼────────┐
       │          Transport Abstraction            │
       ├───────────────────────────────────────────┤
       │  HTTP  │  gRPC  │  DIDComm  │  WebSocket │
       └───────────────────────────────────────────┘
               │                          │
       ┌───────▼────────┐        ┌───────▼────────┐
       │  Application   │        │  Application   │
       │   Client A     │        │   Client B     │
       └────────────────┘        └────────────────┘
```

### **3.2 Data Flow**

```
Client Request
    ↓
Transport Layer (HTTP/gRPC/DIDComm/WebSocket)
    ↓
Message Parser & Validator
    ↓
Authorization Engine (ZCAP-LD Verification)
    ↓
Protocol Rule Enforcement
    ↓
Encryption/Decryption Layer
    ↓
Storage Engine (IPLD/DAG)
    ↓
Index Management (Blind Indexes)
    ↓
Sync Engine (CRDT Replication)
    ↓
Response Formatter
    ↓
Transport Layer
    ↓
Client Response
```

### **3.3 Layered Architecture**

**Layer 1: Identity & Discovery**
- DID resolution
- Service endpoint discovery
- Key management

**Layer 2: Transport Abstraction**
- HTTP REST
- gRPC streaming
- DIDComm messaging
- WebSocket connections

**Layer 3: Message Processing**
- Message parsing
- Descriptor validation
- Authorization verification

**Layer 4: Authorization**
- ZCAP-LD capability verification
- Protocol rule enforcement
- Delegation chain validation

**Layer 5: Encryption**
- Data encryption/decryption
- Key derivation
- Blind index generation

**Layer 6: Storage**
- IPLD content addressing
- Record versioning
- CRDT state management

**Layer 7: Synchronization**
- Multi-node replication
- Conflict resolution
- Selective sync

**Layer 8: Indexing & Query**
- Blind index maintenance
- Query execution
- Result filtering

---

## **4. Identity & Discovery**

### **4.1 DID Integration**

#### **4.1.1 DID Document Requirements**

A DWN owner's DID Document MUST include a service endpoint of type `DecentralizedWebNode`:

```json
{
  "id": "did:example:alice",
  "verificationMethod": [{
    "id": "did:example:alice#key-1",
    "type": "JsonWebKey2020",
    "controller": "did:example:alice",
    "publicKeyJwk": {
      "kty": "OKP",
      "crv": "Ed25519",
      "x": "..."
    }
  }],
  "keyAgreement": [{
    "id": "did:example:alice#key-agreement-1",
    "type": "X25519KeyAgreementKey2020",
    "controller": "did:example:alice",
    "publicKeyJwk": {
      "kty": "OKP",
      "crv": "X25519",
      "x": "..."
    }
  }],
  "service": [{
    "id": "#dwn",
    "type": "DecentralizedWebNode",
    "serviceEndpoint": [
      "https://dwn.alice.example",
      "grpc://dwn.alice.example:50051",
      "didcomm://dwn.alice.example/inbox",
      "wss://dwn.alice.example/ws"
    ],
    "enc": ["#key-agreement-1", "#key-agreement-2"],
    "sig": ["#key-1"]
  }]
}
```

**REQUIRED Properties:**
- `id`: MUST be "#dwn"
- `type`: MUST be "DecentralizedWebNode"
- `serviceEndpoint`: MUST be an array of one or more endpoint URIs
- `enc`: MUST reference one or more key agreement keys for encryption
- `sig`: MUST reference one or more verification methods for signing

#### **4.1.2 DID Method Support**

Implementations MUST support DID resolution for:
- Any DID method that conforms to W3C DID Core specification
- Implementations MAY optimize for specific DID methods

Implementations MUST NOT restrict which DID methods can be used.

#### **4.1.3 Service Endpoint Resolution**

When resolving a DID to locate DWN endpoints:

1. Resolve the DID to obtain the DID Document
2. Locate the service entry with `id: "#dwn"`
3. Extract the `serviceEndpoint` array
4. Select an endpoint based on:
   - Transport capability of the client
   - Geographic proximity (if applicable)
   - Load balancing considerations

**Resolution Limit:**
- If a `serviceEndpoint` value is itself a DID, implementations MUST NOT follow more than ONE level of indirection
- This prevents infinite loops and complexity

**Example:**
```json
// Alice's DID Document
{
  "id": "did:example:alice",
  "service": [{
    "id": "#dwn",
    "type": "DecentralizedWebNode",
    "serviceEndpoint": ["did:example:hosting-provider"]
  }]
}

// Hosting Provider's DID Document (1 hop allowed)
{
  "id": "did:example:hosting-provider",
  "service": [{
    "id": "#dwn",
    "type": "DecentralizedWebNode",
    "serviceEndpoint": ["https://dwn.provider.example"]
  }]
}

// Further indirection NOT allowed
```

### **4.2 Multi-Node Discovery**

A tenant MAY operate multiple DWN nodes. All nodes MUST be advertised in the `serviceEndpoint` array:

```json
{
  "id": "#dwn",
  "type": "DecentralizedWebNode",
  "serviceEndpoint": [
    "https://dwn-us-east.alice.example",
    "https://dwn-eu-west.alice.example",
    "https://dwn-asia-pacific.alice.example"
  ],
  "sharding": {
    "healthcare": ["https://dwn-us-east.alice.example"],
    "financial": ["https://dwn-eu-west.alice.example"],
    "general": ["*"]
  }
}
```

**Sharding Metadata:**
- The `sharding` property is OPTIONAL
- It indicates which nodes contain which protocol data
- Wildcard `"*"` means "all nodes"
- Clients MAY use this to optimize queries

### **4.3 Key Discovery**

Clients MUST obtain encryption and signing keys from the DID Document:

**For Encryption:**
```javascript
const encKeyRefs = serviceEndpoint.enc;
// Resolve to keyAgreement entries
const encKeys = encKeyRefs.map(ref => 
  didDoc.keyAgreement.find(k => k.id === ref)
);
```

**For Signature Verification:**
```javascript
const sigKeyRefs = serviceEndpoint.sig;
// Resolve to verificationMethod entries
const sigKeys = sigKeyRefs.map(ref => 
  didDoc.verificationMethod.find(k => k.id === ref)
);
```

### **4.4 DID Dereferencing**

Implementations MUST support DID URL dereferencing per DID Core:

```
did:example:alice#dwn
└─┬─┘ └──────┬─────┘ └┬┘
  │          │         └─ Fragment (service ID)
  │          └─────────── DID Method Specific Identifier
  └────────────────────── DID Method

Dereferences to the service endpoint object
```

---

## **5. Data Model**

### **5.1 Message Structure**

All operations in HDS are expressed as messages with the following structure:

```typescript
interface Message {
  recordId: string;              // CID of initial entry
  contextId?: string;            // Optional context grouping
  descriptor: MessageDescriptor;
  data?: string;                 // Base64url encoded data
  encryption?: EncryptionMetadata;
  attestation?: GeneralJWS;     // Optional attestation
  authorization: GeneralJWS;     // Required authorization
}

interface MessageDescriptor {
  interface: string;             // "Records" | "Protocols" | "Sync"
  method: string;                // "Write" | "Query" | "Delete" etc.
  messageTimestamp: string;      // ISO 8601 timestamp
  dataFormat?: string;           // MIME type
  dataCid?: string;              // CID of data
  schema?: string;               // Schema URI
  protocol?: string;             // Protocol URI
  protocolVersion?: string;      // SemVer string
  parentId?: string;             // CID of parent record
  dateCreated?: string;          // ISO 8601 timestamp
  datePublished?: string;        // ISO 8601 timestamp
  published?: boolean;           // Publication state
}
```

### **5.2 Record Lifecycle**

#### **5.2.1 Record Creation**

```json
{
  "recordId": "bafyreiabc123...",
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "protocol": "https://example.com/social",
    "protocolVersion": "1.0.0",
    "schema": "https://schema.org/SocialMediaPosting",
    "dataFormat": "application/json",
    "dataCid": "bafyrei...",
    "dateCreated": "2025-10-24T15:30:00Z",
    "published": false
  },
  "data": "base64url_encoded_content",
  "authorization": {
    "payload": {
      "descriptorCid": "bafyrei...",
      "capability": "urn:uuid:cap-123"
    },
    "signatures": [{
      "protected": "eyJ...",
      "signature": "abc..."
    }]
  }
}
```

#### **5.2.2 Record Versioning**

Updates create a new version linked via `parentId`:

```json
{
  "recordId": "bafyreiabc123...",  // Same as original
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "parentId": "bafyrei_previous_descriptor_cid...",
    "dateCreated": "2025-10-24T16:45:00Z"
    // Other fields...
  }
}
```

**Versioning Rules:**
- `recordId` remains constant across versions
- `parentId` MUST reference the previous version's descriptor CID
- `dateCreated` MUST be later than parent's `dateCreated`
- Immutable fields from initial entry MUST NOT change

#### **5.2.3 Record Deletion**

```json
{
  "recordId": "bafyreiabc123...",
  "descriptor": {
    "interface": "Records",
    "method": "Delete",
    "messageTimestamp": "2025-10-24T17:00:00Z",
    "prune": false  // false = tombstone, true = purge descendants
  },
  "authorization": { /* ZCAP */ }
}
```

**Deletion Modes:**
- `prune: false` - Tombstone (mark deleted, preserve metadata)
- `prune: true` - Purge (remove record and all descendants)

### **5.3 Content Addressing**

#### **5.3.1 Record ID Generation**

The `recordId` is generated using IPLD CID:

```typescript
function generateRecordId(descriptor: MessageDescriptor): string {
  // 1. Create generation object
  const genObject = {
    descriptorCid: generateCID(descriptor, 'dag-cbor')
  };
  
  // 2. Encode as DAG-CBOR
  const encoded = encode(genObject, 'dag-cbor');
  
  // 3. Generate CID v1
  return CID.create(1, 0x71, hash(encoded, 'sha2-256'));
}
```

#### **5.3.2 Data CID Generation**

The `dataCid` is generated from the raw data:

```typescript
function generateDataCid(data: Uint8Array): string {
  // Encode as DAG-PB
  return CID.create(1, 0x70, hash(data, 'sha2-256'));
}
```

### **5.4 Context Grouping**

Records can be logically grouped using `contextId`:

```json
{
  "contextId": "thread-2025-10-24-abc",
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "protocol": "https://example.com/social"
  }
}
```

**Context ID Generation:**
- MAY be deterministic (hash of root record)
- MAY be random (UUID)
- MUST be consistent within a logical grouping
- Protocol definitions MAY specify context generation rules

**Use Cases:**
- Discussion threads
- Document versions
- Collaboration sessions
- Shared project data

---

## **6. Protocol System**

### **6.1 Protocol Definitions**

Protocols declaratively define data structures, relationships, and rules.

#### **6.1.1 Protocol Structure**

```json
{
  "protocol": "https://example.com/social",
  "version": "1.0.0",
  "extends": null,
  "published": true,
  "types": {
    "post": {
      "schema": "https://schema.org/SocialMediaPosting",
      "dataFormats": ["application/json", "application/ld+json"]
    },
    "comment": {
      "schema": "https://example.com/schemas/comment",
      "dataFormats": ["application/json"]
    },
    "reaction": {
      "schema": "https://example.com/schemas/reaction",
      "dataFormats": ["application/json"]
    }
  },
  "structure": {
    "post": {
      "$size": {
        "max": 1000000
      },
      "$actions": [{
        "who": "anyone",
        "can": ["read"]
      }, {
        "who": "author",
        "of": "post",
        "can": ["update", "delete"]
      }],
      "$encryption": {
        "rootKeyId": "did:example:alice#key-agreement-1",
        "derivationScheme": "protocolPath"
      },
      "comment": {
        "$size": { "max": 100000 },
        "$actions": [{
          "who": "anyone",
          "can": ["create", "read"]
        }, {
          "who": "author",
          "of": "comment",
          "can": ["update", "delete"]
        }],
        "reaction": {
          "$size": { "max": 1000 },
          "$actions": [{
            "who": "anyone",
            "can": ["create", "read"]
          }]
        }
      }
    }
  },
  "conflictResolution": {
    "strategy": "crdt",
    "crdtType": "lww-element-set",
    "fallback": "manual"
  }
}
```

#### **6.1.2 Protocol Properties**

**Required Fields:**
- `protocol`: URI identifying the protocol
- `version`: SemVer version string
- `published`: Boolean indicating public visibility
- `types`: Object defining record types
- `structure`: Hierarchical rules and relationships

**Optional Fields:**
- `extends`: URI of parent protocol (for inheritance)
- `conflictResolution`: CRDT strategy specification
- `indexes`: Index definitions (see Query System)

### **6.2 Protocol Inheritance**

Protocols can extend other protocols:

```json
{
  "protocol": "https://example.com/social-premium",
  "version": "1.0.0",
  "extends": "https://example.com/social/1.0.0",
  "types": {
    "post": {
      "schema": "https://example.com/schemas/premium-post",
      "dataFormats": ["application/json"]
    },
    "poll": {
      "schema": "https://example.com/schemas/poll",
      "dataFormats": ["application/json"]
    }
  },
  "structure": {
    "post": {
      // Inherits from parent, adds/overrides
      "$size": { "max": 5000000 },
      "poll": {
        "$actions": [{
          "who": "author",
          "of": "post",
          "can": ["create", "update", "delete"]
        }]
      }
    }
  }
}
```

**Inheritance Rules:**
- Child protocol MUST increment major version if breaking parent compatibility
- Child MAY override type definitions
- Child MAY add new types
- Child's structure rules MUST NOT weaken parent's security constraints

### **6.3 Protocol Installation**

Protocols are installed via `ProtocolsConfigure` messages:

```json
{
  "descriptor": {
    "interface": "Protocols",
    "method": "Configure",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "protocol": "https://example.com/social",
    "protocolVersion": "1.0.0",
    "definition": { /* Full protocol definition */ }
  },
  "authorization": { /* Owner capability */ }
}
```

**Configuration Rules:**
- Only the DWN owner (tenant) can install protocols
- Installing a protocol enables record creation under that protocol
- Protocols can be updated by installing a new version
- Old protocol versions remain accessible (version negotiation)

### **6.4 Protocol Querying**

Discover installed protocols:

```json
{
  "descriptor": {
    "interface": "Protocols",
    "method": "Query",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "filter": {
      "protocol": "https://example.com/social",
      "versions": ["1.x.x", "2.x.x"]
    }
  }
}
```

**Response:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "protocols": [{
    "protocol": "https://example.com/social",
    "version": "1.5.0",
    "installedAt": "2025-01-15T10:00:00Z",
    "definition": { /* Protocol definition */ }
  }, {
    "protocol": "https://example.com/social",
    "version": "2.0.0",
    "installedAt": "2025-08-20T14:30:00Z",
    "definition": { /* Protocol definition */ }
  }]
}
```

---

## **7. Authorization Framework**

### **7.1 ZCAP-LD Integration**

This specification uses W3C ZCAP-LD (Authorization Capabilities for Linked Data) for authorization.

#### **7.1.1 Capability Structure**

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/security/suites/jws-2020/v1",
    "https://w3id.org/zcap/v1"
  ],
  "id": "urn:uuid:4d44084c-334e-46dc-ac23-5e26f43adb5c",
  "type": ["VerifiableCredential", "Authorization"],
  "issuer": "did:example:alice",
  "issuanceDate": "2025-10-24T15:30:00Z",
  "expirationDate": "2026-10-24T15:30:00Z",
  "credentialSubject": {
    "id": "did:example:bob",
    "parentCapability": "urn:uuid:root-capability",
    "invocationTarget": {
      "id": "did:example:alice",
      "type": "DWNNode",
      "allowedActions": ["read", "write"],
      "constraints": {
        "protocol": "https://example.com/social",
        "schema": "https://schema.org/SocialMediaPosting",
        "contextId": "thread-abc-123"
      }
    }
  },
  "proof": {
    "type": "JsonWebSignature2020",
    "created": "2025-10-24T15:30:00Z",
    "verificationMethod": "did:example:alice#key-1",
    "proofPurpose": "capabilityDelegation",
    "jws": "eyJ..."
  }
}
```

#### **7.1.2 Capability Properties**

**Required:**
- `id`: Unique capability identifier
- `type`: MUST include "Authorization"
- `issuer`: DID of capability issuer
- `credentialSubject.id`: DID of invoker (recipient)
- `credentialSubject.invocationTarget`: Target resource/node
- `credentialSubject.invocationTarget.allowedActions`: Permitted operations
- `proof`: Cryptographic proof

**Optional:**
- `expirationDate`: Capability expiration
- `credentialSubject.parentCapability`: For delegation chains
- `credentialSubject.invocationTarget.constraints`: Restrictions

#### **7.1.3 Allowed Actions**

Standard actions aligned with DWN operations:
- `create`: Create new records
- `read`: Read existing records
- `update`: Modify records
- `delete`: Delete records
- `query`: Search for records
- `subscribe`: Receive notifications
- `configure`: Install/update protocols (admin)

### **7.2 Capability Invocation**

To invoke a capability, include it in the message authorization:

```json
{
  "recordId": "bafyrei...",
  "descriptor": { /* ... */ },
  "authorization": {
    "payload": {
      "descriptorCid": "bafyrei...",
      "capability": "urn:uuid:4d44084c-334e-46dc-ac23-5e26f43adb5c",
      "capabilityChain": [
        { /* Root capability */ },
        { /* Delegated capability */ }
      ]
    },
    "signatures": [{
      "protected": {
        "alg": "EdDSA",
        "kid": "did:example:bob#key-1"
      },
      "signature": "..."
    }]
  }
}
```

### **7.3 Capability Delegation**

Capabilities can be delegated with attenuation:

```json
{
  "@context": ["..."],
  "id": "urn:uuid:delegated-cap-xyz",
  "type": ["VerifiableCredential", "Authorization"],
  "issuer": "did:example:bob",
  "credentialSubject": {
    "id": "did:example:carol",
    "parentCapability": "urn:uuid:4d44084c-334e-46dc-ac23-5e26f43adb5c",
    "invocationTarget": {
      "id": "did:example:alice",
      "type": "DWNNode",
      "allowedActions": ["read"],  // Attenuated: only read, not write
      "constraints": {
        "protocol": "https://example.com/social",
        "schema": "https://schema.org/SocialMediaPosting",
        "contextId": "thread-abc-123",
        "recordIds": ["bafyrei_specific_record"]  // Further restricted
      }
    }
  },
  "proof": { /* Signed by Bob */ }
}
```

**Delegation Rules:**
- Delegated capability MUST NOT grant more permissions than parent
- Delegated capability MAY add additional constraints
- Delegation chains MUST be verified recursively to root authority

### **7.4 Root Capabilities**

Root capabilities are self-issued by the DWN owner:

```json
{
  "@context": ["..."],
  "id": "urn:uuid:root-cap-alice",
  "type": ["VerifiableCredential", "Authorization"],
  "issuer": "did:example:alice",
  "credentialSubject": {
    "id": "did:example:alice",  // Self-issued
    "invocationTarget": {
      "id": "did:example:alice",
      "type": "DWNNode",
      "allowedActions": ["*"]  // Full control
    }
  },
  "proof": { /* Signed by Alice */ }
}
```

### **7.5 Protocol-Level Authorization**

Protocol definitions specify baseline permissions:

```json
{
  "protocol": "https://example.com/social",
  "structure": {
    "post": {
      "$actions": [{
        "who": "anyone",
        "can": ["read"]
      }, {
        "who": "author",
        "of": "post",
        "can": ["update", "delete"]
      }]
    }
  }
}
```

**Authorization Evaluation:**
1. Check if protocol rules allow the operation
2. If protocol allows, check for ZCAP
3. Verify ZCAP is valid and grants permission
4. Grant access only if both pass

**Protocol Actions:**
- `anyone`: No authentication required
- `author`: Must be the record creator
- `recipient`: Must be specified recipient
- `role`: Must hold a role capability

### **7.6 Granular Capabilities**

Capabilities can be highly granular:

**Per-Record:**
```json
{
  "invocationTarget": {
    "recordIds": ["bafyrei_specific_record_1", "bafyrei_specific_record_2"]
  }
}
```

**Per-Schema:**
```json
{
  "invocationTarget": {
    "schemas": [
      "https://schema.org/SocialMediaPosting",
      "https://schema.org/Comment"
    ]
  }
}
```

**Per-Context:**
```json
{
  "invocationTarget": {
    "contextIds": ["thread-abc-123", "project-xyz-456"]
  }
}
```

**Time-Bound:**
```json
{
  "expirationDate": "2025-12-31T23:59:59Z"
}
```

### **7.7 Capability Management Protocol**

Using the DWN Permissions protocol (`https://tbd.website/dwn/permissions`):

**Request Capability:**
```json
{
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "protocol": "https://tbd.website/dwn/permissions",
    "schema": "https://tbd.website/dwn/permissions/request",
    "dataFormat": "application/json"
  },
  "data": {
    "requester": "did:example:bob",
    "requestedPermissions": {
      "allowedActions": ["read", "write"],
      "protocol": "https://example.com/social"
    }
  }
}
```

**Grant Capability:**
```json
{
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "protocol": "https://tbd.website/dwn/permissions",
    "schema": "https://tbd.website/dwn/permissions/grant",
    "dataFormat": "application/json"
  },
  "data": {
    "grantee": "did:example:bob",
    "capability": { /* ZCAP-LD credential */ }
  }
}
```

**Revoke Capability:**
```json
{
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "protocol": "https://tbd.website/dwn/permissions",
    "schema": "https://tbd.website/dwn/permissions/revocation",
    "dataFormat": "application/json"
  },
  "data": {
    "capabilityId": "urn:uuid:4d44084c-334e-46dc-ac23-5e26f43adb5c",
    "reason": "Access no longer required"
  }
}
```

---

## **8. Encryption Framework**

### **8.1 Overview**

The encryption framework provides:
- Message-level encryption for all records
- Hierarchical key derivation (protocol-path and context-based)
- Multi-recipient support
- Searchable encryption via blind indexes
- Social key recovery (M-of-N)

### **8.2 Encryption Algorithms**

**Supported Schemes:**

| Layer | Algorithm | Usage |
|-------|-----------|-------|
| Data Encryption | AES-256-GCM | Symmetric encryption of record data |
| Key Agreement | X25519 (ECDH) | Asymmetric key exchange |
| Key Wrapping | ECDH-ES+A256KW | Encrypt symmetric keys for recipients |
| Key Derivation | HKDF-SHA256 | Derive keys from root key |
| Blind Indexing | HMAC-SHA256 | Generate searchable indexes |
| Social Recovery | Shamir Secret Sharing | Split/combine recovery shards |

### **8.3 Key Hierarchy**

```
Root Encryption Key (in DID keyAgreement)
    │
    ├─► Protocol Path Derived Key
    │       │
    │       ├─► Protocol Key (e.g., "social-protocol")
    │       │       │
    │       │       ├─► Type Key (e.g., "post")
    │       │       │       │
    │       │       │       └─► SubType Key (e.g., "comment")
    │       │       │
    │       │       └─► Type Key (e.g., "reaction")
    │       │
    │       └─► Protocol Key (e.g., "medical-protocol")
    │
    └─► Context Derived Key
            │
            ├─► Context Key (e.g., "project-abc")
            │
            └─► Context Key (e.g., "thread-xyz")
```

### **8.4 Key Material Management**

#### **8.4.1 Root Key Structure**

```json
{
  "did": "did:example:alice",
  "keyMaterial": {
    "rootEncryptionKey": {
      "id": "did:example:alice#key-agreement-1",
      "type": "X25519KeyAgreementKey2020",
      "privateKeyJwk": {
        "kty": "OKP",
        "crv": "X25519",
        "x": "base64url_public",
        "d": "base64url_private"
      },
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "X25519",
        "x": "base64url_public"
      },
      "version": 1,
      "createdAt": "2025-01-15T10:00:00Z",
      "status": "active",
      "rotationSchedule": "P365D"
    },
    "searchKey": {
      "id": "did:example:alice#search-key-1",
      "type": "HmacKey",
      "keyBytes": "base64url_encoded_256bit_key",
      "version": 1,
      "status": "active"
    }
  }
}
```

### **8.5 Protocol Path Key Derivation**

```typescript
interface DerivedKey {
  derivationPath: string[];
  derivedPublicKey: JsonWebKey;
  derivedPrivateKey?: JsonWebKey;
  rootKeyId: string;
  algorithm: "ECDH-ES+A256KW";
}

function deriveProtocolPathKey(
  rootPrivateKey: Uint8Array,
  path: string[]
): DerivedKey {
  // Example path: ["protocolPath", "https://example.com/social", "post", "comment"]
  
  // Step 1: Concatenate path components
  const pathString = path.join('/');
  const info = new TextEncoder().encode(pathString);
  
  // Step 2: HKDF extraction
  const salt = new Uint8Array(32); // All zeros for determinism
  const prk = hkdfExtract(salt, rootPrivateKey, 'SHA-256');
  
  // Step 3: HKDF expansion
  const derivedKeyBytes = hkdfExpand(prk, info, 32, 'SHA-256');
  
  // Step 4: Convert to X25519 key pair
  const derivedPrivateKey = derivedKeyBytes;
  const derivedPublicKey = x25519.getPublicKey(derivedPrivateKey);
  
  return {
    derivationPath: path,
    derivedPrivateKey: bytesToJwk(derivedPrivateKey),
    derivedPublicKey: bytesToJwk(derivedPublicKey),
    rootKeyId: "did:example:alice#key-agreement-1",
    algorithm: "ECDH-ES+A256KW"
  };
}
```

**Protocol Path Examples:**
```typescript
// Healthcare record
["protocolPath", "https://health.example/medical", "healthRecord", "labTest"]

// Social media post
["protocolPath", "https://example.com/social", "post"]

// Comment on a post
["protocolPath", "https://example.com/social", "post", "comment"]
```

### **8.6 Context-Based Key Derivation**

```typescript
function deriveContextKey(
  rootPrivateKey: Uint8Array,
  contextId: string
): DerivedKey {
  const path = ["protocolContext", contextId];
  const pathString = path.join('/');
  const info = new TextEncoder().encode(pathString);
  
  const salt = new Uint8Array(32);
  const prk = hkdfExtract(salt, rootPrivateKey, 'SHA-256');
  const derivedKeyBytes = hkdfExpand(prk, info, 32, 'SHA-256');
  
  const derivedPrivateKey = derivedKeyBytes;
  const derivedPublicKey = x25519.getPublicKey(derivedPrivateKey);
  
  return {
    derivationPath: path,
    derivedPrivateKey: bytesToJwk(derivedPrivateKey),
    derivedPublicKey: bytesToJwk(derivedPublicKey),
    rootKeyId: "did:example:alice#key-agreement-1",
    algorithm: "ECDH-ES+A256KW"
  };
}
```

**Context Examples:**
```typescript
// Collaboration project
["protocolContext", "project-2025-q4-launch"]

// Group chat
["protocolContext", "team-standup-thread"]

// Shared medical records for a hospital visit
["protocolContext", "hospital-visit-2025-10-24"]
```

### **8.7 Encrypted Record Structure**

```json
{
  "recordId": "bafyrei...",
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "protocol": "https://health.example/medical",
    "schema": "https://health.example/schemas/labTest",
    "dataFormat": "application/json",
    "dataCid": "bafyrei_encrypted_data_cid",
    "dateCreated": "2025-10-24T15:30:00Z",
    "encryption": {
      "algorithm": "ECIES-ES256K+AES-256-GCM",
      "initializationVector": "base64url_12byte_iv",
      "keyEncryption": [{
        "derivationScheme": "protocolPath",
        "rootKeyId": "did:example:alice#key-agreement-1",
        "algorithm": "ECDH-ES+A256KW",
        "ephemeralPublicKey": {
          "kty": "OKP",
          "crv": "X25519",
          "x": "base64url_ephemeral_public"
        },
        "encryptedKey": "base64url_wrapped_symmetric_key",
        "derivedPublicKey": {
          "kty": "OKP",
          "crv": "X25519",
          "x": "base64url_derived_public"
        }
      }, {
        "derivationScheme": "direct",
        "recipientKeyId": "did:example:doctor#key-agreement-1",
        "algorithm": "ECDH-ES+A256KW",
        "ephemeralPublicKey": { /* ... */ },
        "encryptedKey": "base64url_wrapped_for_doctor"
      }]
    }
  },
  "encryptedData": "base64url_aes_gcm_ciphertext_with_auth_tag"
}
```

### **8.8 Encryption Process**

```typescript
async function encryptRecord(
  data: Uint8Array,
  protocol: string,
  recordType: string,
  recipients: string[],  // DIDs of recipients
  rootKey: PrivateKey
): Promise<EncryptedRecord> {
  
  // Step 1: Generate random symmetric key
  const symmetricKey = crypto.getRandomValues(new Uint8Array(32));
  const iv = crypto.getRandomValues(new Uint8Array(12));
  
  // Step 2: Encrypt data with AES-256-GCM
  const encrypted = await aesGcmEncrypt(data, symmetricKey, iv);
  
  // Step 3: Derive protocol-path key
  const path = ["protocolPath", protocol, recordType];
  const derivedKey = deriveProtocolPathKey(rootKey, path);
  
  // Step 4: Wrap symmetric key for protocol-path (owner access)
  const ownerWrapped = await ecdhWrapKey(
    symmetricKey,
    derivedKey.derivedPublicKey
  );
  
  // Step 5: Wrap symmetric key for each recipient
  const recipientWrappedKeys = await Promise.all(
    recipients.map(async (recipientDid) => {
      const recipientPublicKey = await resolveKeyAgreementKey(recipientDid);
      return await ecdhWrapKey(symmetricKey, recipientPublicKey);
    })
  );
  
  return {
    encryptedData: base64url(encrypted),
    encryption: {
      algorithm: "ECIES-ES256K+AES-256-GCM",
      initializationVector: base64url(iv),
      keyEncryption: [
        {
          derivationScheme: "protocolPath",
          rootKeyId: rootKey.id,
          ...ownerWrapped
        },
        ...recipientWrappedKeys.map((wrapped, i) => ({
          derivationScheme: "direct",
          recipientKeyId: recipients[i],
          ...wrapped
        }))
      ]
    }
  };
}
```

### **8.9 Decryption Process**

```typescript
async function decryptRecord(
  encryptedRecord: EncryptedRecord,
  myDid: string,
  myPrivateKey: PrivateKey
): Promise<Uint8Array> {
  
  // Step 1: Find key encryption entry for me
  const myKeyEntry = encryptedRecord.descriptor.encryption.keyEncryption.find(
    entry => {
      if (entry.derivationScheme === "protocolPath") {
        // Check if I control the root key
        return entry.rootKeyId.startsWith(myDid);
      } else {
        // Check if I'm the direct recipient
        return entry.recipientKeyId === myDid;
      }
    }
  );
  
  if (!myKeyEntry) {
    throw new Error("No decryption key available");
  }
  
  // Step 2: Unwrap symmetric key
  let symmetricKey: Uint8Array;
  
  if (myKeyEntry.derivationScheme === "protocolPath") {
    // Derive the private key from root
    const path = extractPathFromDescriptor(encryptedRecord.descriptor);
    const derivedKey = deriveProtocolPathKey(myPrivateKey, path);
    symmetricKey = await ecdhUnwrapKey(
      myKeyEntry.encryptedKey,
      derivedKey.derivedPrivateKey,
      myKeyEntry.ephemeralPublicKey
    );
  } else {
    // Direct unwrapping
    symmetricKey = await ecdhUnwrapKey(
      myKeyEntry.encryptedKey,
      myPrivateKey,
      myKeyEntry.ephemeralPublicKey
    );
  }
  
  // Step 3: Decrypt data
  const iv = base64urlDecode(encryptedRecord.descriptor.encryption.initializationVector);
  const ciphertext = base64urlDecode(encryptedRecord.encryptedData);
  
  return await aesGcmDecrypt(ciphertext, symmetricKey, iv);
}
```

### **8.10 Blind Indexes for Searchable Encryption**

#### **8.10.1 Index Generation**

```typescript
function generateBlindIndex(
  fieldValue: string,
  fieldName: string,
  searchKey: Uint8Array,
  recordSalt?: Uint8Array
): string {
  // Create field-specific key
  const fieldKey = hmacSha256(searchKey, fieldName);
  
  // Optionally salt per-record (prevents correlation)
  const input = recordSalt 
    ? concat(fieldValue, recordSalt)
    : fieldValue;
  
  // Generate blind index
  const index = hmacSha256(fieldKey, input);
  
  return base64url(index);
}
```

#### **8.10.2 Indexed Record Storage**

```json
{
  "recordId": "bafyrei...",
  "descriptor": {
    "encryption": { /* ... */ }
  },
  "encryptedData": "...",
  "blindIndexes": {
    "patientId": "7a3f9c2e1b4d8a9f...",
    "diagnosis": "c4e8f2a1d9b3c7e5...",
    "dateOfVisit": "9f2e4c8a1b7d3f5e..."
  }
}
```

#### **8.10.3 Querying with Blind Indexes**

```typescript
async function queryEncryptedRecords(
  protocol: string,
  searchField: string,
  searchValue: string,
  searchKey: Uint8Array
): Promise<Record[]> {
  
  // Generate query index
  const queryIndex = generateBlindIndex(
    searchValue,
    searchField,
    searchKey
  );
  
  // Query storage
  const results = await storage.query({
    protocol: protocol,
    [`blindIndexes.${searchField}`]: queryIndex
  });
  
  // Decrypt results
  return await Promise.all(
    results.map(rec => decryptRecord(rec, myDid, myPrivateKey))
  );
}
```

#### **8.10.4 Protocol-Specified Indexes**

```json
{
  "protocol": "https://health.example/medical",
  "structure": {
    "labTest": {
      "$encryption": {
        "required": true,
        "indexing": {
          "strategy": "blind-index",
          "fields": [
            {
              "path": "patientId",
              "type": "blind-index",
              "saltPerRecord": true
            },
            {
              "path": "testType",
              "type": "blind-index",
              "saltPerRecord": false
            },
            {
              "path": "dateCollected",
              "type": "order-preserving-encryption"
            }
          ]
        }
      }
    }
  }
}
```

### **8.11 Key Rotation**

#### **8.11.1 Rotation Process**

When rotating encryption keys (Option A: Re-encrypt all data):

```typescript
async function rotateEncryptionKey(
  tenant: Tenant,
  oldKeyId: string,
  newKeyId: string
): Promise<RotationResult> {
  
  // Step 1: Generate new root key
  const newRootKey = await generateX25519KeyPair();
  
  // Step 2: Update DID document
  await updateDIDDocument(tenant.did, {
    keyAgreement: [
      ...existingKeys,
      {
        id: newKeyId,
        type: "X25519KeyAgreementKey2020",
        publicKeyJwk: newRootKey.publicKeyJwk
      }
    ]
  });
  
  // Step 3: Query all encrypted records using old key
  const encryptedRecords = await queryRecordsByKeyId(oldKeyId);
  
  // Step 4: Re-encrypt each record
  const reencryptionTasks = encryptedRecords.map(async (record) => {
    // Decrypt with old key
    const plaintext = await decryptRecord(record, tenant.did, oldKey);
    
    // Re-encrypt with new key
    const newEncrypted = await encryptRecord(
      plaintext,
      record.descriptor.protocol,
      record.descriptor.schema,
      extractRecipients(record),
      newRootKey
    );
    
    // Update record
    return await updateRecord(record.recordId, newEncrypted);
  });
  
  // Step 5: Execute re-encryption (can be batched)
  await Promise.all(reencryptionTasks);
  
  // Step 6: Mark old key as deprecated (keep for grace period)
  await deprecateKey(oldKeyId, {
    replacedBy: newKeyId,
    gracePeriod: "P90D",  // 90 days
    status: "deprecated"
  });
  
  return {
    success: true,
    recordsReencrypted: encryptedRecords.length,
    newKeyId: newKeyId
  };
}
```

#### **8.11.2 Rotation Metadata**

```json
{
  "keyRotationHistory": [{
    "oldKeyId": "did:example:alice#key-agreement-1",
    "newKeyId": "did:example:alice#key-agreement-2",
    "rotatedAt": "2025-10-24T15:30:00Z",
    "reason": "scheduled-rotation",
    "recordsAffected": 1547,
    "status": "completed"
  }]
}
```

### **8.12 Social Key Recovery**

Using Shamir Secret Sharing (M-of-N threshold scheme):

#### **8.12.1 Key Shard Generation**

```typescript
interface RecoveryShard {
  shardId: string;
  trustee: string;  // DID of trustee
  encryptedShard: string;
  threshold: number;
  totalShards: number;
  createdAt: string;
}

async function generateRecoveryShards(
  rootKey: PrivateKey,
  trustees: string[],  // DIDs of trustees
  threshold: number
): Promise<RecoveryShard[]> {
  
  // Step 1: Split key using Shamir Secret Sharing
  const shards = shamirSplit(rootKey.keyBytes, threshold, trustees.length);
  
  // Step 2: Encrypt each shard for its trustee
  const encryptedShards = await Promise.all(
    shards.map(async (shard, index) => {
      const trusteePublicKey = await resolveKeyAgreementKey(trustees[index]);
      const encrypted = await ecdhEncrypt(shard, trusteePublicKey);
      
      return {
        shardId: `shard-${index + 1}`,
        trustee: trustees[index],
        encryptedShard: base64url(encrypted),
        threshold: threshold,
        totalShards: trustees.length,
        createdAt: new Date().toISOString()
      };
    })
  );
  
  // Step 3: Distribute shards to trustees
  await Promise.all(
    encryptedShards.map(shard => 
      sendShardToTrustee(shard.trustee, shard)
    )
  );
  
  return encryptedShards;
}
```

#### **8.12.2 Key Recovery Process**

```typescript
async function recoverKey(
  shards: RecoveryShard[],
  myDid: string,
  myPrivateKey: PrivateKey
): Promise<PrivateKey> {
  
  // Step 1: Verify threshold
  if (shards.length < shards[0].threshold) {
    throw new Error(`Need ${shards[0].threshold} shards, got ${shards.length}`);
  }
  
  // Step 2: Decrypt shards
  const decryptedShards = await Promise.all(
    shards.map(async (shard) => {
      // Verify I'm the trustee
      if (shard.trustee !== myDid) {
        throw new Error("Shard not intended for me");
      }
      
      return await ecdhDecrypt(shard.encryptedShard, myPrivateKey);
    })
  );
  
  // Step 3: Recombine key
  const recoveredKey = shamirCombine(decryptedShards);
  
  // Step 4: Verify recovered key
  const publicKey = x25519.getPublicKey(recoveredKey);
  // Compare with original public key from DID document
  
  return {
    keyBytes: recoveredKey,
    publicKeyJwk: bytesToJwk(publicKey)
  };
}
```

#### **8.12.3 Recovery Metadata**

```json
{
  "recoveryConfiguration": {
    "scheme": "shamir-secret-sharing",
    "threshold": 3,
    "totalShards": 5,
    "trustees": [
      "did:example:trustee1",
      "did:example:trustee2",
      "did:example:trustee3",
      "did:example:trustee4",
      "did:example:trustee5"
    ],
    "createdAt": "2025-01-15T10:00:00Z",
    "instructions": "Contact trustees via secure channel"
  }
}
```

---

## **9. Synchronization & Replication**

### **9.1 Mesh Architecture**

Multiple DWN nodes owned by the same DID synchronize via a mesh topology:

```
     Node 1 (US-East)
        ◄──────►
       /        \
      /          \
     ◄            ◄
    /              \
Node 2           Node 3
(EU-West)      (Asia-Pacific)
    \              /
     ◄            ►
      \          /
       \        /
        ◄──────►
```

### **9.2 Sync Configuration**

#### **9.2.1 Tenant-Level Sync Policy**

```json
{
  "tenantDid": "did:example:alice",
  "syncPolicy": {
    "enabled": true,
    "frequency": "PT5M",  // Every 5 minutes
    "conflictResolution": {
      "strategy": "crdt",
      "crdtType": "lww-element-set",
      "fallback": "manual"
    },
    "nodes": [
      {
        "nodeId": "node-us-east",
        "endpoint": "https://dwn-us-east.alice.example",
        "priority": 1,
        "role": "primary"
      },
      {
        "nodeId": "node-eu-west",
        "endpoint": "https://dwn-eu-west.alice.example",
        "priority": 2,
        "role": "replica"
      },
      {
        "nodeId": "node-asia-pacific",
        "endpoint": "https://dwn-asia-pacific.alice.example",
        "priority": 3,
        "role": "replica"
      }
    ],
    "selectiveSync": {
      "shards": [
        {
          "name": "healthcare",
          "protocols": ["https://health.example/medical"],
          "nodes": ["node-us-east"],
          "compliance": "HIPAA",
          "region": "US"
        },
        {
          "name": "financial",
          "protocols": ["https://finance.example/banking"],
          "nodes": ["node-eu-west"],
          "compliance": "GDPR",
          "region": "EU"
        },
        {
          "name": "general",
          "protocols": ["*"],
          "nodes": ["*"],
          "compliance": "none",
          "region": "*"
        }
      ]
    }
  }
}
```

### **9.3 Selective Synchronization**

#### **9.3.1 Shard-Based Sync**

Records are assigned to shards based on protocol:

```typescript
interface Shard {
  name: string;
  protocols: string[];  // Protocol URIs or "*"
  nodes: string[];      // Node IDs or "*"
  compliance: string;   // Regulatory requirement
  region: string;       // Geographic region
}

function determineShard(record: Record, shards: Shard[]): Shard | null {
  // Find most specific matching shard
  const matches = shards.filter(shard => {
    if (shard.protocols.includes("*")) return true;
    return shard.protocols.includes(record.descriptor.protocol);
  });
  
  // Return most specific (non-wildcard preferred)
  return matches.find(s => !s.protocols.includes("*")) || matches[0];
}
```

#### **9.3.2 Compliance-Driven Replication**

```typescript
async function syncRecord(
  record: Record,
  syncPolicy: SyncPolicy
): Promise<SyncResult> {
  
  // Determine shard
  const shard = determineShard(record, syncPolicy.selectiveSync.shards);
  
  if (!shard) {
    throw new Error("No shard configured for record protocol");
  }
  
  // Verify compliance
  if (shard.compliance !== "none") {
    await verifyCompliance(record, shard.compliance, shard.region);
  }
  
  // Sync to designated nodes only
  const targetNodes = shard.nodes.includes("*")
    ? syncPolicy.nodes
    : syncPolicy.nodes.filter(n => shard.nodes.includes(n.nodeId));
  
  // Execute sync
  const results = await Promise.all(
    targetNodes.map(node => syncToNode(record, node))
  );
  
  return {
    success: results.every(r => r.success),
    syncedNodes: results.filter(r => r.success).map(r => r.nodeId),
    shard: shard.name
  };
}
```

### **9.4 CRDT-Based Conflict Resolution**

#### **9.4.1 LWW-Element-Set (Last-Write-Wins Element Set)**

```typescript
interface LWWElementSet<T> {
  adds: Map<T, Timestamp>;
  removes: Map<T, Timestamp>;
}

class LWWSet<T> {
  private adds: Map<T, number>;
  private removes: Map<T, number>;
  
  constructor() {
    this.adds = new Map();
    this.removes = new Map();
  }
  
  add(element: T, timestamp: number): void {
    const currentAdd = this.adds.get(element) || 0;
    if (timestamp > currentAdd) {
      this.adds.set(element, timestamp);
    }
  }
  
  remove(element: T, timestamp: number): void {
    const currentRemove = this.removes.get(element) || 0;
    if (timestamp > currentRemove) {
      this.removes.set(element, timestamp);
    }
  }
  
  contains(element: T): boolean {
    const addTime = this.adds.get(element) || 0;
    const removeTime = this.removes.get(element) || 0;
    
    // Bias towards add in case of tie
    return addTime >= removeTime && addTime > 0;
  }
  
  merge(other: LWWSet<T>): LWWSet<T> {
    const merged = new LWWSet<T>();
    
    // Merge adds
    for (const [elem, ts] of this.adds) {
      merged.add(elem, ts);
    }
    for (const [elem, ts] of other.adds) {
      merged.add(elem, ts);
    }
    
    // Merge removes
    for (const [elem, ts] of this.removes) {
      merged.remove(elem, ts);
    }
    for (const [elem, ts] of other.removes) {
      merged.remove(elem, ts);
    }
    
    return merged;
  }
}
```

#### **9.4.2 JSON CRDT for Structured Data**

```typescript
interface JSONCRDTOperation {
  type: 'set' | 'delete' | 'arrayInsert' | 'arrayDelete';
  path: string[];
  value?: any;
  timestamp: number;
  nodeId: string;
}

class JSONCRDT {
  private operations: JSONCRDTOperation[];
  
  constructor() {
    this.operations = [];
  }
  
  apply(op: JSONCRDTOperation): void {
    this.operations.push(op);
    this.operations.sort((a, b) => {
      if (a.timestamp !== b.timestamp) {
        return a.timestamp - b.timestamp;
      }
      // Tie-breaker: node ID
      return a.nodeId.localeCompare(b.nodeId);
    });
  }
  
  materialize(): any {
    let doc = {};
    
    for (const op of this.operations) {
      const path = op.path;
      let current = doc;
      
      // Navigate to parent
      for (let i = 0; i < path.length - 1; i++) {
        if (!(path[i] in current)) {
          current[path[i]] = {};
        }
        current = current[path[i]];
      }
      
      // Apply operation
      const key = path[path.length - 1];
      switch (op.type) {
        case 'set':
          current[key] = op.value;
          break;
        case 'delete':
          delete current[key];
          break;
        case 'arrayInsert':
          if (!Array.isArray(current[key])) {
            current[key] = [];
          }
          current[key].push(op.value);
          break;
        case 'arrayDelete':
          if (Array.isArray(current[key])) {
            const index = current[key].indexOf(op.value);
            if (index !== -1) {
              current[key].splice(index, 1);
            }
          }
          break;
      }
    }
    
    return doc;
  }
  
  merge(other: JSONCRDT): JSONCRDT {
    const merged = new JSONCRDT();
    
    // Merge and sort all operations
    const allOps = [...this.operations, ...other.operations];
    allOps.sort((a, b) => {
      if (a.timestamp !== b.timestamp) {
        return a.timestamp - b.timestamp;
      }
      return a.nodeId.localeCompare(b.nodeId);
    });
    
    merged.operations = allOps;
    return merged;
  }
}
```

#### **9.4.3 Protocol-Specified CRDT Strategy**

```json
{
  "protocol": "https://example.com/collaborative-doc",
  "version": "1.0.0",
  "conflictResolution": {
    "strategy": "crdt",
    "crdtType": "json-crdt",
    "fallback": "manual"
  },
  "structure": {
    "document": {
      "$commitStrategy": "json-crdt",
      "$size": { "max": 10000000 }
    }
  }
}
```

### **9.5 Sync Protocol Messages**

#### **9.5.1 Node Authentication**

Before syncing, nodes must prove they belong to the same tenant:

```json
{
  "descriptor": {
    "interface": "Sync",
    "method": "Authenticate",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "nodeId": "node-eu-west",
    "tenantDid": "did:example:alice"
  },
  "authorization": {
    "payload": {
      "descriptorCid": "bafyrei...",
      "capability": "urn:uuid:sync-capability"
    },
    "signatures": [{
      "protected": {
        "alg": "EdDSA",
        "kid": "did:example:alice#key-1"
      },
      "signature": "..."
    }]
  }
}
```

**Verification:**
- Signature MUST be from a key in tenant's DID Document
- Capability MUST grant sync permissions
- Node MUST be listed in tenant's sync policy

#### **9.5.2 Sync Pull Request**

```json
{
  "descriptor": {
    "interface": "Sync",
    "method": "Pull",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "watermark": "bafyrei_last_synced_message_cid",
    "protocols": ["https://example.com/social"],
    "limit": 1000
  }
}
```

**Response:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "messages": [
    { /* RecordsWrite message 1 */ },
    { /* RecordsWrite message 2 */ },
    { /* RecordsDelete message 1 */ }
  ],
  "watermark": "bafyrei_new_watermark",
  "hasMore": false
}
```

#### **9.5.3 Sync Push Operation**

```json
{
  "descriptor": {
    "interface": "Sync",
    "method": "Push",
    "messageTimestamp": "2025-10-24T15:30:00Z"
  },
  "messages": [
    { /* RecordsWrite message to sync */ },
    { /* RecordsWrite message to sync */ }
  ]
}
```

**Response:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "results": [
    {
      "recordId": "bafyrei...",
      "status": { "code": 200, "detail": "Synced" }
    },
    {
      "recordId": "bafyrei...",
      "status": { "code": 409, "detail": "Conflict - merged via CRDT" }
    }
  ]
}
```

### **9.6 Conflict Detection and Resolution**

#### **9.6.1 Conflict Detection**

A conflict occurs when:
1. Same `recordId` modified on multiple nodes
2. Modifications have concurrent timestamps (no causal relationship)
3. Modifications cannot be automatically merged

```typescript
function detectConflict(
  localRecord: Record,
  remoteRecord: Record
): boolean {
  
  // Same record?
  if (localRecord.recordId !== remoteRecord.recordId) {
    return false;
  }
  
  // Check causality via parentId
  if (localRecord.descriptor.parentId === remoteRecord.recordId ||
      remoteRecord.descriptor.parentId === localRecord.recordId) {
    return false;  // One is ancestor of other, not a conflict
  }
  
  // Check if both modified after last common ancestor
  const commonAncestor = findCommonAncestor(localRecord, remoteRecord);
  
  if (!commonAncestor) {
    return true;  // Concurrent modifications
  }
  
  return false;
}
```

#### **9.6.2 CRDT Resolution**

```typescript
async function resolveConflictCRDT(
  localRecord: Record,
  remoteRecord: Record,
  crdtType: string
): Promise<Record> {
  
  // Decrypt both records
  const localData = await decryptRecord(localRecord, myDid, myKey);
  const remoteData = await decryptRecord(remoteRecord, theirDid, theirKey);
  
  let mergedData: any;
  
  switch (crdtType) {
    case 'lww-element-set':
      mergedData = mergeLWWSet(localData, remoteData);
      break;
      
    case 'json-crdt':
      mergedData = mergeJSONCRDT(localData, remoteData);
      break;
      
    case 'operational-transform':
      mergedData = mergeOT(localData, remoteData);
      break;
      
    default:
      throw new Error(`Unsupported CRDT type: ${crdtType}`);
  }
  
  // Create new merged record
  const mergedRecord = await createRecord(
    mergedData,
    localRecord.descriptor.protocol,
    localRecord.descriptor.schema
  );
  
  // Link to both parents (merge commit)
  mergedRecord.descriptor.mergeParents = [
    localRecord.descriptor.descriptorCid,
    remoteRecord.descriptor.descriptorCid
  ];
  
  return mergedRecord;
}
```

#### **9.6.3 Manual Conflict Resolution**

When CRDT fails or not applicable:

```json
{
  "recordId": "bafyrei...",
  "conflictStatus": "unresolved",
  "conflictVersions": [
    {
      "nodeId": "node-us-east",
      "descriptorCid": "bafyrei_version_a",
      "dateCreated": "2025-10-24T15:30:00Z",
      "data": { /* version A data */ }
    },
    {
      "nodeId": "node-eu-west",
      "descriptorCid": "bafyrei_version_b",
      "dateCreated": "2025-10-24T15:30:05Z",
      "data": { /* version B data */ }
    }
  ],
  "requiresManualResolution": true
}
```

User resolves by creating a new version that references both as merge parents.

---

## **10. Query System**

### **10.1 Basic Query Interface**

```json
{
  "descriptor": {
    "interface": "Records",
    "method": "Query",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "filter": {
      "protocol": "https://example.com/social",
      "schema": "https://schema.org/SocialMediaPosting",
      "contextId": "thread-abc-123",
      "attester": "did:example:bob",
      "recipient": "did:example:alice",
      "dateCreated": {
        "from": "2025-01-01T00:00:00Z",
        "to": "2025-12-31T23:59:59Z"
      },
      "dataFormat": "application/json"
    },
    "dateSort": "createdDescending",
    "pagination": {
      "limit": 50,
      "cursor": "bafyrei_pagination_cursor"
    }
  },
  "authorization": { /* ZCAP with query permission */ }
}
```

### **10.2 Filter Options**

**Supported Filters:**
- `protocol`: Protocol URI
- `protocolVersion`: SemVer or range (e.g., "1.x.x")
- `schema`: Schema URI
- `recordId`: Specific record ID
- `parentId`: Parent record ID
- `contextId`: Context grouping ID
- `attester`: DID of record creator
- `recipient`: DID of intended recipient
- `dataFormat`: MIME type
- `dateCreated`: Date range filter
- `datePublished`: Date range filter
- `published`: Boolean
- `tags`: Array of tags (if protocol supports)

### **10.3 Pagination**

```typescript
interface PaginationParams {
  limit: number;       // Max results per page (1-1000)
  cursor?: string;     // Cursor for next page
  offset?: number;     // Alternative to cursor
}

interface PaginatedResponse {
  status: { code: number; detail: string };
  entries: Record[];
  pagination: {
    limit: number;
    count: number;      // Results in this page
    total?: number;     // Total matching records (expensive to compute)
    cursor?: string;    // Cursor for next page
    hasMore: boolean;
  };
}
```

**Example:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "entries": [
    { /* Record 1 */ },
    { /* Record 2 */ }
  ],
  "pagination": {
    "limit": 50,
    "count": 50,
    "cursor": "bafyrei_next_page_cursor",
    "hasMore": true
  }
}
```

### **10.4 Basic SPARQL Subset**

For RDF-compatible protocols, support basic SPARQL:

```sparql
PREFIX dwn: <https://identity.foundation/dwn/vocab#>
PREFIX schema: <https://schema.org/>

SELECT ?post ?author ?content ?date
WHERE {
  ?post a schema:SocialMediaPosting ;
        schema:author ?author ;
        schema:text ?content ;
        dwn:protocol <https://example.com/social> ;
        dwn:dateCreated ?date .
  FILTER(?date > "2025-01-01T00:00:00Z"^^xsd:dateTime)
}
ORDER BY DESC(?date)
LIMIT 10
```

**Supported SPARQL Features:**
- SELECT queries
- WHERE clause with basic triple patterns
- FILTER with comparison operators (<, >, <=, >=, =, !=)
- ORDER BY (ASC/DESC)
- LIMIT and OFFSET

**Not Supported (for simplicity):**
- CONSTRUCT, DESCRIBE, ASK
- OPTIONAL, UNION
- Aggregations (COUNT, SUM, etc.)
- Subqueries
- Property paths

### **10.5 Graph Queries**

Query relationships between records:

```json
{
  "descriptor": {
    "interface": "Records",
    "method": "GraphQuery",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "startNode": "bafyrei_post_record_id",
    "traversal": [{
      "relation": "parentId",
      "direction": "outbound",
      "depth": 3,
      "filter": {
        "schema": "https://schema.org/Comment"
      }
    }, {
      "relation": "contextId",
      "direction": "both",
      "filter": {
        "protocol": "https://example.com/social"
      }
    }]
  }
}
```

**Response:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "graph": {
    "nodes": [
      {
        "recordId": "bafyrei_post",
        "type": "post",
        "data": { /* ... */ }
      },
      {
        "recordId": "bafyrei_comment1",
        "type": "comment",
        "data": { /* ... */ }
      }
    ],
    "edges": [
      {
        "from": "bafyrei_comment1",
        "to": "bafyrei_post",
        "relation": "parentId"
      }
    ]
  }
}
```

### **10.6 Querying Encrypted Data**

Using blind indexes:

```typescript
async function queryEncryptedHealthRecords(
  patientId: string,
  searchKey: Uint8Array
): Promise<Record[]> {
  
  // Generate blind index for patient ID
  const patientIndex = generateBlindIndex(
    patientId,
    "patientId",
    searchKey,
    null  // No per-record salt for patient ID
  );
  
  // Query using blind index
  const results = await query({
    descriptor: {
      interface: "Records",
      method: "Query",
      filter: {
        protocol: "https://health.example/medical",
        [`blindIndexes.patientId`]: patientIndex
      }
    }
  });
  
  // Decrypt results client-side
  return await Promise.all(
    results.entries.map(r => decryptRecord(r, myDid, myKey))
  );
}
```

---

## **11. Notification System**

### **11.1 Subscription Interface**

```json
{
  "descriptor": {
    "interface": "Records",
    "method": "Subscribe",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "filter": {
      "protocol": "https://example.com/social",
      "contextId": "thread-abc-123"
    },
    "deliveryConfig": {
      "transport": "websocket",
      "endpoint": "wss://client.example/notifications",
      "format": "full-record"
    }
  },
  "authorization": { /* ZCAP with subscribe permission */ }
}
```

**Response:**
```json
{
  "status": { "code": 201, "detail": "Subscription created" },
  "subscriptionId": "sub-uuid-123",
  "expiresAt": "2025-11-24T15:30:00Z"
}
```

### **11.2 Delivery Transports**

**Supported Transports:**
- **WebSocket**: Real-time push notifications
- **Webhook**: HTTP POST to client endpoint
- **Server-Sent Events (SSE)**: One-way server push
- **DIDComm**: Encrypted message delivery
- **gRPC Stream**: Bidirectional streaming

**Transport Configuration:**

**WebSocket:**
```json
{
  "transport": "websocket",
  "endpoint": "wss://client.example/notifications",
  "format": "full-record",
  "reconnect": true
}
```

**Webhook:**
```json
{
  "transport": "webhook",
  "endpoint": "https://client.example/webhook",
  "format": "metadata-only",
  "retryPolicy": {
    "maxRetries": 3,
    "backoff": "exponential"
  },
  "authentication": {
    "type": "bearer",
    "token": "secret_token"
  }
}
```

### **11.3 Notification Format**

**Full Record:**
```json
{
  "type": "notification",
  "subscriptionId": "sub-uuid-123",
  "event": "records.create",
  "timestamp": "2025-10-24T15:30:00Z",
  "record": {
    "recordId": "bafyrei...",
    "descriptor": { /* Full descriptor */ },
    "data": "base64url_data",
    "encryption": { /* Encryption metadata */ }
  }
}
```

**Metadata Only:**
```json
{
  "type": "notification",
  "subscriptionId": "sub-uuid-123",
  "event": "records.update",
  "timestamp": "2025-10-24T15:35:00Z",
  "metadata": {
    "recordId": "bafyrei...",
    "protocol": "https://example.com/social",
    "schema": "https://schema.org/SocialMediaPosting",
    "author": "did:example:bob",
    "dateCreated": "2025-10-24T15:35:00Z"
  }
}
```

### **11.4 Event Types**

- `records.create`: New record created
- `records.update`: Existing record updated
- `records.delete`: Record deleted
- `protocols.configure`: Protocol installed/updated
- `sync.complete`: Sync operation completed
- `subscription.expired`: Subscription expired

### **11.5 Subscription Management**

#### **11.5.1 List Subscriptions**

```json
{
  "descriptor": {
    "interface": "Subscriptions",
    "method": "Query",
    "messageTimestamp": "2025-10-24T15:30:00Z"
  }
}
```

**Response:**
```json
{
  "status": { "code": 200, "detail": "OK" },
  "subscriptions": [
    {
      "subscriptionId": "sub-uuid-123",
      "filter": { /* Filter criteria */ },
      "deliveryConfig": { /* Delivery config */ },
      "createdAt": "2025-10-24T15:00:00Z",
      "expiresAt": "2025-11-24T15:00:00Z",
      "status": "active",
      "deliveryStats": {
        "delivered": 150,
        "failed": 2,
        "pending": 0
      }
    }
  ]
}
```

#### **11.5.2 Cancel Subscription**

```json
{
  "descriptor": {
    "interface": "Subscriptions",
    "method": "Delete",
    "messageTimestamp": "2025-10-24T15:30:00Z",
    "subscriptionId": "sub-uuid-123"
  },
  "authorization": { /* Owner capability */ }
}
```

### **11.6 Subscription Limits and Metering**

#### **11.6.1 Subscription Quotas**

```json
{
  "tenantDid": "did:example:alice",
  "subscriptionQuota": {
    "maxSubscriptions": 100,
    "maxNotificationsPerHour": 10000,
    "maxNotificationsPerDay": 100000,
    "tier": "premium"
  },
  "currentUsage": {
    "activeSubscriptions": 12,
    "notificationsThisHour": 1547,
    "notificationsToday": 45632
  }
}
```

#### **11.6.2 Rate Limiting**

```json
{
  "status": { "code": 429, "detail": "Subscription quota exceeded" },
  "error": {
    "type": "RateLimitExceeded",
    "message": "Maximum subscriptions reached (100/100)",
    "retryAfter": "2025-10-25T00:00:00Z"
  }
}
```

### **11.7 Subscription Access Control**

Only authorized agents can subscribe:

```json
{
  "@context": ["https://w3id.org/zcap/v1"],
  "id": "urn:uuid:subscribe-capability",
  "type": ["VerifiableCredential", "Authorization"],
  "issuer": "did:example:alice",
  "credentialSubject": {
    "id": "did:example:bob",
    "invocationTarget": {
      "id": "did:example:alice",
      "allowedActions": ["subscribe"],
      "constraints": {
        "protocol": "https://example.com/social",
        "maxSubscriptions": 5
      }
    }
  },
  "proof": { /* Signature */ }
}
```

### **11.8 Subscription Persistence**

Subscriptions persist across node restarts:

```typescript
interface PersistedSubscription {
  subscriptionId: string;
  tenantDid: string;
  subscriberDid: string;
  filter: QueryFilter;
  deliveryConfig: DeliveryConfig;
  capability: string;  // Capability ID
  createdAt: string;
  expiresAt: string;
  status: "active" | "paused" | "expired";
  lastDelivery?: string;
  deliveryStats: {
    delivered: number;
    failed: number;
    pending: number;
  };
}

// Stored in persistent database
await db.subscriptions.insert(persistedSubscription);

// Restored on node startup
async function restoreSubscriptions() {
  const subs = await db.subscriptions.find({ status: "active" });
  for (const sub of subs) {
    await reestablishSubscription(sub);
  }
}
```

---

## **12. Transport Layer**

### **12.1 Transport Abstraction**

All transports present a unified message interface:

```typescript
interface TransportAdapter {
  name: string;  // "http", "grpc", "didcomm", "websocket"
  
  // Initialize transport
  init(config: TransportConfig): Promise<void>;
  
  // Receive message
  receive(): AsyncIterator<Message>;
  
  // Send message
  send(message: Message, endpoint: string): Promise<TransportResponse>;
  
  // Close transport
  close(): Promise<void>;
}
```

### **12.2 HTTP Transport**

#### **12.2.1 Endpoint Structure**

```
Base URL: https://dwn.alice.example

Endpoints:
  POST   /messages              - Send message(s)
  GET    /records/{recordId}    - Read record
  POST   /records/query         - Query records
  POST   /protocols/query       - Query protocols
  POST   /sync/pull             - Pull sync changes
  POST   /sync/push             - Push sync changes
  WS     /subscribe             - WebSocket subscriptions
```

#### **12.2.2 Message Request/Response**

**Request:**
```http
POST /messages HTTP/1.1
Host: dwn.alice.example
Content-Type: application/json
Authorization: DPoP <access_token>

{
  "messages": [
    { /* RecordsWrite message */ },
    { /* RecordsQuery message */ }
  ]
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "replies": [
    {
      "status": { "code": 201, "detail": "Created" },
      "recordId": "bafyrei..."
    },
    {
      "status": { "code": 200, "detail": "OK" },
      "entries": [
        { /* Record 1 */ },
        { /* Record 2 */ }
      ]
    }
  ]
}
```

#### **12.2.3 RESTful Shortcuts**

**Create Record (POST):**
```http
POST /records HTTP/1.1
Content-Type: application/json

{
  "protocol": "https://example.com/social",
  "schema": "https://schema.org/SocialMediaPosting",
  "data": { /* Record data */ }
}
```

Internally converted to `RecordsWrite` message.

**Read Record (GET):**
```http
GET /records/bafyrei... HTTP/1.1
Accept: application/json
```

Internally converted to `RecordsRead` message.

**Update Record (PUT):**
```http
PUT /records/bafyrei... HTTP/1.1
Content-Type: application/json

{
  "data": { /* Updated data */ }
}
```

Internally converted to `RecordsWrite` with `parentId`.

**Delete Record (DELETE):**
```http
DELETE /records/bafyrei... HTTP/1.1
```

Internally converted to `RecordsDelete` message.

### **12.3 gRPC Transport**

#### **12.3.1 Service Definition**

```protobuf
syntax = "proto3";

package dwn.v1;

service DWNService {
  // Single message
  rpc SendMessage(Message) returns (MessageResult);
  
  // Batch messages
  rpc SendMessages(stream Message) returns (stream MessageResult);
  
  // Query
  rpc Query(QueryMessage) returns (QueryResult);
  
  // Subscribe (bidirectional streaming)
  rpc Subscribe(SubscribeMessage) returns (stream Notification);
  
  // Sync
  rpc SyncPull(SyncPullMessage) returns (stream SyncMessage);
  rpc SyncPush(stream SyncMessage) returns (SyncPushResult);
}

message Message {
  string record_id = 1;
  string context_id = 2;
  MessageDescriptor descriptor = 3;
  bytes data = 4;
  EncryptionMetadata encryption = 5;
  GeneralJWS authorization = 6;
}

message MessageResult {
  Status status = 1;
  string record_id = 2;
  repeated Record entries = 3;
}

message Status {
  int32 code = 1;
  string detail = 2;
}
```

#### **12.3.2 gRPC Client Example**

```typescript
import { DWNServiceClient } from './generated/dwn_grpc_pb';

const client = new DWNServiceClient(
  'dwn.alice.example:50051',
  credentials.createSsl()
);

// Send message
const message = new Message()
  .setRecordId('bafyrei...')
  .setDescriptor(descriptor)
  .setData(data)
  .setAuthorization(authorization);

const result = await client.sendMessage(message);
```

### **12.4 DIDComm Transport**

#### **12.4.1 DIDComm Message Wrapper**

```json
{
  "type": "https://didcomm.org/dwn/1.0/message",
  "id": "uuid-123",
  "from": "did:example:bob",
  "to": ["did:example:alice"],
  "created_time": 1698172800,
  "body": {
    "dwnMessage": {
      "recordId": "bafyrei...",
      "descriptor": { /* ... */ },
      "data": "base64url_data",
      "authorization": { /* ... */ }
    }
  },
  "attachments": []
}
```

#### **12.4.2 Routing**

DIDComm messages are routed via mediators:

```
Client (Bob)
    │
    └──► DIDComm Message
            │
            └──► Mediator
                    │
                    └──► DWN Service Endpoint (Alice)
```

### **12.5 WebSocket Transport**

#### **12.5.1 Connection Establishment**

```javascript
const ws = new WebSocket('wss://dwn.alice.example/ws');

// Authenticate
ws.send(JSON.stringify({
  type: 'auth',
  capability: 'urn:uuid:cap-123',
  proof: { /* JWS */ }
}));

// Send message
ws.send(JSON.stringify({
  type: 'message',
  id: 'msg-uuid-456',
  payload: {
    recordId: 'bafyrei...',
    descriptor: { /* ... */ },
    data: 'base64url_data',
    authorization: { /* ... */ }
  }
}));

// Receive response
ws.onmessage = (event) => {
  const response = JSON.parse(event.data);
  console.log(response);
};
```

#### **12.5.2 Subscription Delivery**

```json
{
  "type": "notification",
  "subscriptionId": "sub-uuid-123",
  "event": "records.create",
  "timestamp": "2025-10-24T15:30:00Z",
  "record": { /* Full record */ }
}
```

### **12.6 Transport Negotiation**

Single endpoint with automatic transport detection:

```
Client Request:
  HTTP/1.1 → HTTP adapter
  HTTP/2 with application/grpc → gRPC adapter
  WebSocket Upgrade → WebSocket adapter
  Content-Type: application/didcomm-encrypted+json → DIDComm adapter
```

**Capability Advertisement:**
```json
{
  "serviceEndpoint": "https://dwn.alice.example",
  "transports": {
    "http": {
      "versions": ["1.1", "2"],
      "methods": ["GET", "POST", "PUT", "DELETE"]
    },
    "grpc": {
      "available": true,
      "services": ["DWNService"]
    },
    "websocket": {
      "available": true,
      "endpoint": "wss://dwn.alice.example/ws"
    },
    "didcomm": {
      "available": true,
      "versions": ["2.0"]
    }
  }
}
```

---

## **13. Error Handling**

### **13.1 Error Model**

**Request-Level Errors:**
```json
{
  "status": {
    "code": 401,
    "detail": "Authentication required"
  }
}
```

**Message-Level Errors:**
```json
{
  "replies": [
    {
      "status": {
        "code": 400,
        "detail": "Malformed message descriptor"
      },
      "error": {
        "type": "ValidationError",
        "field": "descriptor.dateCreated",
        "message": "Invalid ISO 8601 timestamp"
      }
    }
  ]
}
```

### **13.2 Error Codes**

**Success (2xx):**
- `200` - OK
- `201` - Created
- `202` - Accepted (async processing)

**Client Errors (4xx):**
- `400` - Bad Request (malformed message)
- `401` - Unauthorized (invalid/missing capability)
- `403` - Forbidden (valid auth, insufficient permissions)
- `404` - Not Found
- `409` - Conflict (protocol constraint, CRDT merge conflict)
- `422` - Unprocessable Entity (invalid protocol definition)
- `429` - Too Many Requests (rate limit)

**Server Errors (5xx):**
- `500` - Internal Server Error
- `501` - Not Implemented (unsupported method)
- `503` - Service Unavailable

### **13.3 Machine-Readable Error Types**

```typescript
enum ErrorType {
  // Validation errors
  VALIDATION_ERROR = "ValidationError",
  SCHEMA_VALIDATION_ERROR = "SchemaValidationError",
  
  // Authorization errors
  INVALID_CAPABILITY = "InvalidCapability",
  EXPIRED_CAPABILITY = "ExpiredCapability",
  INSUFFICIENT_PERMISSIONS = "InsufficientPermissions",
  
  // Protocol errors
  PROTOCOL_NOT_FOUND = "ProtocolNotFound",
  PROTOCOL_CONSTRAINT_VIOLATION = "ProtocolConstraintViolation",
  SIZE_LIMIT_EXCEEDED = "SizeLimitExceeded",
  
  // Sync errors
  SYNC_CONFLICT = "SyncConflict",
  NODE_AUTHENTICATION_FAILED = "NodeAuthenticationFailed",
  
  // Encryption errors
  DECRYPTION_FAILED = "DecryptionFailed",
  KEY_NOT_FOUND = "KeyNotFound",
  
  // Resource errors
  RECORD_NOT_FOUND = "RecordNotFound",
  CONTEXT_NOT_FOUND = "ContextNotFound",
  
  // Rate limiting
  RATE_LIMIT_EXCEEDED = "RateLimitExceeded",
  QUOTA_EXCEEDED = "QuotaExceeded"
}
```

**Error Response Example:**
```json
{
  "status": {
    "code": 409,
    "detail": "Protocol constraint violated"
  },
  "error": {
    "type": "ProtocolConstraintViolation",
    "protocol": "https://example.com/social",
    "constraint": "$size.max",
    "message": "Record size (1.5MB) exceeds protocol limit (1MB)",
    "field": "data"
  }
}
```

### **13.4 Security-Conscious Error Messages**

**Bad: Reveals too much**
```json
{
  "error": {
    "message": "Record bafyrei... exists but belongs to did:example:alice, not you"
  }
}
```

**Good: Minimal disclosure**
```json
{
  "status": { "code": 404, "detail": "Not found" }
}
```

**Rationale:** Avoid leaking information about resource existence or ownership to unauthorized parties.

---

## **14. Security Considerations**

### **14.1 Threat Model**

**Threats:**
1. **Unauthorized Access**: Attackers gaining access to private records
2. **Capability Theft**: Stolen capabilities used to perform unauthorized actions
3. **Man-in-the-Middle**: Interception of data in transit
4. **Replay Attacks**: Reusing captured messages
5. **Denial of Service**: Resource exhaustion attacks
6. **Data Tampering**: Modification of records in storage
7. **Side-Channel Attacks**: Timing attacks, traffic analysis
8. **Social Engineering**: Tricking users into granting excessive capabilities

### **14.2 Mitigations**

#### **14.2.1 Transport Security**

**MUST:**
- Use TLS 1.3 for all HTTP connections
- Use authenticated encryption for DIDComm
- Validate certificates
- Use certificate pinning where appropriate

#### **14.2.2 Capability Security**

**MUST:**
- Verify capability signatures
- Check expiration dates
- Validate delegation chains
- Revoke compromised capabilities
- Use short expiration times for sensitive operations

**SHOULD:**
- Implement capability rotation
- Monitor for anomalous capability usage
- Rate-limit capability invocations

#### **14.2.3 Message Authentication**

**MUST:**
- Sign all messages with DID keys
- Verify signatures before processing
- Check message timestamps (prevent replay)
- Use nonces where appropriate

**Message Timestamp Validation:**
```typescript
const MAX_MESSAGE_AGE = 300000; // 5 minutes

function validateMessageTimestamp(timestamp: string): boolean {
  const msgTime = new Date(timestamp).getTime();
  const now = Date.now();
  
  // Reject if too old
  if (now - msgTime > MAX_MESSAGE_AGE) {
    return false;
  }
  
  // Reject if too far in future (clock skew tolerance)
  if (msgTime - now > 60000) {  // 1 minute
    return false;
  }
  
  return true;
}
```

#### **14.2.4 Encryption Security**

**MUST:**
- Use authenticated encryption (AES-GCM, not AES-CBC)
- Generate unique IVs for each encryption
- Verify authentication tags
- Use secure key derivation (HKDF)
- Protect private keys (HSM, secure enclave)

**SHOULD:**
- Implement key rotation schedules
- Monitor for weak keys
- Use hardware-backed keystores

#### **14.2.5 Rate Limiting**

**MUST:**
- Implement per-DID rate limits
- Implement per-IP rate limits (for HTTP)
- Implement per-capability rate limits
- Return 429 status code when exceeded

**Example:**
```typescript
const rateLimits = {
  perDID: {
    windowMs: 60000,      // 1 minute
    maxRequests: 1000
  },
  perCapability: {
    windowMs: 60000,
    maxRequests: 100
  }
};
```

#### **14.2.6 Input Validation**

**MUST:**
- Validate all message descriptors against schemas
- Sanitize string inputs
- Validate CIDs
- Check data sizes
- Validate protocol definitions

**MUST NOT:**
- Trust client-provided CIDs without verification
- Execute user-provided code
- Allow arbitrary file system access

#### **14.2.7 Side-Channel Protection**

**SHOULD:**
- Implement constant-time string comparison
- Avoid timing-based information leakage
- Use padding for encrypted data
- Implement query result limiting

**Constant-Time Comparison:**
```typescript
function constantTimeEqual(a: Uint8Array, b: Uint8Array): boolean {
  if (a.length !== b.length) {
    return false;
  }
  
  let result = 0;
  for (let i = 0; i < a.length; i++) {
    result |= a[i] ^ b[i];
  }
  
  return result === 0;
}
```

### **14.3 Audit Logging**

**MUST log:**
- Authentication attempts (success/failure)
- Capability invocations
- Protocol installations
- Key rotations
- Sync operations
- Admin actions

**Log Format:**
```json
{
  "timestamp": "2025-10-24T15:30:00Z",
  "eventType": "capability.invoked",
  "actor": "did:example:bob",
  "target": "did:example:alice",
  "capability": "urn:uuid:cap-123",
  "action": "read",
  "resource": "bafyrei...",
  "result": "success",
  "metadata": {
    "transport": "https",
    "ipAddress": "192.0.2.1"
  }
}
```

---

## **15. Privacy Considerations**

### **15.1 Data Minimization**

**Implementations SHOULD:**
- Only store necessary metadata
- Use encrypted indexes to minimize plaintext exposure
- Provide selective disclosure mechanisms
- Allow users to delete data permanently

### **15.2 Selective Disclosure**

Using Verifiable Credentials with BBS+ signatures:

```json
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiableCredential"],
  "credentialSubject": {
    "id": "did:example:alice",
    "age": 30,
    "citizenship": "US",
    "medicalLicense": "12345"
  },
  "proof": {
    "type": "BbsBlsSignature2020",
    // Allows selective disclosure of individual fields
  }
}
```

### **15.3 Metadata Leakage**

**Potential Leakage:**
- Record sizes (even when encrypted)
- Access patterns
- Sync timing
- Query patterns

**Mitigations:**
- Pad encrypted data to fixed sizes
- Add dummy queries
- Use constant-time operations
- Batch operations when possible

### **15.4 Compliance**

#### **15.4.1 GDPR Considerations**

**Right to Access:**
- Provide export functionality
- Support data portability

**Right to Erasure:**
- Implement permanent deletion (prune mode)
- Purge from all sync nodes
- Remove from backups

**Right to Rectification:**
- Support record updates
- Maintain version history if required

#### **15.4.2 HIPAA Considerations**

**Administrative Safeguards:**
- Access controls via ZCAP
- Audit logging
- User authentication

**Physical Safeguards:**
- Encrypt data at rest
- Secure data centers

**Technical Safeguards:**
- Encryption in transit (TLS)
- Audit controls
- Transmission security

#### **15.4.3 Geographic Data Residency**

Using selective sync with regional shards:

```json
{
  "selectiveSync": {
    "shards": [
      {
        "name": "eu-healthcare",
        "protocols": ["https://health.example/medical"],
        "nodes": ["node-eu-west"],
        "compliance": "GDPR",
        "region": "EU"
      }
    ]
  }
}
```

---

## **16. Implementation Guidance**

### **16.1 Reference Architecture**

```
┌─────────────────────────────────────────────────────┐
│              Application Layer                       │
│  (Business Logic, UI, Application-Specific Protocols)│
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│             Client SDK Layer                         │
│  (Message Construction, Capability Management,       │
│   Encryption/Decryption, Query Building)             │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│           Transport Adapter Layer                    │
│  (HTTP, gRPC, DIDComm, WebSocket)                   │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ Network
                  │
┌─────────────────▼───────────────────────────────────┐
│            DWN Server Layer                          │
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │         Message Router                       │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │    Authorization Engine (ZCAP Verification)  │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │      Protocol Rule Processor                 │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │    Encryption/Decryption Service            │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │       Storage Engine (IPLD/DAG)              │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │       Index Manager (Blind Indexes)          │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │        Sync Engine (CRDT)                    │  │
│  └──────────┬───────────────────────────────────┘  │
│             │                                       │
│  ┌──────────▼───────────────────────────────────┐  │
│  │    Notification Manager                      │  │
│  └──────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│           Persistence Layer                          │
│  (Database, Key-Value Store, File System)           │
└──────────────────────────────────────────────────────┘
```

### **16.2 Suggested Technology Stack**

**Server:**
- **Language**: TypeScript/Node.js, Go, or Rust
- **Database**: PostgreSQL (metadata), LevelDB/RocksDB (IPLD blocks)
- **Cache**: Redis
- **Message Queue**: RabbitMQ or Kafka (for async processing)
- **Key Store**: HashiCorp Vault or AWS KMS

**Client SDK:**
- **Language**: TypeScript (web), Swift (iOS), Kotlin (Android)
- **Storage**: IndexedDB (web), SQLite (mobile)
- **Crypto**: WebCrypto API (web), native crypto (mobile)

### **16.3 Performance Optimization**

#### **16.3.1 Caching Strategy**

```typescript
interface CacheConfig {
  recordCache: {
    enabled: true,
    ttl: 300,  // 5 minutes
    maxSize: 10000
  },
  protocolCache: {
    enabled: true,
    ttl: 3600,  // 1 hour
    maxSize: 100
  },
  capabilityCache: {
    enabled: true,
    ttl: 60,  // 1 minute
    maxSize: 1000
  }
}
```

#### **16.3.2 Database Indexes**

**Metadata Table Indexes:**
```sql
CREATE INDEX idx_protocol ON records(protocol);
CREATE INDEX idx_schema ON records(schema);
CREATE INDEX idx_context_id ON records(context_id);
CREATE INDEX idx_date_created ON records(date_created);
CREATE INDEX idx_tenant_did ON records(tenant_did);
CREATE INDEX idx_blind_indexes ON records USING GIN(blind_indexes);
```

#### **16.3.3 Query Optimization**

**Pagination:**
- Use cursor-based pagination instead of offset
- Index on sort fields

**Blind Indexes:**
- Store in separate table for faster lookup
- Use bloom filters for initial filtering

**Graph Queries:**
- Limit traversal depth
- Implement query timeout

### **16.4 Scalability**

#### **16.4.1 Horizontal Scaling**

**Stateless DWN Nodes:**
- Load balance across multiple instances
- Shared database backend
- Distributed cache

**Shard-Based Distribution:**
- Partition by protocol
- Partition by DID (tenant)
- Geographic partitioning

#### **16.4.2 Vertical Scaling**

**Resource Limits:**
```json
{
  "limits": {
    "maxRecordSize": 10485760,      // 10 MB
    "maxBatchSize": 100,
    "maxQueryResults": 1000,
    "maxSubscriptions": 100,
    "maxConcurrentConns": 10000
  }
}
```

### **16.5 Monitoring and Observability**

**Metrics to Track:**
- Request rate (per transport, per tenant)
- Error rate (by error type)
- Response latency (P50, P95, P99)
- Sync lag (time behind primary node)
- Encryption/decryption time
- Storage usage (per tenant)
- Capability invocation rate

**Instrumentation:**
```typescript
import { metrics } from './monitoring';

async function handleMessage(msg: Message): Promise<Result> {
  const startTime = Date.now();
  
  try {
    const result = await processMessage(msg);
    
    metrics.increment('messages.processed', {
      interface: msg.descriptor.interface,
      method: msg.descriptor.method,
      status: 'success'
    });
    
    metrics.timing('messages.latency', Date.now() - startTime);
    
    return result;
  } catch (error) {
    metrics.increment('messages.errors', {
      errorType: error.type
    });
    throw error;
  }
}
```

---

## **17. References**

### **17.1 Normative References**

- **[RFC2119]** Key words for use in RFCs to Indicate Requirement Levels
- **[DID-CORE]** Decentralized Identifiers (DIDs) v1.0
- **[VC-DATA-MODEL]** Verifiable Credentials Data Model v1.1
- **[ZCAP-LD]** Authorization Capabilities for Linked Data
- **[IPLD]** InterPlanetary Linked Data
- **[CBOR]** Concise Binary Object Representation
- **[JWS]** JSON Web Signature (RFC 7515)
- **[JWK]** JSON Web Key (RFC 7517)
- **[HKDF]** HMAC-based Extract-and-Expand Key Derivation Function (RFC 5869)
- **[TLS]** The Transport Layer Security (TLS) Protocol Version 1.3 (RFC 8446)
- **[HTTP]** HTTP Semantics (RFC 9110)

### **17.2 Informative References**

- **[DWN]** Decentralized Web Node Specification (DIF)
- **[SOLID-PROTOCOL]** Solid Protocol (W3C CG)
- **[CRDT]** Conflict-free Replicated Data Types
- **[DIDCOMM]** DIDComm Messaging Specification
- **[SPARQL]** SPARQL 1.1 Query Language
- **[SHAMIR]** Shamir's Secret Sharing
- **[BBS+]** BBS+ Signatures

---

## **18. Appendices**

### **Appendix A: Complete Protocol Definition Example**

```json
{
  "protocol": "https://example.com/collaborative-document",
  "version": "1.0.0",
  "extends": null,
  "published": true,
  "description": "A protocol for collaborative document editing",
  
  "types": {
    "document": {
      "schema": "https://example.com/schemas/document",
      "dataFormats": ["application/json"]
    },
    "edit": {
      "schema": "https://example.com/schemas/edit",
      "dataFormats": ["application/json"]
    },
    "comment": {
      "schema": "https://example.com/schemas/comment",
      "dataFormats": ["application/json"]
    },
    "collaborator": {
      "schema": "https://example.com/schemas/collaborator",
      "dataFormats": ["application/json"]
    }
  },
  
  "structure": {
    "document": {
      "$size": { "max": 10000000 },
      "$actions": [{
        "who": "author",
        "of": "document",
        "can": ["read", "update", "delete"]
      }],
      "$encryption": {
        "rootKeyId": "did:example:alice#key-agreement-1",
        "derivationScheme": "protocolContext"
      },
      
      "collaborator": {
        "$role": true,
        "$size": { "max": 1000 },
        "$actions": [{
          "who": "author",
          "of": "document",
          "can": ["create", "delete"]
        }]
      },
      
      "edit": {
        "$size": { "max": 100000 },
        "$actions": [{
          "role": "collaborator",
          "can": ["create", "read"]
        }, {
          "who": "author",
          "of": "edit",
          "can": ["update", "delete"]
        }]
      },
      
      "comment": {
        "$size": { "max": 10000 },
        "$actions": [{
          "role": "collaborator",
          "can": ["create", "read"]
        }, {
          "who": "author",
          "of": "comment",
          "can": ["update", "delete"]
        }]
      }
    }
  },
  
  "conflictResolution": {
    "strategy": "crdt",
    "crdtType": "json-crdt",
    "fallback": "manual"
  },
  
  "indexes": {
    "documentsByAuthor": {
      "type": "blind-index",
      "fields": ["author", "dateCreated"],
      "saltPerRecord": true
    },
    "documentsByTitle": {
      "type": "blind-index",
      "fields": ["title"],
      "saltPerRecord": false
    }
  }
}
```

### **Appendix B: Complete Message Flow Example**

**Scenario:** Alice creates a post, Bob comments on it.

**Step 1: Alice creates post**
```json
{
  "recordId": "bafyrei_post_abc",
  "contextId": "social-thread-123",
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "messageTimestamp": "2025-10-24T15:00:00Z",
    "protocol": "https://example.com/social",
    "protocolVersion": "1.0.0",
    "schema": "https://schema.org/SocialMediaPosting",
    "dataFormat": "application/json",
    "dataCid": "bafyrei_data_abc",
    "dateCreated": "2025-10-24T15:00:00Z",
    "published": true
  },
  "data": {
    "text": "Hello, decentralized world!",
    "author": "did:example:alice"
  },
  "authorization": {
    "payload": {
      "descriptorCid": "bafyrei_desc_abc",
      "capability": "urn:uuid:alice-root-cap"
    },
    "signatures": [{
      "protected": {
        "alg": "EdDSA",
        "kid": "did:example:alice#key-1"
      },
      "signature": "..."
    }]
  }
}
```

**Step 2: Alice grants Bob comment capability**
```json
{
  "@context": ["https://w3id.org/zcap/v1"],
  "id": "urn:uuid:bob-comment-cap",
  "type": ["VerifiableCredential", "Authorization"],
  "issuer": "did:example:alice",
  "issuanceDate": "2025-10-24T15:05:00Z",
  "expirationDate": "2026-10-24T15:05:00Z",
  "credentialSubject": {
    "id": "did:example:bob",
    "parentCapability": "urn:uuid:alice-root-cap",
    "invocationTarget": {
      "id": "did:example:alice",
      "type": "DWNNode",
      "allowedActions": ["create"],
      "constraints": {
        "protocol": "https://example.com/social",
        "schema": "https://schema.org/Comment",
        "contextId": "social-thread-123",
        "parentId": "bafyrei_post_abc"
      }
    }
  },
  "proof": {
    "type": "JsonWebSignature2020",
    "created": "2025-10-24T15:05:00Z",
    "verificationMethod": "did:example:alice#key-1",
    "proofPurpose": "capabilityDelegation",
    "jws": "..."
  }
}
```

**Step 3: Bob creates comment**
```json
{
  "recordId": "bafyrei_comment_xyz",
  "contextId": "social-thread-123",
  "descriptor": {
    "interface": "Records",
    "method": "Write",
    "messageTimestamp": "2025-10-24T15:10:00Z",
    "protocol": "https://example.com/social",
    "protocolVersion": "1.0.0",
    "schema": "https://schema.org/Comment",
    "dataFormat": "application/json",
    "dataCid": "bafyrei_data_xyz",
    "dateCreated": "2025-10-24T15:10:00Z",
    "parentId": "bafyrei_post_abc",
    "published": true
  },
  "data": {
    "text": "Great post, Alice!",
    "author": "did:example:bob"
  },
  "authorization": {
    "payload": {
      "descriptorCid": "bafyrei_desc_xyz",
      "capability": "urn:uuid:bob-comment-cap",
      "capabilityChain": [
        { /* Alice's root capability */ },
        { /* Bob's comment capability */ }
      ]
    },
    "signatures": [{
      "protected": {
        "alg": "EdDSA",
        "kid": "did:example:bob#key-1"
      },
      "signature": "..."
    }]
  }
}
```

### **Appendix C: Version History**

**Version 0.1.0-draft (Current)**
- Initial specification
- DID-first identity and discovery
- Transport-agnostic design
- ZCAP-LD authorization
- Message-level encryption with hierarchical key derivation
- CRDT-based synchronization
- Blind index searchable encryption
- Social key recovery
- Multi-transport support (HTTP, gRPC, DIDComm, WebSocket)
- Subscription system with persistence
- Comprehensive error handling

---

## **Document Maintenance**

**Change Process:**
1. Submit issues/PRs to specification repository
2. Discuss in working group meetings
3. Reach consensus on changes
4. Update version number (SemVer)
5. Publish new version

**Version Numbering:**
- **Major** (X.0.0): Breaking changes
- **Minor** (0.X.0): New features, backward compatible
- **Patch** (0.0.X): Bug fixes, clarifications

**Current Status:** DRAFT
**Next Milestone:** Version 0.2.0 with implementation feedback

---

**END OF SPECIFICATION**

This specification is maintained at: [TBD GitHub Repository]  
For questions and discussion: [TBD Discussion Forum]  
For implementation support: [TBD Developer Community]
