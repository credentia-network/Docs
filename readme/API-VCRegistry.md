# DemoVCRegistry contract

## General

DemoVCRegistry is a reference implementation of the VC registry contract that can be a backbone for decentralized document management system based on w3c VC specification. The contract implements registration of VCs, revoking and also a Verifiable Presentation request/response flow.

## Public state changing functions

### Function issueDemoVC()

Function issueDemoVC() is designed to register Verifiable Credentials document that is issued by an Issuer address for the Holder

It require 5 parameters to be provided:
- merkleRoot - 32 bytes. Subject data merkle root. Can be used for sharing credentials with the help of selective disclosure protocol.
- ipfsHash - 32 bytes as the decoded ipfsHash from base58 with leading 2 bytes truncated. Declares the verifiable credentials stored on ipfs. Can be used for generating base58 representation of IPFS hash and downloading VC document.
- schemaHash - 32 bytes. Declares the hash from schema, which is constructed from IPFS hash of the schema document. Can be used to generate base58 IPFS hash of the schema document.
- holder - hash of the Holders casper account, which is represented by 32 bytes.
- revocationFlag - boolean. False when the document is active, true - when the VC document is revoked by the Issuer.

```
    #[no_mangle]
    pub extern "C" fn issueDemoVC(){
    let merkleRoot : [u8;32] = runtime::get_named_arg("merkleRoot");
    let ipfsHash : [u8;32] = runtime::get_named_arg("ipfsHash");
    let schemaHash : [u8;32] = runtime::get_named_arg("schemaHash");
    let holder : AccountHash = runtime::get_named_arg("holder");
    let revocationFlag : bool = runtime::get_named_arg("revocationFlag");
    let issuer : AccountHash = runtime::get_caller();
    let vcLengthKey = &vc_length_key(&issuer);
    let index : u64 = get_key(vcLengthKey);
    set_key(&vc_key(&issuer,index,"merkleRoot"), merkleRoot);
    set_key(&vc_key(&issuer,index,"ipfsHash"), ipfsHash);
    set_key(&vc_key(&issuer,index,"schemaHash"), schemaHash);
    set_key(&vc_key(&issuer,index,"holder"), holder);
    set_key(&vc_key(&issuer,index,"revocationFlag"), revocationFlag);
    set_key(&vclink_key(&holder, index), vc_key(&issuer, index, ""));
    let next_index : u64 = index + 1;
    set_key(vcLengthKey, next_index);
    set_key(&vclink_length_key(&holder), next_index);
    }
```

### Function changeVCRevocationFlag()

Function changeVCRevocationFlag() changes the revocation flag on verifiable credentials of the issuer by the issuer's verifiable credentials index.

Has 2 parameters:
- index - unsigned 64-bit integer. The ID of the Verifiable Credentials document in the VC Registry storage
- revocationFlag - boolean. The flag that declares the revocation state. True - revocated, false - active

```
    #[no_mangle]
    pub extern "C" fn changeVCRevocationFlag(){
    // Parameter::new("index", CLType::U64),
    // Parameter::new("revocationFlag",CLType::Bool),
    let index : u64 = runtime::get_named_arg("index");
    let revocationFlag : bool = runtime::get_named_arg("revocationFlag");
    let issuer : AccountHash = runtime::get_caller();
    let vc_length : u64 = get_key(&vc_length_key(&issuer));
    if index >= vc_length {
      runtime::revert(ApiError::InvalidArgument);
      }
    let vcRevocationFlagKey = &vc_key(&issuer, index,"revocationFlag");
    set_key(vcRevocationFlagKey, revocationFlag);
    }
```

### Function sendVPRequest()

Function sendVPRequest() is designed to register verifiable presentation request issued by a Verifier in the address of a Holder.

It requires 2 parameters:
- ipfsHash - 32 bytes as the hex encoded IPFS has of the Verifiable Presentation document. Can be used to generate base58 representation of the IPFS in order to download VP request from IPFS.
- holder - hash of the Holders casper account, which is represented by 32 bytes. Declares the holder who neds to receive this request and generate the response.

```
    #[no_mangle]
    pub extern "C" fn sendVPRequest(){
    // Parameter::new("ipfsHash", CLType::ByteArray(32)),
    // Parameter::new("holder",AccountHash::cl_type()),
    let ipfsHash : [u8;32] = runtime::get_named_arg("ipfsHash");
    let holder : AccountHash = runtime::get_named_arg("holder");
    let ipfsHashResponce : [u8;32] = [0u8;32];
    let verifier : AccountHash = runtime::get_caller();
    let vprequestLengthKey : &String = &vprequest_length_key(&verifier);
    let index : u64 = get_key(vprequestLengthKey);
    set_key(&vprequest_key(&verifier, index, "ipfsHash"), ipfsHash);
    set_key(&vprequest_key(&verifier, index, "ipfsHashResponce"), ipfsHashResponce);
    set_key(&vprequest_key(&verifier, index, "holder"), holder);
    set_key(&vprequest_key(&verifier, index, "status"), 0u8); // set pending status
    set_key(&vprequestlink_key(&holder, index), vprequest_key(&verifier, index, ""));
    let next_index : u64 = index + 1;
    set_key(vprequestLengthKey, next_index);
    set_key( &vprequestlink_length_key(&holder), next_index);
    }
```

### Function changeVPRequestStatus()

Function changeVPRequestStatus() changes the status of verifiable presentation. The verifier can only cancel the VPRequest. The holder can approve or reject the VPRequest. The mapping of statuses is:

0 => pending
1 => approved
2 => rejected
3 => canceled

Has 4 parameters:
- verifier - hash of casper account, which is represented by 32 bytes. Declares the verifier
- index - unsigned 64-bit integer. Declares the index of verifiable presentation issued by verifier
- newStatus - unsigned 8-bit integer. Declares the new status of verifiable presentation
- ipfsHashResponce - 32 bytes as the decoded ipfsHash from base58 and truncated from first two bytes. Declares the verifiable presentation stored on ipfs. Declares the response from holder that stored on ipfs.

```
#[no_mangle]
pub extern "C" fn changeVPRequestStatus(){
// Parameter::new("verifier", AccountHash::cl_type()),
// Parameter::new("index", CLType::U64),
// Parameter::new("newStatus",CLType::U8),
// Parameter::new("ipfsHashResponce", CLType::ByteArray(32)),
/* statuses:
0 => pending
1 => approved
2 => rejected
3 => canceled
- /
let verifier : AccountHash = runtime::get_named_arg("verifier");
let index : u64 = runtime::get_named_arg("index");
let newStatus : u8 = runtime::get_named_arg("newStatus");
let ipfsHashResponce : [u8;32] = runtime::get_named_arg("ipfsHashResponce");
let vprequest_length : u64 = get_key(&vprequest_length_key(&verifier));
if index >= vprequest_length || newStatus == 0u8 {
  runtime::revert(ApiError::InvalidArgument);
  }
let vpRequestStatusKey : &String = &vprequest_key(&verifier, index, "status");
let currentStatus : u8 = get_key(vpRequestStatusKey);
if currentStatus != 0u8 {
  runtime::revert(ApiError::ContractHeader(currentStatus));
  }
  let msgSender : AccountHash = runtime::get_caller();
  if msgSender == verifier && newStatus == 3 {
    set_key(vpRequestStatusKey, 3);
    return;
  }
  let holder : AccountHash = get_key(&vprequest_key(&verifier, index, "holder"));
  if msgSender == holder && newStatus < 3u8 {
    set_key(vpRequestStatusKey, newStatus);
    set_key(&vprequest_key(&verifier, index, "ipfsHashResponce"), ipfsHashResponce);
  } else {
    runtime::revert(ApiError::PermissionDenied);
  }
}
```
