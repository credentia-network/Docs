# Casper DID Smart Contract Tutorial

## Description

This guide provides step-by-step guide to compile, deploy and operate DID smart contract in the Casper blockchain test network. It is expected that the reader has basic understanding of how Casper blockchain works, understands the blockchain transaction lifecycle flow and is fluent in Rust and JavaScript.

## Installing tookits and building the project

1.  Install Rust

    1. Install Rust (if you don’t have it installed already). Please follow this document for installation guidelines: [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)
    2. Install wasm target builder. Please follow this document for installation guidelines: [https://www.hellorust.com/setup/wasm-target/](https://www.hellorust.com/setup/wasm-target/)
    3. Set "nightly" toolchain

    ```
    rustup update
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```
2.  Build the project: simply build with npm

    ```
    npm i
    ```

    then

    ```
    npm run build
    ```
3.  Install casper signer [https://cspr.live/sign-in](https://cspr.live/sign-in)

    1. Follow user onboarding process until it prompts to create new account.

    ![images/Casper\_DID\_Smart\_Contract\_guide/CasperSigner.png](../images/Casper\_DID\_Smart\_Contract\_guide/CasperSigner.png)

    1. Create a set of test accounts with the following names (this is just for demo, please keep names so that they match account names in demo scripts)
    2. Create /network\_keys folder with the following accounts subfolders in it:
       1. deployer
       2. ippolit
       3. victor
       4. trent
    3. Generate keys for each of them with casper signer. Then in casper signer select menu -> key management and download key data into the appropriate folder for each account. There should be 3 files in each folder: public\_key\_hex, public\_key.pem and secret\_key.pem
    4. After all, the folder should look like this:

    ![Folder](../images/Casper\_DID\_Smart\_Contract\_guide/FolderStructure.png)
4.  Request test tokens on [https://testnet.cspr.live/tools/faucet](https://testnet.cspr.live/tools/faucet) for each account

    ![images/Casper\_DID\_Smart\_Contract\_guide/CasperFaucet.png](../images/Casper\_DID\_Smart\_Contract\_guide/CasperFaucet.png)

    Make sure you’ve received test tokens. It takes about 5 min for them to arrive (average block time in casper testnet is 3 min)

## Deploying DID Smart Contract into Casper Testnet

1.  Deploy DID contract with the following command:

    1. Run deploy DID contract script via node.js. Please note that it takes about 3-5 min for transactions to be included into Casper testnet chain.

    ```
    node ./js/did/deploy_did.js
    ```

    1. A contract has been created:

    ```
    Deploy CasperDIDContract hash: 28db52d6093e10a088103fef39d646b5c7fb203bf23d039c7d8f18bc4df778c

    Contract Hash = hash-6a810712e737d78f1c57bb2123308cc08d16f61a67e804b3bba4325578b97e18
    ```
2.  Copy a Contract hash value into /js/constants.js under the CONTRACT\_DID\_HASH key. It should look like this:

    ```
    CONTRACT_HASH: "hash-6a810712e737d78f1c57bb2123308cc08d16f61a67e804b3bba4325578b97e18",
    ```
3. Let’s have fun testing contract now! :)

## Operating DID Smart Contract

### Registering DID Owner.

By default DID owner is not required to stored on-chain. If there's no information about identity on-chain, then the private key of this identity is considered to be owner. Registering DID owner in DID smart contract enforces DID information to be stored on-chain. This can be not just identity private key, but another identity key. For example: Natural person Alice (having her owner DID and controlling it via her private key) can "own" company AliceCorp, inc. identified by another DID. AliceCorp, inc. DID will have a "owner" record in DID registry pointing to Alices' DID.

1.  Register DID owner by invoking.

    ```
    node ./js/did/transact_changeOwner.js
    ```

    Output:

    ```
    Change owner for Identity: da6f12be9c441060e5cc979d3995c60b8fcdcdb09f9765dcd110a19694013ada

    new owner address: cdf55fa93276d59196ec24dcecdb817e5e372d758122fddba57fb80fcfc65384
    ```
2.  Check the owner by invoking:

    ```
    node ./js/did/read_did.js
    ```

    Output:

    ```
    Read Owner for Trent

    owner_identity: cdf55fa93276d59196ec24dcecdb817e5e372d758122fddba57fb80fcfc65384

    Read Owner for Victor

    owner_identity: da6f12be9c441060e5cc979d3995c60b8fcdcdb09f9765dcd110a19694013ada
    ```

### Working with Attributes.

DID Registry contract provides key-value storage for every DID it holds. Each key-value pair is called "Attribute" and can be recorded via special "setAttribute" transaction. Setting attribute requires a "key", which is an Attribute name and a "value". Both are of string type. Expiration date (a timestamp in milliseconds) is set for each Attribute within setAttribute transaction. This parameter needs to be verified every time Attribute is validated. Revoking Attribute is equivalent to setting Expiration date to 0.

1.  Add attribute for identity:

    ```
    node ./js/did/transact_setAttribute.js
    ```

    Output:

    ```
    Deploy result
    a6c20000f1d6fe8cb0ecb200ddc20d84eb2f8fed6488019ba55c5f6058876085
    ```
2.  Read attributes for identity:

    ```
    node ./js/did/read_did.js
    ```

    Output:

    ```
    .........................
    Reading attribute result:
    Read attribute for Victor
    Identity:  da6f12be9c441060e5cc979d3995c60b8fcdcdb09f9765dcd110a19694013ada
    service-endpoint => ''

    Read attribute for Trent
    Reading attribute result:
    Identity:  cdf55fa93276d59196ec24dcecdb817e5e372d758122fddba57fb80fcfc65384
    service-endpoint => https://myservice2.com
    Expires at:  10/11/2021, 6:40:30 AM
    ```
3.  Revoke attribute for identity:

    ```
    node ./js/did/transact_revokeAttribute.js
    ```

    Output:

    ```
    Deploy result
    dd3f40690d6710b1e5ebbe7dfc4c2fef7c10211777f043ed37998bd3027b8a46
    ```

### Working with Delegates.

In case an Identity (identified by it's DID) is willing to delegate its credentials to another Identity (identified by another DID) for a limited amount of time, it may send a special transaction providing a "delegationType", "delegatee" (identity of the DID that is going to work on behalf of) and Expiration date. Delegation type is stored as a 32 bytes value, so it is expected that it will contain a hash data. Something like sha256("Power of Attorney"). Delegatee is identified by a it's public\_key and Expiration date is a timestamp in milliseconds. In case Delegation needs to be revoked, a transaction invoking revokeDelegate will do the job. Revocation transaction call will terminate Delegatees permissions by setting Expiration date to current timestamp.

1.  Add delegate for identity:

    ```
    node ./js/did/transact_addDelegate.js
    ```

    Output:

    ```
    Deploy result
    ad284c360a6d956d813484c71f8246aba5d971ce5863e9f63933ea077988f4e6
    ```
2.  Read delegates for Identity:

    ```
    node ./js/did/read_did.js
    ```

    Output:

    ```
    ................................
    Reading delegate result
    Delegatee exists and expires at: 10/11/2021, 6:37:12 AM
    ```
3.  Revoking delegation data for Identity:

    ```
    node ./js/did/transact_revokeDelegate.js
    ```

    Output:

    ```
    Deploy result
    21306c99030dd97650382e327a8d1483713a2d81dbf7e14aec5fc2ef5ae75214
    ```

    Then, the output of read\_did.js script will show

    ```
    ................................
    Reading delegate result
    Delegatee has been revoked
    ```
