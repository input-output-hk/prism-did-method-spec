# DID Method Specification - did:prism

## Status of this document

This document describes the first version of the `prism` DID method.
The method is in constant evolution, and future versions are under development.
It is recommended to any implementer to follow the evolution of the new versions closely 
to understand, comment on, and/or suggest future changes.

## Abstract

The `prism` DID method defines data models and protocol rules to create, manage, and resolve Decentralized Identifiers (DIDs). The protocol is defined on top of the Cardano blockchain as its verifiable data registry, where DID's information is stored.

The method is defined as a protocol, that describes operations, serialization formats, and rules. The protocol describes how to manage the lifecycle of DIDs and their associated DID documents. We will use the term `PRISM node` to refer to software that implements the protocol defined in this document.

We would like to remark that, any reference to the blockchain such as, "Cardano network", "underlying chain", "on-chain", "ledger", "the blockchain" and similar ones throughout this document refer to **Cardano mainnet** unless explicitly said otherwise. The `prism` DID method, in its current form, solely depends on and uses Cardano mainnet.

## Versioning and protocol parameters

This document describes the first version of the `prism` DID method. Each version of the protocol defines certain parameters such as hashing algorithms, encoding functions, size limits for data, and others. Each version MAY make changes to these set of parameters. For this version, considered the first one, the protocol parameters are the following:


| Parameter | Description | Value |
| -------- | -------- | -------- |
| `SYSTEM_UPDATE_DID` | The DID that has the power to trigger a protocol update. | TBD`*` |
| `GENESIS_PRISM_BLOCK` | The block number from which the `prism` DID method begins reading the blockchain. | TBD`*` |
| `SECURE_DEPTH` | Number of confirmations a block needs to wait to not be rollbacked with high probability. | 112 blocks |
| `PROTOBUF_VERSION` | The version of Protocol Buffers used. | 3 |
| `CRYPTOGRAPHIC_CURVE` | Cryptographic curve used with a key. | secp256k1 |
| `SIGNATURE_ALGORITHM` | Asymmetric public key signature algorithm. | SHA256 with ECDSA |
| `HASH_ALGORITHM` | Algorithm for generating hashes. | SHA-256 |
| `SHORT_ENCODING_ALGORITHM` | Encoding selected for various data (signatures, hashes, etc.). | HEX |
| `LONG_ENCODING_ALGORITHM` | Encoding selected for Protobuf binary. | base64URL |
| `MAX_ID_SIZE` | Maximum number of characters for an `id` field value. | 50 |
| `MAX_TYPE_SIZE` | Maximum number of characters for a `type` field value. | 100 |
| `MAX_SERVICE_ENDPOINT_SIZE` | Maximum number of characters for a `serviceEndpoint` field value. | 300 |
| `MAX_SERVICE_NUMBER` | Maximum number of active services a DID Document can have at the same time. | 50 |
| `MAX_VERIFICATION_METHOD_NUMBER` | Maximum number of active keys a DID can have at the same time. Note that this includes keys with `usage` `MASTER_KEY` and `REVOCATION_KEY` | 50 |
| `SECP256K1_CURVE_NAME` | String identifier for the SECP256K1 eliptic curve | "secp256k1" |
| `ED25519_CURVE_NAME` | String identifier for the ED25519 eliptic curve | "Ed25519" |
| `X25519_CURVE_NAME` | String identifier for the Curve25519 eliptic curve | "X25519" |

`*` These values will be filled in once the mainnet deployment is confirmed.

## DID Method Name

The namestring that shall identify this DID `method-name` is: `prism`.

A DID that uses this method MUST begin with the following prefix `did:prism`. The prefix MUST be in lowercase.

The remainder of the DID is specified below.

### Method Specific Identifier

### Prism DID Method Syntax
```abnf
prism-did          = "did:prism:" initial-hash [encoded-state]
initial-hash       = 64HEXDIGIT 
encoded-state      = ":" 1*id-char
id-char            = ALPHA / DIGIT / "-" / "_" 
```
### Examples of `did:prism` Identifiers

The method defines two types of DIDs. The first one represents the DIDs that have been anchored on the blockchain following the protocol rules:
```abnf
did:prism:9b5118411248d9663b6ab15128fba8106511230ff654e7514cdcc4ce919bde9b
```
The DID method specific identifier corresponds to the hex-encoded hash of the initial state of the DID document.

The protocol also allows to create DIDs without interaction with the blockchain:
```abnf
did:prism:9b5118411248d9663b6ab15128fba8106511230ff654e7514cdcc4ce919bde9b:Cj8KPRI7CgdtYXN0ZXIwEAFKLgoJc2VjcDI1NmsxEiEDHpf-yhIns-LP3tLvA8icC5FJ1ZlBwbllPtIdNZ3q0jU
```
For this case, the DID adds after the hash section, the actual encoded initial state.

In the following sections we will describe more details about the state mentioned. 

## DID Documents

The `prism` DID method allows to create fairly expressive DID documents. In this first version, the method supports the creation of DID documents that contain arbitrary services. With respect to verification methods, all W3C verification relationships are supported (namely, authentication, key agreement, assertion, capability invocation and capability delegation). The supported verification method type is `JsonWebKey2020` where keys are expressed as JWK. The supported keys however are restricted to `ED25519_CURVE_NAME`, `SECP256K1_CURVE_NAME` and `X25519_CURVE_NAME`. In future versions, more verification method types and keys will be supported.


### EXAMPLE: DID document (JSON-LD)

```json
{
    "@context": [
      "https://www.w3.org/ns/did/v1",
      "https://w3id.org/security/suites/jws-2020/v1",
      "https://didcomm.org/messaging/contexts/v2",
      "https://identity.foundation/.well-known/did-configuration/v1"
    ],
    "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290",
    "verificationMethod": [
      {
        "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#authentication0",
        "type": "JsonWebKey2020",
        "controller": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290",
        "publicKeyJwk": {
            "crv": "secp256k1",
            "kty": "EC",
            "x": "KDiRms8trMvdT4aaJbRtNxtDCYTvFJbstRrC3TgQNnc",
            "y": "AUwPe-RV4D9c9WbVKHeg7-imv2ZVDX8hDAEkMDphpqU"
         }
      },
      {
        "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#issuing0",
        "type": "JsonWebKey2020",
        "controller": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290",
        "publicKeyJwk" : {
          "kty" : "OKP",
          "crv" : "Ed25519",
          "x" : "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo"
        }
      },
      {
        "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#key-agreement0",
        "type": "JsonWebKey2020",
        "controller": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290",
        "publicKeyJwk": {
          "kty": "OKP",
          "crv": "X25519", 
          "x": "pE_mG098rdQjY3MKK2D5SUQ6ZOEW3a6Z6T7Z4SgnzCE"
        }
      }
    ],
    "authentication": [
      "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#authentication0"
    ],
    "assertionMethod": [
      "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#issuing0"
    ],
    "keyAgreement": [
      "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#key-agreement0"
    ],
    "service": [
      {
        "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#DIDCommMessaging",
        "type": "DIDCommMessaging",
        "serviceEndpoint": [ 
          { 
            "uri": "https://example.com/path", 
            "accept": [ "didcomm/v2", "didcomm/aip2;env=rfc587" ], 
            "routingKeys": ["did:example:somemediator#somekey"] 
          }
        ]
      },
      {
        "id" : "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#linked-domain-1",
        "type": "LinkedDomains",
        "serviceEndpoint": {
          "origins": ["https://foo.example.com", "https://identity.foundation"]
        }
      },
      {
        "id": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290#linked-domain-2",
        "type": "LinkedDomains",
        "serviceEndpoint": "https://bar.example.com"
      }
    ]
  }
```

## High level protocol description

At a high level, the protocol that defines the `prism` DID method works as follows:
- Any user can run a `PRISM node`, to self validate information; or can rely on a set of actors that run nodes on his behalf. The level of delegation of trust is a decision made by each user.
- Any user willing to create a DID can do so without any need to interact with any `PRISM node`. The creation of a DID can be optionally announced publicly by publishing a creation operation on-chain. The action of posting an operation on-chain does require the interaction with a `PRISM node`
- Users can update the DID documents associated to their DIDs. To do this, they need to publish respective update operations on-chain. This again requires interaction with a `PRISM node`
- Deactivation of a DID can be performed in the same lines of updates, but publishing a deactivation operation
- `PRISM nodes` read the operations published on-chain, and maintain internally the map of DIDs to the history of changes of their associated DID documents.
- Any client can query any `PRISM node` and obtain information (state) associated to a DID
- DID resolvers, can take the output of `PRISM nodes` and construct the current DID document associated to a DID

An additional consideration is that operations can be posted on-chain in blocks, helping on the scalability side and general reduction of fees.

In the following sections we will describe in detail the format of operations and how to validate them.

## DID operations construction

The `prism` method represents DID operations such as creation, updates and deactivations using [Protocol Buffers](https://developers.google.com/protocol-buffers).
Each operation is defined as a message, which we will describe in the next sub-sections.

At a high level, operations represent actions upon DIDs (i.e. Create, Update, Deactivate). Operations are batched in `blocks`, and blocks are wrapped in `objects`. Each object is attached to a transaction's metadata in the Cardano network. 
Users construct these operations and submit them to the blockchain. Conversely `PRISM nodes` read Cardano transactions, examine their metadata, they extract (when present) encoded objects. The objects contains blocks, and the block contains a sequence of DID operations. `PRISM nodes` validate and interpret these operations, which allow them to build their shared global view of the map of DIDs to DID documents.

Below, we present the main protobuf messages mentioned before. For purpose of making the reading simple, we moved some protobuf definitions to an Appendix at the end of this specification. We will describe for each operation how users must construct them to be considered valid.


### Block and object definitions 

We first show the way to represent `blocks` of operations.

```protobuf
// Represent a block that holds operations.
message AtalaBlock {
  reserved 1; // Represents the version of the block. Deprecated
  repeated SignedAtalaOperation operations = 2; // A signed operation, necessary to post anything on the blockchain.
}
```

As we mentioned before, `blocks` are wrapped inside `objects`, which will add some information about them in the planned future:

```protobuf
// Wraps an AtalaBlock and its metadata.
message AtalaObject {
  reserved 1, 2, 3; // Removed block_hash field.
  reserved "block_hash";
  reserved "block_operation_count";
  reserved "block_byte_length";

  AtalaBlock block_content = 4; // The block content.
}
```

When a user wants to perform DID operations, he:
- Puts them in the desired order inside an `AtalaBlock`, 
- wraps the block in an `AtalaObject`.
- Encodes the `AtalaObject` inside the metadata of a Cardano transaction (see Appendix A for encoding details)
- Sends the transaction with metadata to the Cardano blockchain

### Signed operation definition 

The different DID operations are signed (as we will describe in next sections), the message that represents signed operations is:

```protobuf
// A signed operation, necessary to post anything on the blockchain.
message SignedAtalaOperation {
    string signed_with = 1; // The key ID used to sign the operation, it must belong to the DID that signs the operation.
    bytes signature = 2; // The actual signature.
    AtalaOperation operation = 3; // The operation that was signed.
}

// The possible operations affecting the blockchain.
message AtalaOperation 
    reserved 3, 4; // fields used by an extension of the protocol. Not relevant for the DID method
    // The actual operation.
    oneof operation {
        // Used to create DIDs.
        CreateDIDOperation create_did = 1;
        // Used to update an existing public DID.
        UpdateDIDOperation update_did = 2;
        // Used to announce new protocol update
        ProtocolVersionUpdateOperation protocol_version_update = 5;
        // Used to deactivate DID
        DeactivateDIDOperation deactivate_did = 6;
    };
}
```
**Construction rules**
- all fields MUST be present.
- The `operation` field MUST contain one of `CreateDIDOperation`, `UpdateDIDOperation` ,`DeactivateDIDOperation`, `ProtocolVersionUpdateOperation` which MUST be constructed correctly according to rules defined in the next sections of this document.

Lets move to the specific description of each operation

### Create DID

In order to create a DID, the controller will construct a `CreateDIDOperation` message using the following definitions.

```protobuf
// The operation to create a public DID.
message CreateDIDOperation {
    DIDCreationData did_data = 1; // DIDCreationData with public keys and services

    // The data necessary to create a DID.
    message DIDCreationData {
        reserved 1; // Removed DID id field which is empty on creation
        repeated PublicKey public_keys = 2; // The keys that belong to this DID Document.
        repeated Service services = 3; // The list of services that belong to this DID Document.
        repeated string context = 4; // The list of @context values to consider on JSON-LD representations
    }
}

// Represents a public key with metadata, necessary for a DID document.
message PublicKey {
    reserved 3, 4, 5, 6;
    string id = 1; // The key identifier within the DID Document.
    KeyUsage usage = 2; // The key's purpose.

    // The key's representation.
    oneof key_data {
        ECKeyData ec_key_data = 8; // The Elliptic Curve (EC) key.
        CompressedECKeyData compressed_ec_key_data =  9; // Compressed Elliptic Curve (EC) key.
    };
}

// Every key has a single purpose:
enum KeyUsage {
    // UNKNOWN_KEY is an invalid value - Protobuf uses 0 if no value is provided and we want the user to explicitly choose the usage.
    UNKNOWN_KEY = 0;
    MASTER_KEY = 1;
    ISSUING_KEY = 2;
    KEY_AGREEMENT_KEY = 3;
    AUTHENTICATION_KEY = 4;
    REVOCATION_KEY = 5;
    CAPABILITY_INVOCATION_KEY = 6;
    CAPABILITY_DELEGATION_KEY = 7;
}

// Holds the necessary data to recover an Elliptic Curve (EC)'s public key.
message ECKeyData {
    string curve = 1; // The curve name, e.g. secp256k1.
    bytes x = 2; // The x coordinate, represented as bytes.
    bytes y = 3; // The y coordinate, represented as bytes.
}

// Holds the compressed representation of data needed to recover Elliptic Curve (EC)'s public key.
message CompressedECKeyData {
    string curve = 1; // The curve name, e.g. secp256k1.
    bytes data = 2; // compressed Elliptic Curve (EC) public key.
}

message Service {
    string id = 1;
    string type = 2;
    string service_endpoint = 3;
}
```

**Construction rules**

Below we see the rules to construct a well-formed `CreateDIDOperation`, these rules MUST be followed while creating the operation, and will be checked by `PRISM nodes`. Any construction error will make nodes ignore the operation.

- The `did_data` field of the `CreateDIDOperation` MUST not be empty and MUST be properly constructed
- Each element of `public_keys` and `services` fields MUST be constructed correctly
- `services` can be empty. The list MUST not exceed `MAX_SERVICE_NUMBER` elements.
- For each `Service` message in the list:
    - The `type` field MUST be a string or a non empty JSON array of strings. 
      - The `type` value MUST not start nor end with whitespaces, and MUST have at least a non whitespace character. 
      - The `type` value MUST not exceed `MAX_TYPE_SIZE` characters in length.
      - The strings constructing the value in `type` SHOULD be registered in the DID Specification Registries [DID-SPEC-REGISTRIES](https://www.w3.org/TR/did-spec-registries/)
      - To be more precise, the `type` value MUST conform the following ABNF rule
            
              type-value         = type-string / "[" *(quoted-type-string ",") quoted-type-string "]"
              quoted-type-string = DQUOTE type-string DQUOTE
              type-string        = type-char *(*SP type-char)
              type-char          = ALPHA / DIGIT / "-" / "_"
    - The `id` field MUST be a valid [fragment](https://www.rfc-editor.org/rfc/rfc3986#section-3.5) string in accordance to [DID URL syntax](https://www.w3.org/TR/did-core/#did-url-syntax). The `id` MUST not exceed `MAX_ID_SIZE` characters in length.
    - The `service_endpoint` field MUST contain one of:
       - a URI 
       - a JSON object
       - a non-empty JSON array of URIs and/or JSON objects
       - all mentioned URIs MUST conform to [RFC3986](https://www.rfc-editor.org/rfc/rfc3986) and normalized according to the [Normalization and Comparison rules in RFC3986 and to any normalization](https://www.rfc-editor.org/rfc/rfc3986#section-6) rules in its applicable URI scheme specification
    - The `service_endpoint` value MUST not exceed `MAX_SERVICE_ENDPOINT_SIZE` characters in length
- The `public_keys` field MUST contain at least one `PublicKey` message for which its `usage` field MUST be `MASTER_KEY`. 
- `public_keys` MUST not exceed `MAX_VERIFICATION_METHOD_NUMBER` elements.
- For each `PublicKey` message:
    - The `id` field MUST follow the same rules as `id`s for `Service`s
    - The `usage` field MUST not be `UNKNOWN_KEY`
    - The remaining key usages are:
        - `MASTER_KEY`: Used to sign `AtalaOperation`s. It won't appear in DID documents during resolution
        - `ISSUING_KEY`: equivalent to `assertion` verification relationship
        - `REVOCATION_KEY`: Used for related protocol outside of this spec. It will not appear in the DID documents.
        - `KEY_AGREEMENT_KEY`, `AUTHENTICATION_KEY`, `CAPABILITY_INVOCATION_KEY` and `CAPABILITY_DELEGATION_KEY`: represent their mirror verification relationship according to DID core specification.
    - `key_data` field MUST not be empty
        - If it contains an `ECKeyData`
            - The `curve` MUST be one of `SECP256K1_CURVE_NAME`, `ED25519_CURVE_NAME` or `X25519_CURVE_NAME`. When the `usage` is `MASTER_KEY` the `curve` MUST be `SECP256K1_CURVE_NAME`.
            - The `x` MUST contain a valid `x` coordinate of the public key
            - The `y` MUST contain a valid `y` coordinate of the public key
        - If it contains a `CompressedECKeyData` 
            - The `curve` MUST be one of `SECP256K1_CURVE_NAME`, `ED25519_CURVE_NAME` or `X25519_CURVE_NAME`. When the `usage` is `MASTER_KEY` the `curve` MUST be `SECP256K1_CURVE_NAME`.
            - The `x` MUST contain a valid compressed representation of the public key in accordance to the `curve` value
- `context` field MUST contain a list of strings. Each string represents additional `@context` values that the resolver will use to produce JSON-LD output during DID Document generation. The list MUST NOT contain repeated values.

**Signing and submission**
1. Once the controller created the `CreateDIDOperation`, he can construct an `AtalaOperation` and sign it using the `SIGNATURE_ALGORITHM`. With that, construct a `SignedAtalaOperation` that contains the generated signature in the `signature` field; the `AtalaOperation` in the `operation` field, and the key identifier of the master key used to generate this signature in `signed_with`. Note that the `usage` of the key used to sign the operation MUST be `MASTER_KEY`
2. With the `SignedAtalaOperation` the user can decide to create an `AtalaBlock` and `AtalaObject` messages, and submit a transaction himself containing the operation. Alternatively, he can gather more operations before submitting a transaction in order to save fees.
3. Once the transaction containing the operation has a `SECURE_DEPTH` in the blockchain, then the operation will be processed by `PRISM nodes`.
4. The DID generated will have `did:prism:` as prefix, the method specific identifier is produced by applying the `SHORT_ENCODING_ALGORITHM` to the hash produced by the `HASHING_ALGORITHM` applied to the binary representation of the `AtalaOperation` that was signed. We call these DIDs, short form DIDs.

#### Long form DIDs (unpublished DIDs)

The `prism` DID method also supports the creation of DIDs without the need of publishing operations on-chain.
To do so:
- create the `CreateDIDOperation` as described in the previous section. 
- Then, compute the DID as described in step 4. 
- Finally, use the `LONG_ENCODING_ALGORITHM` on the binary representation of the `AtalaOperation` that contains the `CreateDIDOperation` built before, and append this string with an starting `":"` at the end of the original DID from step 4.

This DID, called Long Form DID, can later be published (by posting the corresponding `CreateDIDOperation` as described in the previous section), and updated by the controller. In the meantime, the user will be able to use it without submitting anything to the blockchain.

Note that each long form DID in the `prism` DID method has a unique short form associated DID. We can validate the correspondence by computing the hash of the encoded `AtalaOperation` in the long form DID, and checking its equality with the last section of the short form DID.
Also note, that a short form DID is a prefix of its corresponding long form DID. 


### Read DID

In this section we will describe the read process at a high level. For more details, refer to the section that translate internal state to DID documents later in this document.

`PRISM nodes` are constantly reading the underlying blockchain in search of transactions that contain `AtalaObjects` in their metadata. These nodes read the operations extracted from the objects, validate them and construct a map of DIDs to their corresponding DID documents. The rules by which `PRISM nodes` do this are described later in this text.

When a `PRISM node` receives a DID for resolution, it first checks if it is a short or long form DID.

If it is a short form:
* It checks if there is information about its corresponding DID document on its database. 
  * If there is, it returns the latest state known to it. 
  * If there is no information, the resolution returns the `notFound` error

If the DID to resolve is in long form:
* It extracts the short form DID from the DID
* It validates the short form DID correspondence (i.e. the encoded operation must match the expected hash), and validates that the decoded operation is a well constructed `AtalaOperation` that contains a `CreateDIDOperation` 
  * If the validations fail, it returns the `invalidDid` error.
* It then checks if there is information about the DID document corresponding to the short DID on its database. 
  * If there is, it returns the data on its database.
  * If there is no information, it decodes the `AtalaOperation` from the long form DID, and returns the corresponding DID document. See next sections to find the translation between protobuf models and DID documents as well as DID document metadata.


### Update DID 

In order to update the DID document associated with a DID, the initial `CreateDIDOperation` must be already published. For the case of long form `prism` DIDs, the user can first create the corresponding `SignedAtalaOperation` for the DID creation; then, the intended update operation and finally submit both corresponding signed operations in a single transaction by gathering the two in the same `AtalaBlock`.

The model for `UpdateDIDOperation`s looks as follows:

```protobuf
// Specifies the necessary data to update a public DID.
message UpdateDIDOperation {
    bytes previous_operation_hash = 1; // The hash of the most recent operation that was used to create or update the DID.
    string id = 2; 
    repeated UpdateDIDAction actions = 3; // The actual updates to perform on the DID.
}
```

where

```protobuf
// The potential details that can be updated in a DID.
message UpdateDIDAction {

    // The action to perform.
    oneof action {
        AddKeyAction add_key = 1; // Used to add a new key to the DID.
        RemoveKeyAction remove_key = 2; // Used to remove a key from the DID.
        AddServiceAction add_service = 3; // Used to add a new service to a DID,
        RemoveServiceAction remove_service = 4; // Used to remove an existing service from a DID,
        UpdateServiceAction update_service = 5; // Used to Update a list of service endpoints of a given service on a given DID.
        PatchContextAction patch_context = 6; // Used to Update a list of `@context` strings used during resolution for a given DID.
    }
}

// The necessary data to add a key to a DID.
message AddKeyAction {
    PublicKey key = 1; // The key to include.
}

// The necessary data to remove a key from a DID.
message RemoveKeyAction {
    string keyId = 1; // the key id to remove
}

message AddServiceAction {
    Service service = 1;
}

message RemoveServiceAction {
    string serviceId = 1;
}

message UpdateServiceAction {
    string serviceId = 1; // scoped to the did, unique per did
    string type = 2; // new type if provided
    // Will replace all existing service endpoints of the service with provided ones
    string service_endpoints = 3;
}

message PatchContextAction {
    repeated string context = 1; // The list of strings to use by resolvers during resolution when producing a JSON-LD output
}
```

**Construction rules**
* The `previous_operation_hash` field MUST contain the hash of the last `AtalaOperation` message that was used to successfully update the DID. This is, the hash of the `AtalaOperation` that contained the last `UpdateDIDOperation` applied or, if this is the first update, the hash of the `AtalaOperation` that contained the `CreateDIDOperation` that created the DID. The hash is obtained through the `HASHING_ALGORITHM` of the said message.
* The `id` field MUST contain the DID method-specific identifier of the short form DID that the update operation intends to update.
- The `actions` field MUST contain a non empty list of well constructed `UpdateDIDAction`s
- If the action is:
    - `AddKeyAction`, the `key` field MUST not be empty, and it MUST contain a well formed `PublicKey` as described in previous sections
    - `RemoveKeyAction`, the `keyId` field MUST contain the `id` of the `PublicKey` to delete from the DID document
    - `AddServiceAction`, the `service` field MUST not be empty, and it MUST contain a well constructed `Service` message as described in previous sections
    - `RemoveServiceAction`, the `serviceId` field MUST contain the `id` of the service to remove from the DID document
    - `UpdateServiceAction`: 
        - the `serviceId` field MUST contain the `id` of the service to update
        - it MUST NOT be the case that both, the `type` field and `service_endpoint` field are empty. This is, at least one of this field MUST be correctly populated
        - the `type` field, if present, MUST follow the same rules of the `type` field of the `Service` message. The `type` represents the new type (if present) that will replace the existing one.
        - the `service_endpoints` field, if non empty, MUST conform the same rules as the `service_endpoint` field of the `Service` message
    - `PatchContextAction`, the `context` field MUST contain a list of strings. The list MUST not contain repeated elements

**Signing and submission**
1. Once the controller created the `UpdateDIDOperation`, he can construct an `AtalaOperation` and sign it using the `SIGNATURE_ALGORITHM`. With that, construct a `SignedAtalaOperation` that contains the generated signature in the `signature` field; the `AtalaOperation` in the `operation` field, and the key identifier of the master key used to generate this signature in `signed_with`
2. With the `SignedAtalaOperation` the user can decide to create an `AtalaBlock` and `AtalaObject` messages, and submit a transaction himself containing the operation. Alternatively, he can gather more operations before submitting a transaction in order to save fees.
3. Once the transaction containing the operation has a `SECURE_DEPTH` in the blockchain, then the operation will be processed by `PRISM nodes`.


### Deactivate DID

In order to deactivate a DID, the controller can build an instance of `DeactivateDIDOperation`

```protobuf
message DeactivateDIDOperation {
    bytes previous_operation_hash = 1; // The hash of the most recent operation that was used to create or update the DID.
    string id = 2; // DID method-specific identifier of the DID to be deactivated
}
```

**Construction rules**
* The `previous_operation_hash` field MUST conform to the same rules as the `previous_operation_hash` field in `UpdateDIDOperation`
* The `id` field MUST conform the same rules as the `id` field in `UpdateDIDOperation`

**Signing and submission**
1. Once the controller created the `DeactivateDIDOperation`, he can construct an `AtalaOperation` and sign it using the `SIGNATURE_ALGORITHM`. With that, construct a `SignedAtalaOperation` that contains the generated signature in the `signature` field; the `AtalaOperation` in the `operation` field, and the key identifier of the master key used to generate this signature in `signed_with`
2. With the `SignedAtalaOperation` the user can decide to create an `AtalaBlock` and `AtalaObject` messages, and submit a transaction himself containing the operation. Alternatively, he can gather more operations before submitting a transaction in order to save fees.
3. Once the transaction containing the operation has a `SECURE_DEPTH` in the blockchain, then the operation will be processed by `PRISM nodes`.

### Protocol Version Update

As a coordination mechanism, the protocol defines a message to trigger protocol updates. A protocol update triggers a version change of the protocol. Such change can lead to different protocol parameters, formats, or even modify operations.
The messages associated to a protocol update are:

```protobuf
// Specifies the protocol version update
message ProtocolVersionUpdateOperation {
    string proposer_did = 1; // The DID method-specific identifier that proposes the protocol update.
    ProtocolVersionInfo version = 2; // Information of the new version
}

message ProtocolVersionInfo {
    reserved 2, 3;
    string version_name = 1; // (optional) name of the version
    int32 effective_since = 4; // Cardano block number that tells since which block the update is enforced

    // New major and minor version to be announced,
    // If major value changes, the node MUST stop issuing and reading operations, and upgrade before `effective_since` because the new protocol version.
    // If minor value changes, the node can opt to not update. All events _published_ by this node would be also
    // understood by other nodes with the same major version. However, there may be new events that this node won't _read_
    ProtocolVersion protocol_version = 5;
}

message ProtocolVersion {
    // Represent the major version
    int32 major_version = 1;
    // Represent the minor version
    int32 minor_version = 2;
}
```

**Construction rules**
- The `proposer_did` MUST be the method specific identifier of the `SYSTEM_UPDATE_DID`
- The `version` MUST not be empty, and MUST contain a well constructed `ProtocolVersionInfo` message
- The `ProtocolVersionInfo` message:
    - `version_name` can be empty, and represents an optional name of the version
    - `effective_since` MUST be possitive, and describes which is the first Cardano block where the new version becomes effective, all operations found on-chain from that block (included) MUST follow the new protocol version.
    - The `ProtocolVersion` MUST contain positive integrs in `major_version` and `minor_version` fields
        - A change of the `major_version` implies that the protocol changes in a way in which current DID operations will become invalid. For example, the format of an operation changes
        - A change of the `minor_version` implies that the protocol changes in a way in which current DID operations will still be valid, but other new changes may not be properly processed by `PRISM nodes` running old versions of the protocol. For example, a new operation is added, or a new `KeyUsage` is supported. 

Note that `minor_version` changes inform node operators that they may be able to skip the update, as their nodes will still interpret what they need. 

**Signing and submission**
1. Once the administrator created the `ProtocolVersionUpdateOperation`, he can construct an `AtalaOperation` and sign it using the `SIGNATURE_ALGORITHM`. With that, construct a `SignedAtalaOperation` that contains the generated signature in the `signature` field; the `AtalaOperation` in the `operation` field, and the key identifier of the master key used to generate this signature in `signed_with`
2. With the `SignedAtalaOperation` the user can decide to create an `AtalaBlock` and `AtalaObject` messages, and submit a transaction himself containing the operation. Alternatively, he can gather more operations before submitting a transaction in order to save fees.
3. Once the transaction containing the operation has a `SECURE_DEPTH` in the blockchain, then the operation will be processed by `PRISM nodes`.


## Processing of operations 

Up until now, we have described the rules to construct operations. However, we haven't described how the operations are processed and interpreted by `PRISM nodes`.

We described that operations are grouped in `blocks`, which are wrapped in `objects`, and they are submitted as part of transactions metadata. In the Cardano blockchain, transactions are added to the chain in blocks. Each Cardano block is added on top of the previous block. A transaction is added to the blockchain when a block that contains it is added to the blockchain. We say that a block gets a confirmation when a new block is added to the chain on top of it. In the same lines, we say that a transaction added to the blockchain gets a confirmation, when the block that contains the transaction receives a confirmation. 

Due to the nature of blockchains, there is a probability for a block added to the chain to be rolled back, which removes the block from the chain. This probability decreases as the block receives confirmations on top of it (i.e. new blocks are added after it). It is because of this rollback probability that `PRISM nodes` only process Cardano transactions after they receive a `SECURE_DEPTH` of confirmations, making the probability of rollbacks negligible in practical terms. Therefore, the protocol assumes a rollback free environment.

Below, we describe how `PRISM nodes` process Cardano blocks, `AtalaObject`s, `AtalaBlock`s and operations.

### Processing of Cardano blocks

`PRISM nodes` read the Cardano blockchain from the `GENESIS_PRISM_BLOCK`. Once a Cardano block reaches a `SERCURE_DEPTH` of confirmations, `PRISM nodes` explore each transaction of the block in order looking for encoded `AtalaObject`s. The encoding format MUST follow the one described in Appendix A, and there MUST be only one `AtalaObject` encoded per transaction.

### Processing of Atala objects and blocks

As they evaluate confirmed transactions, nodes extract the `AtalaObject` messages from metadata (when found), and process them in the order they are found. The nodes MUST validate that the `AtalaBlock` inside it is not empty. If it is empty, the object is ignored.

Once the object is validated, each `PRISM node` will proceed to inspect the operations inside the `AtalaBlock`. Operations MUST be processed in the order they have in the `AtalaBlock`. If an operation fails the validation rules, then the operation MUST be ignored, and the nodes MUST proceed with the next operations (if any) in the `AtalaBlock`. 

### PRISM Nodes internal map

When `PRISM nodes` process operations, they update an internal map of information.
The map takes DIDs, and maps them to:
- a hash that represents the last operation that affected the DID
- a list of strings representing the context auxiliary information
- keys' information
- services' information

The keys and services information consist of:
- the list of keys associated to the DID.
    - Each key has additional timestamps and transaction ids (from the underlying Cardano blockchain) that describe when the keys were added or deleted.
- the services associated to the DID. For each service we also have
    - the list of `type`s values with timestamps and transaction ids that declares when each type value has been added or deleted.
    - the list of `service_endpoint` values, with timestamps and transaction ids that declare when the values were added or deleted.

For example, when a DID is created, an entry is added to the map, that adds the DID and maps it to the initial keys and services described in the corresponding `CreateDIDOperation`. This is:
- it adds the list of keys and set their timestamp indicating when they were added on, it also set to each key the transaction id of the transaction that carried the said operation and
- for each service it adds the singleton lists with `type` and `service_endpoint` values with a corresponding "added on" timestamp and transaction id to each entity.
- if there is a non-empty list in `context`, it also associates it to the DID.
- it also associates the hash of the `AtalaOperation` that wrapped the corresponding `CreateDIDOperation` to the DID.

In the next sections, we will describe the structure of these timestamps, and the precise effect each operation induces in this map.


#### Definition of time

Each Cardano block has an associated timestamp attached to it which describes the real world time. All transactions inside a Cardano block share the same timestamp. Each `SignedAtalaOperation` is located inside an `AtalaBlock` which is attached to a Cardano transaction. We define the timestamp of an operation as the tuple _(cbt, absn, osn)_ where
- `cbt` (*C*ardano *B*lock *T*ime) is the timestamp of the Cardano block that contains the transaction that carries the operation
- `absn` (*A*tala *B*lock *S*equence *N*umber) is the position of the Cardano transaction that carries the `AtalaBlock` in the Cardano block
- `osn` (*O*peration *S*equence *N*umber) is the position of the operation in the `AtalaBlock` that carries it

every DID operation will have an associated timestamp (and transaction id), which `PRISM nodes` will store in their internal state while processing the operations.

NOTE: in order to simplify the reading, from this point on, whenever we write that a timestamp is set/added/assigned, we also implicitly estate that the corresponding transaction id that carries the operation in context is also set/added/assigned accordingly. We recall that all operations carried by the same `AtalaBlock` will share a common transaction id.

### Processing of CreateDIDOperations

Once extracted from an `AtalaBlock`, if a `PRISM node` finds a `SignedAtalaOperation` that contains a `CreateDIDOperation`, then:
- The `SignedAtalaOperation` MUST be well constructed (as defined in previous sections)
- The value in `signature` MUST match a signature of the operation in `operation` performed by the `signed_with` key present in the `public_keys` list defined in the operation. `signed_with` MUST refer to a key of `usage` `MASTER_KEY` 
- The corresponding DID MUST not already be registered in the `PRISM node` internal map
- The `id`s of the public keys listed in `public_keys` MUST be different from each other
- The `id`s of the services listed in `services` MUST be different from each other
- If any of the above checks fail, the operation is ignored and no changes are made to the node map


**Update of internal map**

The `PRISM node` will add to its map, the DID derived from the create operation and associate to it:
- the list of `PublicKey`s, attaching to each of them the corresponding operation timestamp as their addition timestamp.
- the list of `Service`s, for each service
    - it adds a singleton list with the type with the corresponding operation timestamp as it addition timestamp attach to it.
    - it adds a singleton list with the service endpoint value with the corresponding operation timestamp as it addition timestamp attach to it. 
- if `context` is not empty, it associates to this DID the value of `context` as auxiliary data for the resolver.
- It will also associate to this DID the hash of the `AtalaOperation` that contains the `CreateDIDOperation`. We will refer to this hash as its last operation hash.


### Processing of UpdateDIDOperations

Once extracted from an `AtalaBlock`, if a `PRISM node` finds a `SignedAtalaOperation` that contains a `UpdateDIDOperation`, then:
- The `SignedAtalaOperation` MUST be well constructed (as defined in previous sections)
- The value in `signature` MUST match a signature of the operation in `operation` performed by the `signed_with` key. The key MUST be of `MASTER_KEY` usage, and be present in the `public_keys` list associated to the DID to update in the node internal map. The key MUST not have a revocation timestamp associated to it.
- In the node map, the DID corresponding to the `id` of the update operation MUST have the value of `previous_operation_hash` as its associated last operation hash.
- The update operation contains a list of update actions. 
    - `PRISM nodes` MUST process all actions in order (see details below). 
    - If any action fails to be processed, then the entire operation MUST be ignored. This means that any previous successful action from the list that may have been applied, MUST be rolled back 
    - After applying the effect of all the actions of the update operation:
       - There MUST be at least one public key associated to the updated DID in the map, that has `MASTER_KEY` as usage and has no revocation time associated to it. 
       - There MUST not be more than `MAX_SERVICE_NUMBER` services associated to the updated DID in the map where all have no revocation time associated to them. This means that there must not be more than `MAX_SERVICE_NUMBER` active services after the update.
       - There MUST not be more than `MAX_VERIFICATION_METHOD_NUMBER` keys associated to the updated DID in the map where all have no revocation time associated to them. This means that there must not be more than `MAX_VERIFICATION_METHOD_NUMBER` active keys after the update.
       - If any of the previous conditions is not met, the actions MUST be rolled back and the entire update operation is ignored.

Each action will affect the information in the internal map, if all the actions are applied successfully then the last operation hash MUST be updated to the hash of the `AtalaOperation` that wrapped the corresponding `UpdateDIDOperation`

Lets describe how to update the map of DIDs to their state based on each update action type

#### Process AddKeyAction

- The `id` of the `PublicKey` to add MUST NOT already be associated to the DID to update in the map 

**Update of internal map**

The node updates the map, and adds to the corresponding DID the new key on the list of `PublicKeys`, attaching to the key the operation timestamp as addition time. 

#### Process RemoveKeyAction

- The `keyId` of the action MUST already be associated to the DID to update in the map, this key MUST NOT have already an associated revocation time.

**Update of internal map**

The node updates the map, and adds to the key of the corresponding DID, the operation timestamp as its revocation time.


#### Process AddServiceAction

- The `id` of the `Service` to add MUST NOT already be associated to the DID to update in the map 

**Update of internal map**

The node updates the map, and adds to the corresponding DID the new service, attaching to its lists of type and service endpoints the operation timestamp as addition time. 

#### Process RemoveServiceAction

- The `serviceId` of the action MUST already be associated to the DID to update in the map, this service MUST NOT have already an associated deletion time.

**Update of internal map**

The node updates the map, and adds to the service of the corresponding DID, and for the currently active `type` and list of `service_endpoints` the operation timestamp as their deletion times.

#### Process UpdateServiceAction

- The `serviceId` of the action MUST already be associated to the DID to update in the map, 
    - this service will have a list of types, there MUST be exactly one type in this list that does not have already an associated deletion time. We will call this type `current_type`
    - this service will have a list of service endpoints, there MUST be exactly one service endpoint that does not have already an associated revocation time. We will call this list `current_endpoints` 

**Update of internal map**
- If the `UpdateServiceAction` has a non empty `type`, we will refer to it as `new_type`
    - The node adds the operation timestamp to `current_type` as its deletion time, and adds `new_type` to the corresponding list of types with the operation timestamp as its addition timestamp.
- If the `UpdateServiceAction` has a non empty `service_endpoints`, we will refer to it as `new_services`
    - The node adds the operation timestamp to `current_endpoints` as its revocation time, and adds `new_endpoints` to the corresponding list of service endpoints with the operation timestamp as its addition timestamp.

#### Processing PatchContextAction 

If the DID to update has an empty context associated to it in the map:
- the field `context` MUST NOT be empty and MUST NOT contain repeated values

**Update of the internal map**
- If `context` is empty, the DID removes the previous context list associated to it.
- It `context` is not empty, the DID replaces the old list for the new one on its map.

### Processing of DeactivateDIDOperations

Once extracted from an `AtalaBlock`, if a `PRISM node` finds a `SignedAtalaOperation` that contains a `DeactivateDIDOperation`, then:
- The `SignedAtalaOperation` MUST be well constructed (as defined in previous sections)
- The value in `signature` MUST match a signature of the operation in `operation` performed by the `signed_with` key. The key MUST be of `MASTER_KEY` usage, and present in the `public_keys` list associated to the DID to update in the node internal map. The key MUST not have a deletion timestamp associated to it.
- In the node map, the DID corresponding to the `id` of the update operation MUST have the value of `previous_operation_hash` as its associated last operation hash.
- If any of the above checks fail, the operation is ignored and no changes are made to the node map


**Update of internal map**
- For all keys and services that do not have a revocation/deletion timestamp, nodes MUST set the deactivation operation timestamp as the deletion timestamp.
- The new last operation hash MUST be the hash of the `AtalaOperation` that wrapped the corresponding `DeactivateDIDOperation`

Note that this marks all keys (including `MASTER_KEY` keys) as deleted. Meaning that no further updates will be possible on the corresponding DID.

### Processing of ProtocolVersionUpdateOperations

Once extracted from an `AtalaBlock`, if a `PRISM node` finds a `SignedAtalaOperation` that contains a `ProtocolVersionUpdateOperation`, then:
- The `SignedAtalaOperation` MUST be well constructed (as defined in previous sections)
- The value in `signature` MUST match a signature of the operation in `operation` performed by the `signed_with` key. The key MUST be of `MASTER_KEY` usage, and present in the `public_keys` list associated to the `SYSTEM_UPDATE_DID` in the node internal map. The key MUST not have a revocation timestamp associated to it.
- The `effective_since` MUST be positive.
- The new version MUST:
    - Either increase the `major_version` of the protocol, or keep the `major_version` unchanged while increasing the `minor_version`. If the version change is different, the operation is rejected
- If any of the above checks fail, the operation is ignored and no changes are made to the node map


The purpose of this operation is to announce when the new version will become effective. The internal state map will only change if the new rules of the protocol establishes so after the version becomes effective. This is something that the new version of the protocol would define. 


## Translation of PRISM nodes states to DID Documents

In this section, we will refine the DID resolution process.
Given the DID `d` that a user is resolving:

- If `d` is a short form DID`
    - Any `PRISM node` will look in its internal map in search for `d`
        - If `d` is not found, the node returns a `notFound` error
        - If `d` is found, the node will return to the user the content of the map associated to `d`. This is, the list of keys and services information associated to `d` with all the timestamp information. If non-empty, it will also return the list of `context` strings associated to the DID.
- If `d` is in long form, `PRISM nodes` MUST:
    - extract the short form DID from the DID
    - validate the short form DID correspondence (i.e. the encoded operation must match the expected hash), and validates that the decoded operation is a well constructed `AtalaOperation` that contains a `CreateDIDOperation` 
          - If the validations fail, it returns an `invalidDid` error.
    - check if there is information about the short form DID in the internal map. 
        - If there is, it returns the data of the map as in the case of `d` being a short form
        - If there is no information:
            - it decodes the `AtalaOperation` from the long form DID, 
            - run the validations described for a `CreateDIDOperation`
                - if validations fail, the node will return an `invalidDid` error
                - if the validations are successful, the node returns the information that would be generated in its internal map if the node would process the effect of the `CreateDIDOperation` with the difference that there would add no timestamp information (this is because the decoded operation has no associated timestamp)

In order to transform this returned information to a DID document, we do as follows. 
- From the list of public keys' information, extract the keys that have no revocation timestamps associated to them.
- From the list of services' information:
    - take the services' id that have exactly one type and service endpoint's string with no deletion data attach to them
    - construct a list of services that have the mentioned `id`s as `id` and add as `type` and `service_endpoint`, the corresponding only elements that have no deletion time associated to them.
This leaves us with a list of active keys, and a list of active services.
- If we obtain a non-empty list of contexts, we keep it without changes

### Constructing a JSON-LD DID document

If the services' and keys' lists obtained in the previous section are empty, we have that the DID has been deactivated, and we can return an empty JSON object. 

NOTE: The protocol rules guarantee that if the list of keys is empty, then the list of services must be empty. If this is not the case, then this indicates an implementation error.

If the list of keys is not empty, then we construct the DID document as follows: 
- the top level `id` and `controller` are the DID received for resolution, `d`
- the `@context` of the generated document MUST include at a minimum 
      ```
        [
          "https://www.w3.org/ns/did/v1"
        ]
      ```
- Depending on the verification method types and services types in the DID document, additional entries MUST be added to the `@context` after "https://www.w3.org/ns/did/v1", such as:
    - For verification method type `"JsonWebKey2020"`, add `"https://w3id.org/security/suites/jws-2020/v1"`
    - For service type `"DIDCommMessaging"`, add `"https://didcomm.org/messaging/contexts/v2"`
    - For service type `"LinkedDomains"`, add "https://identity.foundation/.well-known/did-configuration/v1"
- If the user added more services' types not covered in the above list, he is responsible for adding their corresponding contexts in the `context` list. If there is a list of context strings associated to the DID, the resolver MUST append it to the end of the `@context` constructed so far.
- For each key that does not have usage `MASTER_KEY` nor `REVOCATION_KEY`, we create an object in the `verificationMethod` field. For each object:
    - The `id` is the key id prepended by the DID received as input and a `#` character separating the strings. For instance, for an identifier `key-1`, and `d` equals to `did:prism:abs` we will obtain the `id` equal to `did:prism:abs#key-1`
    - If the `curve` field value associated to the key is `SECP256K1_CURVE_NAME`
        - The `type` is  "JsonWebKey2020"
        - The `controller` is the DID received for resolution, `d`
        - The `publicKeyJwk` is the JWK public key where:
            - `crv` is "secp256k1"
            - `kty` is "EC"
            - `x` and `y` are the corresponding coordinates of the key
    - If `curve` field value associated to the key is `ED25519_CURVE_NAME`
        - The `type` is "JsonWebKey2020"
        - The `controller` is the DID received for resolution, `d`
        - The `publicKeyJwk` is the JWK public key where:
            - `kty` is "OKP"
            - `crv` is "Ed25519"
            - `x` is derived from the coordinates of the key
    - If `curve` field value associated to the key is `X25519_CURVE_NAME`
        - The `type` is "JsonWebKey2020"
        - The `controller` is the DID received for resolution, `d`
        - The `publicKeyJwk` is the JWK public key where:
            - `kty` is "OKP"
            - `crv` is "X25519"
            - `x` is derived from the coordinates of the key
- for each verification relationship, if there is a key usage matching to it, the verification relationship will make a reference by `id` to the respective key in `verificationMethod`
- for each active service, the translation is trivial as the fields have a one on one correspondence to the W3C data model 
    - The `id` of each service do follow the same rule as `id`s of verification methods. This is, the translation adds the DID to resolve as prefix to the id known by `PRISM nodes` and use a `#` character to separate the strings

### Constructing the DID document metadata

In addition to the DID document, the DID resolution process also returns DID document metadata.

- If `d` is in long form and has been published, DID document metadata MUST contain a `canonicalId` property with the short form DID as its value.
- If `d` is in short form or has not been published, DID document metadata MUST NOT contain a `canonicalId` property.
- DID document metadata MUST contain a `created` property with the timestamp of the Cardano block that contained the first valid `SignedAtalaOperation` with a `CreateDIDOperation` that created the DID.
- DID document metadata MUST contain an `updated` property with the timestamp of the Cardano block that contained the latest valid `SignedAtalaOperation` that changed the DID's internal state.
- DID document metadata MUST contain a `versionId` property with the hash of the `AtalaOperation` contained in the latest valid `SignedAtalaOperation` that created the DID or changed the DID's internal state.
- DID document metadata MUST contain a `deactivated` property with the boolean value `true` if the DID has been deactivated with a valid `DeactivateDIDOperation`. Otherwise, this property is OPTIONAL, but if included, MUST have the boolean value `false`.
- Resolvers MAY add additional method-specific DID document metadata properties, such as
  - `cardanoTransactionPosition`: The position of the Cardano transaction in the Cardano block (i.e. Atala block sequence number)
  - `operationPosition`: The position of the `SignedAtalaOperation` in the `AtalaBlock` (i.e. the operation sequence number)
  - `originTxId`: The Cardano transaction id of the valid `SignedAtalaOperation` that carried the `CreateDIDOperation` that anchored the DID on-chain (if published)
  - `updatedTxId`: The Cardano transaction id of the latest operation that changed the DID's internal state
  - `deactivatedTxId`: Similar for deactivation

Example:

```json
{
  "didDocumentMetadata": {
    "canonicalId": "did:prism:db47e78dd57d2043a7a704fbd9d186a586682110a2097ac06dbc83b35602f290",
    "created": "2023-02-04T13:52:10Z",
    "updated": "2023-01-18T02:19:25Z",
    "versionId": "588ae32c68fe6c2a3af1b076284b6d880a40888025e382ab39dc9dc7cd9de382",
    "deactivated": true,
    "cardanoTransactionPosition": 0,
    "operationPosition": 0
  }
}
```

## Security Considerations

In this section, we will include adequate comments about the common security aspects around DID methods, and how they apply to the `prism` method.

### Eavesdropping

The `prism` method does not generate messages of private nature as part of its protocol. Users create operations only when they intend to submit them to the blockchain or, in the case of DID creation, when they intend to share the DID. Therefore, the `prism` DID method is secure against eavesdropping attacks. It is recommended to users, not to generate `prism` operations if they do not intend to make them public.

### Replay

No operation in the `prism` protocol can suffer a replay attack by construction. `CreateDIDOperation`, once submitted, is associated to a DID. A second publication of the same operation will fail to be applied by `PRISM nodes`. For updates and deactivation, the corresponding operations contain a hash to the preceding operation, creating a linear hash chain. An operation replay would fail a previous hash check.
For the case of protocol updates, the operation always increases the protocol version, making impossible to post twice the same operation.

### Message insertion

All operations that are published in the `prism` protocol require a cryptographic signature. The protocol relays the protection against message insertion attacks to:
1. The security of the signature scheme
2. The security of communication channels

Users should not create operations unless they plan to publish them.

### Deletion

Protocol messages that are publicly shared, are published on the Cardano blockchain.
Due to the nature of decentralized blockchains, a deletion attack would require the attacker to force the deletion of information from all the nodes in the Cardano chain. We consider this a reasonably hard condition to perform such an attack.

### Modification

All messages in the `prism` protocol have an integrity guarantee. Creation operations have an integrity hash check that binds operations to the DID generated. All other operations are cryptographically signed. The method security hence relies in the security of the cryptographic scheme used.

### Denial of service 

In the setting where a user is not hosting his own `PRISM node`, the security relies on the security measures taken by the delegated party.
Any user running a `PRISM node` should take traditional mitigations to these type of attacks. This includes but is not limited to proper resource provision to `PRISM node` deployments, load balancing techniques, rate limitations on queries, push-back to clients.

### Amplification

There are three types of requests a `PRISM node` replies to:
1. Resolution queries 
2. Operation submission queries (publish operations to the blockchain)
3. Processing new operations read from the blockchain

Resolution and operation submission queries can be limited by traditional techniques such as rate limiters and load balancers.
For the case of reading operations from the Cardano blockchain, an attacker would need to publish those operations first, and this requires fees to be paid. These fees impose a cost limit to perform attacks of this nature. Therefore, we rely partially on the underlying blockchain spam protection to avoid resource exhaustion.
Additionally, the protocol limits the length of identifiers, URIs, and other strings inside operations; it also restricts a maximum number of verification methods and services associated to a DID Document. These limitations also contribute to maintain the consumption of resources bounded to reasonable limits. 

### Man-in-the-middle

The messages shared as part of the `prism` protocol do not have expected recipients, making man in the middle attacks non applicable.

### Other considerations

#### Trust delegation settings

We would like to add some comments in relation to the trust setting that the method allows to use. A user can opt to not self host a `PRISM node`. This situation generates a security dependency on the trusted `PRISM node` provider. For instance, a user could submit a DID update to the provider, but this actor could opt to not forward the operation to the blockchain. Furthermore, the node provider could lie to the user and convince him that the operation was indeed submitted and applied.

In order to avoid these type of situations, we suggest the following recommendations:
1. If possible, self host an instance of the `PRISM node`
2. If self hosting is not possible, query a set of `PRISM nodes` provided by different non related actors
3. When an operation is submitted to a node, query about its confirmation to multiple providers
4. `PRISM nodes`, based on implementation, are able to return the transaction id of the underlying blockchain that carries the published operations. 

#### Blockchain funds safety

`PRISM nodes` can submit transactions to their underlying operating blockchain. This implies that fees are paid to the blockchain when transactions are submitted. Such fees are paid by a wallet/account on the chain. When the wallet is managed by the `PRISM node`, proper access control to the node should be in place to avoid draining of the said funds. In particular, it is recommended to have minimal funds on the mentioned wallet, and refund it periodically as demand requires.

#### Protocol update keys

The owner of `SYSTEM_UPDATE_DID` should take the appropriate measures to protect such DID. 
These is, to protect the cryptographic keys that control the said DID. Some possible measures can include applying threshold techniques, which allow the decomposition of cryptographic secrets into pieces, which could be stored by separate actors. The reconstruction of the secrets could be done with access to a subset of all the initial pieces. 

## Privacy Considerations

As most DID methods, it is recommended to users to not add personal identifiable information in their DID documents. This includes:
- Verification method ids
- Services ids
- URIs
- Any other place where it could fit inside DID Documents

In addition, consider correlation risks associated to blockchain transactions. When a user self hosts a `PRISM node`, all his published events will be tied to blockchain transactions. The user should take adequate measures to avoid correlation of independent DIDs. Examples of mitigation strategies are:
- Have multiple wallets for different DIDs
- Use `PRISM nodes` hosted by external providers to publish operations

We want to remind the reader that all published operations of the `prism` protocol are hard, if not impossible, to delete.

## References

## Appendix A: Cardano's metadata encoding

[Cardano's metadata](https://developers.cardano.org/docs/transaction-metadata/) encodes a subset of JSON values.
For the case of `prism` `AtalaObjects`, the top level value is a JSON object of the following shape:

```json
{
  "21325" : {
    "map" : [
      {
         "k" : { "string": "v" },
         "v" : { "int" : 1 }
      },
      {
         "k" : { "string" : "c" },
         "v" : {  
           "list" : group(atalaObject)
         }
      }
    ]
  }
}
```

where `group` takes the `AtalaObject` message to encode and:
1. converts it to a byte array,
1. splits the array into sections of 64 bytes (the last section may contain less bytes)
1. returns a list of JSON objects (one object per section) of the form

```json
  {
    "bytes" : HEX_ENCODED_SEGMENT
  }
```

where `HEX_ENCODED_SEGMENT` is the hexadecimal encoding of the corresponding section of up to 64 bytes.

NOTE: the value `21325` represents the last 16 bits of 344977920845, which is the decimal representation of the concatenation of the hexadecimals 50 52 49 53 4d that form the word PRISM in ASCII.

## Appendix B: Protobuf models

In this section, we can find all the protobuf definitions  referenced in previous sections

```protobuf

/**
 * Represent a block that holds operations.
 */
message AtalaBlock {
  reserved 1; // Represents the version of the block. Deprecated
  repeated SignedAtalaOperation operations = 2; // A signed operation, necessary to post anything on the blockchain.
}

/**
 * Wraps an AtalaBlock and its metadata.
 */
message AtalaObject {
  reserved 1, 2, 3; 
  reserved "block_hash";
  reserved "block_operation_count"; // Number of operations in the block.
  reserved "block_byte_length"; // Byte length of the block.
  
  AtalaBlock block_content = 4; // The block content.
}

// Every key has a single purpose:
enum KeyUsage {
    // UNKNOWN_KEY is an invalid value - Protobuf uses 0 if no value is provided and we want the user to explicitly choose the usage.
    UNKNOWN_KEY = 0;
    MASTER_KEY = 1;
    ISSUING_KEY = 2;
    KEY_AGREEMENT_KEY = 3;
    AUTHENTICATION_KEY = 4;
    REVOCATION_KEY = 5;
    CAPABILITY_INVOCATION_KEY = 6;
    CAPABILITY_DELEGATION_KEY = 7;
}

/**
 * Holds the necessary data to recover an Elliptic Curve (EC)'s public key.
 */
message ECKeyData {
    string curve = 1; // The curve name, like secp256k1.
    bytes x = 2; // The x coordinate, represented as bytes.
    bytes y = 3; // The y coordinate, represented as bytes.
}

/**
 * Holds the compressed representation of data needed to recover Elliptic Curve (EC)'s public key.
 */
message CompressedECKeyData {
    string curve = 1; // The curve name, like secp256k1.
    bytes data = 2; // compressed Elliptic Curve (EC) public key data.
}

/**
 * Represents a public key with metadata, necessary for a DID document.
 */
message PublicKey {
    reserved 3, 4, 5, 6;
    string id = 1; // The key identifier within the DID Document.
    KeyUsage usage = 2; // The key's purpose.

    // The key's representation.
    oneof key_data {
        ECKeyData ec_key_data = 8; // The Elliptic Curve (EC) key.
        CompressedECKeyData compressed_ec_key_data =  9; // Compressed Elliptic Curve (EC) key.
    };
}


// The operation to create a public DID.
message CreateDIDOperation {
    DIDCreationData did_data = 1; // DIDCreationData with public keys and services

    // The data necessary to create a DID.
    message DIDCreationData {
        reserved 1; // Removed DID id field which is empty on creation
        repeated PublicKey public_keys = 2; // The keys that belong to this DID Document.
        repeated Service services = 3; // The list of services that belong to this DID Document.
        repeated string context = 4; // The list of @context values to consider on JSON-LD representations
    }
}

// The necessary data to add a key to a DID.
message AddKeyAction {
    PublicKey key = 1; // The key to include.
}

// The necessary data to remove a key from a DID.
message RemoveKeyAction {
    string keyId = 1; // the key id to remove
}

message AddServiceAction {
    Service service = 1;
}

message RemoveServiceAction {
    string serviceId = 1;
}

message UpdateServiceAction {
    string serviceId = 1; // scoped to the did, unique per did
    string type = 2; // new type if provided
    string service_endpoints = 3;
}

message PatchContextAction {
    repeated string context = 1; // The list of strings to use by resolvers during resolution when producing a JSON-LD output
}

// The potential details that can be updated in a DID.
message UpdateDIDAction {

    // The action to perform.
    oneof action {
        AddKeyAction add_key = 1; // Used to add a new key to the DID.
        RemoveKeyAction remove_key = 2; // Used to remove a key from the DID.
        AddServiceAction add_service = 3; // Used to add a new service to a DID,
        RemoveServiceAction remove_service = 4; // Used to remove an existing service from a DID,
        UpdateServiceAction update_service = 5; // Used to Update a list of service endpoints of a given service on a given DID.
        PatchContextAction patch_context = 6; // Used to Update a list of `@context` strings used during resolution for a given DID.
    }
}

// Specifies the necessary data to update a public DID.
message UpdateDIDOperation {
    bytes previous_operation_hash = 1; // The hash of the most recent operation that was used to create or update the DID.
    string id = 2; // @exclude TODO: To be redefined after we start using this operation.
    repeated UpdateDIDAction actions = 3; // The actual updates to perform on the DID.
}

// Specifies the protocol version update
message ProtocolVersionUpdateOperation {
    string proposer_did = 1; // The DID suffix that proposes the protocol update.
    ProtocolVersionInfo version = 2; // Information of the new version
}


message ProtocolVersion {
    // Represent the major version
    int32 major_version = 1;
    // Represent the minor version
    int32 minor_version = 2;
}

message ProtocolVersionInfo {
    reserved 2, 3;
    string version_name = 1; // (optional) name of the version
    int32 effective_since = 4; // Cardano block number that tells since which block the update is enforced

    // New major and minor version to be announced,
    // If major value changes, the node MUST stop issuing and reading operations, and upgrade before `effective_since` because the new protocol version.
    // If minor value changes, the node can opt to not update. All events _published_ by this node would be also
    // understood by other nodes with the same major version. However, there may be new events that this node won't _read_
    ProtocolVersion protocol_version = 5;
}

message DeactivateDIDOperation {
    bytes previous_operation_hash = 1; // The hash of the most recent operation that was used to create or update the DID.
    string id = 2; // DID Suffix of the DID to be deactivated
}

// The possible operations affecting the blockchain.
message AtalaOperation {
   reserved 3, 4; // fields used by an extension of the protocol. Not relevant for the DID method 
   // The actual operation.
    oneof operation {
        // Used to create a public DID.
        CreateDIDOperation create_did = 1;

        // Used to update an existing public DID.
        UpdateDIDOperation update_did = 2;

        // Used to announce new protocol update
        ProtocolVersionUpdateOperation protocol_version_update = 5;

        // Used to deactivate DID
        DeactivateDIDOperation deactivate_did = 6;
    };
}

// A signed operation, necessary to post anything on the blockchain.
message SignedAtalaOperation {
    string signed_with = 1; // The key ID used to sign the operation, it must belong to the DID that signs the operation.
    bytes signature = 2; // The actual signature.
    AtalaOperation operation = 3; // The operation that was signed.
}

message Service {
    string id = 1;
    string type = 2;
    string service_endpoint = 3;
}
```
