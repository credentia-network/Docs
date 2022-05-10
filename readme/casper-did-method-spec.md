# Casper DID Method Specification

This document is derived from [casper DID spec](https://github.com/credentia-network/Docs/blob/main/readme/casper-did-method-spec.md) and is following it in many aspects.

## Preface

The Casper DID method specification conforms to the requirements specified in the [DID specification](https://w3c-ccg.github.io/did-core/), currently published by the W3C Credentials Community Group. For more information about DIDs and DID method specifications, please see the [DID Primer](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md)

## Abstract

Decentralized Identifiers (DIDs, see \[1]) are designed to be compatible with any distributed ledger or network.

The described DID method allows any Casper smart contract or key pair account to become a valid identity. An identity needs no registration. In the case that key management or additional attributes such as "service endpoints" or "verification keys" are required, they could be stored in the DID registry implemented with the smart contract, which can be deployed to any Casper network. Please also see the reference implementation of this standard [here](https://github.com/credentia-network/caspercontracts)

### Identity Controller

By default, each identity in the SSI world is controlled by itself. Each identity can only be controlled by a single address at any given point of time. By default, this is the address of the identity itself. The controller can replace themselves with any other Casper address, including contracts, to allow more advanced models such as multi-signature controllership.

## Target System

The target system is the Casper blockchain network where the DID registry contract is deployed. Technically, this could be any blockchain operating Casper smart contract engine.

### Advantages

* No transaction fee for identity creation
* Uses Casper blockchain's built-in account abstraction
* Supports secp256k1 public keys as identifiers (on the same infrastructure)
* Supports Ed25519 public keys as identifiers (on the same infrastructure)
* Decoupling claims data from the underlying identity
* Flexibility to use key management
* Supports any Casper-compliant blockchain

## JSON-LD Context Definition

Since this DID method still supports `publicKeyHex` and `publicKeyBase64` encodings for verification methods, it requires a valid JSON-LD context for those entries. To enable JSON-LD processing, the `@context` used when constructing DID documents for `did:casper` should be:

```javascript
"@context": [
  "https://www.w3.org/ns/did/v1",
  "https://identity.foundation/EcdsaSecp256k1RecoverySignature2020/lds-ecdsa-secp256k1-recovery2020-0.0.jsonld"
]
```

You will also need this `@context` if you need to use `EcdsaSecp256k1RecoveryMethod2020` in your apps.

## DID Method Name

The namestring that shall identify this DID method is: `casper`

A DID that uses this method MUST begin with the following prefix: `did:casper`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

The method specific identifier is represented as the Hex encoded secp256k1 public key (in compressed form), or the corresponding Hex-encoded Casper address on the target network, prefixed with `0x`.

```
casper-did = "did:casper:" casper-specific-identifier
casper-specific-identifier = [ casper-network ":" ] casper-address / public-key-hex
casper-network = "casper" / "casper-test" / "any-other-network-name"
casper-address = "[01|02]...." 66*HEXDIG
public-key-hex = "0x...." 90*HEXDIG for Ed25519 or 114*DEXDIG for Secp256k1 keys
```

The `casper-address` or `public-key-hex` are case-insensitive.

Note, if no public casper network was specified, it is assumed that the DID is anchored on the Casper mainnet by default. This means the following DIDs will resolve to equivalent DID Documents:

```
did:casper:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097
did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097
```

If the identifier is a `public-key-hex`:

* it MUST be represented in compressed form (see https://en.bitcoin.it/wiki/Secp256k1)
* the corresponding `blockchainAccountId` entry is also added to the default DID document, unless the `owner` has been changed to a different address.
* all Read, Update, and Delete operations MUST be made using the corresponding `blockchainAccountId` and MUST originate from the correct controller (`owner`) address.

## Relationship to DID Registry Contract

The subject of a `did:casper` is mapped to an `identity` address in the DID contract. When dealing with public key identifiers, the corresponding Casper address is used.

The controller address of a `did:casper` is mapped to the `owner` of an `identity` in the DID registry contract. The controller address is not listed as the [DID `controller`](https://www.w3.org/TR/did-core/#did-controller) property in the DID document. This is intentional, to simplify the verification burden required by the DID spec. Rather, this address it is a concept specific to DID Registry Contract and defines the address that is allowed to perform Update and Delete operations on the registry on behalf of the `identity` address. This address MUST be listed with the ID `${did}#controller` in the `verificationMethod` section and also referenced in all other verification relationships listed in the DID document. In addition to this, if the identifier is a public key, this public key MUST be listed with the ID `${did}#controllerKey` in all locations where `#controller` appears.

## CRUD Operation Definitions

### Create (Register)

In order to create a `casper` DID, an Casper address, i.e., key pair, needs to be generated. At this point, no interaction with the target Casper network is required. The registration is implicit as it is impossible to brute force an Casper address, i.e., guessing the private key for a given public key on the Koblitz Curve (secp256k1). The holder of the private key is the entity identified by the DID.

The minimal DID document for an Casper address on mainnet, e.g., `02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097` with no transactions to the DID Registry Contract registry looks like this:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://identity.foundation/EcdsaSecp256k1RecoverySignature2020/lds-ecdsa-secp256k1-recovery2020-0.0.jsonld"
  ],
  "id": "did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097",
  "verificationMethod": [
    {
      "id": "did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097#controller",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097",
      "blockchainAccountId": "02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097"
    }
  ],
  "authentication": ["did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097#controller"],
  "assertionMethod": ["did:casper:02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097#controller"]
}
```

The minimal DID Document for a public key where there are no corresponding TXs to the DID Registry Contract registry looks like this:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://identity.foundation/EcdsaSecp256k1RecoverySignature2020/lds-ecdsa-secp256k1-recovery2020-0.0.jsonld"
  ],
  "id": "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe",
  "verificationMethod": [
    {
      "id": "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controller",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe",
      "blockchainAccountId": "020360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe"
    },
    {
      "id": "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controllerKey",
      "type": "EcdsaSecp256k1VerificationKey2019",
      "controller": "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe",
      "publicKeyHex": "0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe"
    }
  ],
  "authentication": [
    "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controller",
    "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controllerKey"
  ],
  "assertionMethod": [
    "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controller",
    "did:casper:0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe#controllerKey"
  ]
}
```

### Read (Resolve)

The DID document is built by using read only functions of the DID Registry Contract registry.

Any value from the registry that returns an Casper address will be added to the `verificationMethod` array of the DID document with type `EcdsaSecp256k1RecoveryMethod2020` or `Ed25519VerificationKey2020` and an `blockchainAccountId` attribute containing the address.

#### Controller Address

Each identity always has a controller address. By default, it is the same as the identity address, but check the read only contract function `identityOwner(address identity)` on the deployed version of the DID Registry Contract contract.

The identity controller will always have a `verificationMethod` entry with the id set as the DID with the fragment `#controller` appended.

An entry for the controller is also added to the `authentication` array of the DID document.

#### Requesting data from the DID registry contract

Unlike Ethereum, Casper doesn't have anything like Events in smart contracts. Casper DID Registry implementation completely relies on the contract key-value storage. Owner, Attribute or Delegate data will be stored on-chain and can be requested via `getBlockState()` function.

Assuming `accountHash` is an accountID in Casper blockchain and `index` is a slot index in array of attributes or delegates, the DID Registry Contract data can be requested using the following keys:

* `owner_accountHash` (blockchainAccountID, the `controller`)
* `accountHash_attributeLength` (Number, the length of attributes array stored for a particular identity)
* `accountHash_delegateLength` (Number, the length of delegates array stored for a particular identity)
* `accountHash_attribute_index` (Tuple \<attributeName, attributeValue, expirationTimestamp>, attribute data)
* `accountHash_delegate_index` (Tuple \<delegateName, delegateValue, expirationTimestamp>, delegate data)

NOTES:

* `accountHash` to be replaces with appropriate value. Example: `owner_accountHash` => `owner_02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097`
* `index` to be replaces with appropriate value between 0 and the value of `accountHash_attributeLength` or `accountHash_delegateLength`. Example: `accountHash_attribute_index` => `02036c69ff629ddacd368b21ff35f93f750290198094eca39a529f1d76f7249fc097_attribute_0`

Each Attribute Tuple consists of 3 parameters:

* `attributeName` (String, an attribute name. See Attributes naming convention details below)
* `attributeValue` (String, an attribute value)
* `expirationTimestamp` (Date-time UTC, expiration date-time. i.e. attribute value is valid until this date/time)

Each Delegate Tuple consists of 3 parameters:

* `delegateName` (String, a delegate name. See Delegates naming convention details below)
* `delegateValue` (String, an delegate value)
* `expirationTimestamp` (Date-time UTC, expiration date-time. i.e. delegate value is valid until this date/time)

All the delegates and attributes historically recorded under some identity can not be completely removed from the DID registry. Instead, a revocation procedure is applied. A revocation of an Attribute or Delegate is equal to changing `expirationTimestamp` to the block timestamp. This way, when block is mined, Attribute or Delegate is no longer valid;

**Controller changes**

When the controller address of a `did:casper` is changed, a new owner can be obtained by querying a value under the `owner_accountHash` key, where accountHash is a corresponding identity address:

The owner data MUST be used to update the `#controller` entry in the `verificationMethod` array. When resolving DIDs with publicKey identifiers, if the controller(owner) address is different from the corresponding address of the publicKey, then the `#controllerKey` entry in the `verificationMethod` array MUST be omitted.

**Delegate Keys**

Delegate keys are Casper addresses that can either be general signing keys or optionally also perform authentication.

They are verifiable from DID Registry smart contract (on-chain).

When a delegate is added, it's value appears under `accountHash_delegate_index` key and the value `accountHash_attributeLength` is increased by 1. When the delegate is removed, the `expirationTimestamp` parameter of the tuple under `accountHash_delegate_index` key is set to current block timestamp, which means that after block is mined this delegate is no longer valid.

The only 2 `delegateTypes` that are currently published in the DID document are:

* `veriKey` which adds a `EcdsaSecp256k1RecoveryMethod2020` or `Ed25519VerificationKey2020` to the `verificationMethod` section of the DID document with the `blockchainAccountId`(`CasperAddress`) of the delegate.
* `sigAuth` which adds a `EcdsaSecp256k1RecoveryMethod2020` to the `verificationMethod` section of document and a corresponding entry to the `authentication` section.

Only records with a `expirationTimestamp` (measured in milliseconds) greater or equal to the current time should be included in the DID document. When resolving an older version (using `versionId` in the didURL query string), the `expirationTimestamp` entry MUST be compared to the timestamp of the block of `versionId` height.

Such valid delegates MUST be added to the `verificationMethod` array as `EcdsaSecp256k1RecoveryMethod2020` or `Ed25519VerificationKey2020` entries, with `delegate` listed under `blockchainAccountId`

Example:

```json
{
  "id": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a#delegate-1",
  "type": "EcdsaSecp256k1RecoveryMethod2020",
  "controller": "did:casper:013544b192ace929b6e1615328d1b3f79d57be81e9f88c5a74e8a8b9bba7080139",
  "blockchainAccountId": "013544b192ace929b6e1615328d1b3f79d57be81e9f88c5a74e8a8b9bba7080139"
}
```

**Attributes**

Non-Casper related keys, service endpoints etc. can be added using attributes. Attributes are stored on-chain and can be queried from within smart contract code.

While any attribute can be stored, for the DID document we currently support adding to each of these sections of the DID document:

* Public Keys (Verification Methods)
* Service Endpoints

**Public Keys**

The name of the attribute added to DID Registry Contract should follow this format: `did/pub/(Secp256k1|RSA|Ed25519|X25519)/(veriKey|sigAuth|enc)/(hex|base64|base58)`

(Essentially `did/pub/<key algorithm>/<key purpose>/<encoding>`)

**Key purposes**

* `veriKey` adds a verification key to the `verificationMethod` section of document
* `sigAuth` adds a verification key to the `verificationMethod` section of document and adds an entry to the `authentication` section of document.
* `enc` adds a key agreement key to the `verificationMethod` section. This is used to perform a Diffie-Hellman key exchange and derive a secret key for encrypting messages to the DID that lists such a key.

> **Note** The `<encoding>` only refers to the key encoding in the resolved DID document. Attribute values sent to the DID Registry Contract registry should always be hex encodings of the raw public key data.

**Example Hex encoded Secp256k1 Verification Key**

An attribute record for the identity `01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a` with the name `did/pub/Secp256k1/veriKey/hex` and the value of `02360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe` generates a key entry like the following (assuming the Attribute index = 0):

```json
{
  "id": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a#delegate-1",
  "type": "EcdsaSecp256k1VerificationKey2019",
  "controller": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a",
  "publicKeyHex": "0x3036301006072a8648ce3d020106052b8104000a0322000360f05668a291e01fdd7b3543a85f353eaf1e98172ddf5e138dbf27e2d7a3acfe"
}
```

**Example Base58 encoded Ed25519 Verification Key**

An attribute record for the identity `01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a` with the name `did/pub/Ed25519/veriKey/hex` and the value of `01706798f37e5e10ba90fcfa0ac81e2564a8d3f78edacbe84050c20e520891f70a` generates a public key entry like this:

```json
{
  "id": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a#delegate-1",
  "type": "Ed25519VerificationKey2018",
  "controller": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a",
  "publicKeyBase58": "0x302a300506032b6570032100706798f37e5e10ba90fcfa0ac81e2564a8d3f78edacbe84050c20e520891f70a"
}
```

**Example Base64 encoded X25519 Encryption Key**

An attribute record for the identity`01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a` with the name `did/pub/X25519/enc/base64` and the value of `MCowBQYDK2VuAyEAEYVXd3/7B4d0NxpSsA/tdVYdz5deYcR1U+ZkphdmEFI=` generates a public key entry like this:

```json
{
  "id": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a#delegate-1",
  "type": "X25519KeyAgreementKey2019",
  "controller": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a",
  "publicKeyBase64": "MCowBQYDK2VuAyEAEYVXd3/7B4d0NxpSsA/tdVYdz5deYcR1U+ZkphdmEFI="
}
```

**Service Endpoints**

The name of the attribute should follow this format:

`did/svc/[ServiceName]`

Example:

An attribute record for the `0xf3beac30c498d9e26865f34fcaa57dbb935b0d74` with the name `did/svc/api` and value of the URL `https://my.api.com` generates a service endpoint entry like the following:

```json
{
  "id": "did:casper:01013a0741b3703f40ec00b40b00f38a0637c933f38fc2dc86131d6aaa1f52a85a#service-1",
  "type": "HubService",
  "serviceEndpoint": "https://hubs.uport.me"
}
```

#### `id` properties of entries

With the exception of `#controller` and `#controllerKey`, the `id` properties that appear throughout the DID document MUST be stable across updates. This means that the same key material will be referenced by the same ID after an update.

* Attribute or delegate changes that result in `verificationMethod` entries MUST set the `id` `${did}#delegate-${attributeIndex}`.
* Attributes that result in `service` entries MUST set the `id` to `${did}#service-${attributeIndex}`

where `attributeIndex` is the index of the attribute that modifies that section of the DID document.

**Example**

* add key => `#delegate-1` is added
* add another key => `#delegate-2` is added
* add delegate => `#delegate-3` is added
* add service => `#service-1` ia added
* revoke first key => `#delegate-1` gets removed from the DID document; `#delegate-2` and `#delegte-3` remain.
* add another delegate => `#delegate-5` is added (earlier revocation is counted as an event)
* first delegate expires => `delegate-3` is removed, `#delegate-5` remains intact

### Delete (Revoke)

Two cases need to be distinguished:

* In case no changes were written to DID Registry Contract, nothing needs to be done, and the private key which belongs to the Casper address needs to be deleted from the storage medium used to protect the keys, e.g., mobile device.
* In case DID Registry Contract was utilized, the owner of the identity needs to be set to `010000000000000000000000000000000000000000000000000000000000000000`. Although, `010000000000000000000000000000000000000000000000000000000000000000`is a valid Casper address, this will indicate the identity has no owner which is a common approach for invalidation, e.g., tokens. To detect if the owner is the null address, one must get the logs of the last change to the identity and inspect if the owner was set to the null address (`010000000000000000000000000000000000000000000000000000000000000000`). It is impossible to make any other changes to the DID document after such a change, therefore all preexisting keys and services are considered revoked.

If the intention is to revoke all the signatures corresponding to the DID, the second option MUST be used.

The DID resolution result for a deactivated DID has the following shape:

```json
{
  "didDocumentMetadata": {
    "deactivated": true
  },
  "didResolutionMetadata": {
    "contentType": "application/did+ld+json"
  },
  "didDocument": {
    "@context": "https://www.w3.org/ns/did/v1",
    "id": "<the deactivated DID>",
    "verificationMethod": [],
    "authentication": []
  }
}
```

## Metadata

The `resolve` method returns an object with the following properties: `didDocument`, `didDocumentMetadata`, `didResolutionMetadata`.

### DID Document Metadata

When resolving a DID document that has had updates, the latest update MUST be listed in the `didDocumentMetadata`.

* `versionId` MUST be the block number of the latest update.
* `updated` MUST be the ISO date string of the block time of the latest update (without sub-second resolution).

Example:

```json
{
  "didDocumentMetadata": {
    "versionId": "12090175",
    "updated": "2021-03-22T18:14:29Z"
  }
}
```

### DID Resolution Metadata

```json
{
  "didResolutionMetadata": {
    "contentType": "application/did+ld+json"
  }
}
```

#### Security considerations of DID versioning

Applications must take precautions when using versioned DID URIs. If a key is compromised and revoked then it can still be used to issue signatures on behalf of the "older" DID URI. The use of versioned DID URIs is only recommended in some limited situations where the timestamp of signatures can also be verified, where malicious signatures can be easily revoked, and where applications can afford to check for these explicit revocations of either keys or signatures. Wherever versioned DIDs are in use, it SHOULD be made obvious to users that they are dealing with potentially revoked data.

## Reference Implementations

The code at [https://github.com/credentia-network/caspercontracts](https://github.com/credentia-network/Docs/blob/main/readme/casper-did-method-spec.md) is intended to present a reference implementation of this DID Registry contract.

## References

**\[1]** [https://w3c-ccg.github.io/did-core/](https://w3c-ccg.github.io/did-core/)

**\[2]** [https://github.com/credentia-network/caspercontracts](https://github.com/credentia-network/caspercontracts)

**\[3]** [https://github.com/credentia-network/casper-did-resolver](https://github.com/credentia-network/casper-did-resolver)
