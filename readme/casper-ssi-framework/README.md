# Casper DID Documentation

## Introduction

Casper Self-Sovereign Identity Framework is created to make use of W3C [Verifiable Credentials Data Model 1.0](https://www.w3.org/TR/vc-data-model/) compliant documents, creating/managing W3C [Decentralized Identifiers (DIDs) v1.03 spec](https://www.w3.org/TR/did-core) compliant DIDs and more. The client SDK contains a library and tooling to interact with the Casper blockchain and a suite of tools to manage a full lifecycle of verifiable credentials.

### Video Tutorial

{% embed url="https://www.loom.com/share/51d94daa8ffd4ed09113f4c29aa879bc" %}

### What is this all about?

Casper Self-Sovereign Identity Framework is the project intended to create base layer infrastructure over Casper Blockchain network that allows developers to make use and integrate Verifiable Credentials into their products (assuming these products are also Casper Blockchain based).

Features:

* Creating/Manipulating/Revoking DIDs derived from Casper blockchain addresses (and keys)
* Resolving Casper DIDs using Universal Resolver
* Creating/Revoking Verifiable Credentials

### What is it for?

Decentralized identifiers (DIDs) can be used in a variety of use cases, for example: no-password authentication. More information about use cases can be found [here](https://www.w3.org/TR/did-use-cases/). Here we provide a toolkit for creating and manipulating the DIDs created for Casper Blockchain network.

Verifiable Credentials (VC) is the digital document format that usually describes a fact about a subject or a relationship between different subjects, and is issued by the respective Issuer. When referencing to the Subjects or Issuer, VCs make use of DIDs (so requires for DIDs to be created before referenced in VCs). Also, VC verification procedure requires that Verifier validates all referenced DIDs to make sure they were valid as of the issuance date. DID validation process is called “resolution” and is performed with the help of a so-called DID resolver.

### Roadmap

Below is the project Implementation roadmap. Please note that the dates are approximate. We are working hard to stay with these dates.

* 08/2021 Roadmap, Whitepaper, Technical overview
* 10/2021 Casper DIDs (CRUR operations)
* 11/2021 Verifiable Credentials (CRUR operations + revocation lists)
* 12/2021 Demo application

### What we are not doing

Here we create a toolkit for using with the Casper Blockchain network. By default, this toolkit is not intended to be used for creating DIDs for methods other than “casper”. Same to DIDs, Verifiable credentials toolkit can be used only with Casper based DIDs.

The toolkit is based on open source Veramo toolkit, which can be configured to use different DIDs and VC Storages by adding and configuring plugins. So, in case other DID methods or VC storages are required, they can be added manually by adding and configuring related Veramo plugins.

##
