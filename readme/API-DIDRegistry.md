# DID Registry Contract API

Did contract is the implementation of the decentralized identity registry implemented according to [casper DID spec](./casper-did-method-spec.md).


## Function changeOwner
Function changeOwner changes the owner of the identity.

Has 2 parameters:
- identity - hash of casper account, which is represented by 32 bytes. Defines the identity to set owner to.
- newOwner - hash of casper account, which is represented by 32 bytes. A new owner

```
    pub extern "C" fn changeOwner(){
    let identity: AccountHash = runtime::get_named_arg("identity");
    let new_owner: AccountHash = runtime::get_named_arg("newOwner");
    if _identity_owner(identity) != runtime::get_caller(){
      runtime::revert(ApiError::PermissionDenied);
      }
    set_key(&owner_key(&identity),new_owner);
    }
```

## Function addDelegate()
Function addDelegate() adds a delegate to identity. The delegate is an identity that can either be general signing keys or optionally also perform authentication. The list of delegates can be imagined as an array, with enumeration from 0. By adding the delegate, the length of the delegate array increases by 1.**Example**: Initially an identity has no delegates and length of delegates array is zero. When addDelegate() is invoked, the size of the array increases by 1 and the index of the new delegate is 0.

Function has 4 parameters:

- identity - hash of casper account, which is represented by 32 bytes. Declares the identity a delegate will be added to
- delegateKey - string variable. Declares the delegate key
- delegateValue - string variable. Delegate pub key.
- expire - unsigned 64-bit integer. The timestamp in milliseconds with an expiration date of the delegation.

```
    pub extern "C" fn addDelegate(){
    let identity: AccountHash = runtime::get_named_arg("identity");
    let delegateKey: String = runtime::get_named_arg("delegateKey");
    let delegateValue: String = runtime::get_named_arg("delegateValue");
    let expire: u64 = runtime::get_named_arg("expire");
    if _identity_owner(identity) != runtime::get_caller(){
      runtime::revert(ApiError::PermissionDenied);
      }
    let now_blocktime: BlockTime = runtime::get_blocktime();
    let now: u64 = now_blocktime.into();
    if expire <= now {
      runtime::revert(ApiError::InvalidArgument);
      }
    let value:(String,String,u64) = (delegateKey, delegateValue, expire);
    let index: u64 = get_key(&delegate_length_key(&identity));
    let next_index:u64 = index+1;
    set_key(&delegate_key(&identity, index), value);
    set_key(&delegate_length_key(&identity), next_index);
    }
```

## Function revokeDelegate()
Function revokeDelegate() essentially revokes the existing and active delegate of an identity. Can be invoked by the identity owner only.

Function has 2 parameters:
- identity - hash of casper account, which is represented by 32 bytes. The parent identity a delegate to be revoked from.
- index - unsigned 64-bit integer. The index of identities delegate, that has to be revoked.

```
    pub extern "C" fn revokeDelegate(){
    let identity: AccountHash = runtime::get_named_arg("identity");
    let index: u64 = runtime::get_named_arg("index");
    if _identity_owner(identity) != runtime::get_caller(){
      runtime::revert(ApiError::PermissionDenied);
      }
    let mut value: (String,String, u64) = get_key(&delegate_key(&identity, index));
    let now_blocktime: BlockTime = runtime::get_blocktime();
    let now: u64 = now_blocktime.into();
    value.2 = now;
    set_key(&delegate_key(&identity, index), value);
    }
```

## Function setAttribute()
Function setAttribute() adds an attribute to identity. Can be invoked by the identity owner only.

Has 4 parameters:
- identity - hash of casper account, which is represented by 32 bytes. The identity an attribute will be set for.
- attributeKey - string variable. An attribute key.
- attributeValue - string variable. An attribute value
- expire - unsigned 64-bit integer. Timestamp, milliseconds. Expiration date for an attribute. Attribute considered invalid after this date passed.

```
    pub extern "C" fn setAttribute(){
    let identity: AccountHash = runtime::get_named_arg("identity");
    let attributeKey: String = runtime::get_named_arg("attributeKey");
    let attributeValue: String = runtime::get_named_arg("attributeValue");
    let expire: u64 = runtime::get_named_arg("expire");
    if _identity_owner(identity) != runtime::get_caller(){
    runtime::revert(ApiError::PermissionDenied);
    }
    let now_blocktime: BlockTime = runtime::get_blocktime();
    let now: u64 = now_blocktime.into();
    if expire <= now {
    runtime::revert(ApiError::InvalidArgument);
    }
    let value: (String,String,u64) = (attributeKey,attributeValue,expire);
    let index: u64 = get_key(&attribute_length_key(&identity));
    let next_index: u64 = index+1;
    set_key(&attribute_key(&identity,index), value);
    set_key(&attribute_length_key(&identity), next_index);
    }
```

## Function revokeAttribute()
Function revokeAttribute() revokes an attribute by attribute index. Revoking essentially means that the expiration date of an attribute is changed to the current timestamp, thus the attribute becomes inactive.

Function has 2 parameters:
- identity - hash of casper account, which is represented by 32 bytes. An identity to revoke attribute from
- index - unsigned 64-bit integer. The index of an attribute to revoke.

```
    pub extern "C" fn revokeAttribute(){
    let identity: AccountHash = runtime::get_named_arg("identity");
    if _identity_owner(identity) != runtime::get_caller(){
    runtime::revert(ApiError::PermissionDenied);
    }
    let index: u64 = runtime::get_named_arg("index");
    let mut value: (String,String,u64) = get_key(&attribute_key(&identity, index));
    let now_blocktime: BlockTime = runtime::get_blocktime();
    let now: u64 = now_blocktime.into();
    value.2 = now;
    set_key(&attribute_key(&identity, index), value);
    }
```
