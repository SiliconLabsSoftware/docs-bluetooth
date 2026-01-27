# NVM3 API Options

This chapter describes the three different APIs available to access NVM3 objects.

- Token API

- Persistent Store API

- Native NVM3 API

## Token API

The token API is used to access data stored in SimEEv1 and SimEEv2 with the EmberZNet and Connect stacks, as well as multiprotocol applications. Information on how to define and access tokens can be found in [Using Tokens for Non-Volatile Data Storage](https://docs.silabs.com/gecko-platform/latest/using-tokens-for-non-volatile-storage/), and users should read this document before using the token API. When selecting the NVM3 Library plugin instead of one the SimEE plugins, the NVM3 default instance is used to store the token data instead of SimEE. The same token API can still be used to access tokens stored in NVM3, but the token definition needs some modifications to work with NVM3, as described below.

When defining a token to be used with SimEE, a creator code must be defined as an identifier for the token. Similarly, when defining a token to be used with NVM3, an NVM3 key must be defined for the token. A token definition that is compatible with both NVM3 and SimEE would include both an NVM3 key and a creator code and look like this:

```sh
#define CREATOR_name 16bit_value
#define NVM3KEY_name 20bit_value
#ifdef DEFINETYPES
  typedef data_type type #endif
#ifdef DEFINETOKENS
DEFINE_*_TOKEN(name, type, ... ,defaults) #endif
```

Select a 20-bit NVM3 key for the token, according to the domains in [NVM Default Instance Key Space](./02-nvm3-default-instance.md#nvm3-default-instance-key-space). Each token must have a unique NVM3 key, except for indexed tokens, where more NVM3 keys must be reserved as outlined in [Special Considerations for Indexed Tokens](#special-considerations-for-indexed-tokens).

### Deleting Tokens

As tokens are created at compile time, they cannot be created or deleted at run time. NVM3 objects are, however, created and deleted at run time, and the token initialization function creates NVM3 objects for each defined token if they do not already exist. The token initialization generally does not delete NVM3 objects found that do not have a corresponding token associated with them. Therefore, if a token is no longer included in an application, the application should manually delete the associated NVM3 object by using the NVM3 Native API described in [Native NVM3 API](#native-nvm3-api). For indexed tokens, however, the token initialization checks if indexed tokens have more or less indexes than the number of NVM3 objects found in the indexed token's NVK3KEY range. If there are fewer indexes, the token initialization deletes the extra NVM3 objects. If the number of indexes has been increased, new NVM3 objects will be created to hold these indexes.

When NVM3 objects are deleted, the actual object data remains in NVM3 but is marked as deleted. The deleted object data remains in NVM3 and consumes cache space until NVM3 repacks have erased the page(s) holding all versions of these objects.

### Special Considerations for Indexed Tokens

NVM3 does not have native support for indexed tokens. Therefore an extra requirement is imposed on the NVM3 key selection for indexed tokens. With NVM3, indexed tokens are implemented by storing each index in a separate object, starting with index 0 stored at the defined NVM3KEY_name key value and the last index (127) stored with key NVM3KEY_name + 127. Because of this implementation, 128 NVM3 keys must be reserved for each indexed token. The user still only defines one NVM3KEY_name key value, but no other tokens should be defined with key values in the 127 values following this defined key. Even if the token is defined with fewer than 128 indices, all 128 indices should be reserved as the token might be expanded with more indices later on.

The example below shows two indexed tokens defined in the user key domain:

```sh
// This key is used for an indexed token and the subsequent 0x7F keys are also reserved #define
NVM3KEY_MY_INDEXED_TOKEN_A 0x00000
// This key is used for an indexed token and the subsequent 0x7F keys are also reserved #define
NVM3KEY_MY_INDEXED_TOKEN_B 0x00080
```

The table below provides indexed token NVM3 key selections in the example.

| **NVM3KEY** | **NVM3 Objects Contents** |
|-------------|---------------------------|
| 0x00000 | Reserved for TOKEN_MY_INDEXED_TOKEN_A index 0 |
| 0x00001 | Reserved for TOKEN_MY_INDEXED_TOKEN_A index 1 |
| 0x00002 | Reserved for TOKEN_MY_INDEXED_TOKEN_A index 2 |
| … |  |
| 0x0007F | Reserved for TOKEN_MY_INDEXED_TOKEN_A index 127 |
| 0x00080 | Reserved for TOKEN_MY_INDEXED_TOKEN_B index 0 |
| 0x00081 | Reserved for TOKEN_MY_INDEXED_TOKEN_B index 1 |
| … |  |
| 0x000FF | Reserved for TOKEN_MY_INDEXED_TOKEN_B index 127 |

## Bluetooth NVM API

The Bluetooth NVM API that was originally designed for PS Store can be used in the same way when using NVM3 as when using PS Store. The Bluetooth stack will automatically translate its NVM API calls to PS Store API calls or to NVM3 API calls in the background, depending on what components the project uses. The same applies for Zigbee + Bluetooth DMP projects, where NVM3 is applied as the storage mechanism, but the Bluetooth NVM API can still be used. The Bluetooth API is documented in the Bluetooth API Reference Manual. In Bluetooth SDK v2.x it can be found under the *Flash* class (commands starting with `gecko_cmd_flash_`), while in Bluetooth SDK v3.x it can be found under the *NVM* class (commands starting with `sl_bt_nvm`).

16-bit keys are used with the Bluetooth NVM API, which are then mapped to a 20-bit NVM3 key when NVM3 is used as storage mechanism. The four most significant bits are set to 0x4 to place these objects in the Bluetooth domain of the NVM3 default instance key space. As the Bluetooth NVM API is fixed to use only the Bluetooth domain, any objects to be placed in other domains, for example the User domain, must be created and accessed using the native NVM3 API.

If you want to use the Bluetooth NVM API and the native NVM3 API in the same app, then:

1. Call `gecko_init(pconfig)` to initialize the Bluetooth stack and open its own NVM3 instance. This is done in all Bluetooth sample apps.

2. Open NVM3 by calling the `nvm3_open()` function with the default (!) parameters to open your NVM3 instance:

   `nvm3_open(nvm3_defaultHandle, nvm3_defaultInit);`

User data can now be saved to:

- PS key range 0x4000 - 0x407F. All other PS keys (0x0000-0xFFFF) are reserved for the stack (for example for storing bonding data).

- NVM3 key range 0x00000-0x0FFFF (NVM3 user data area), and 0x44000-0x4407F (PS Store user data area).

For example, the following API calls will have the same effect (writing to the same area):

- `nvm3_writeData(nvm3_defaultHandle,0x44000,(void\*)data,len);`

- `gecko_cmd_flash_ps_save(0x4000,len,data); //in Bluetooth SDK v2.x`

- `sl_bt_nvm_save(0x4000,len,data); //in Bluetooth SDK v3.x`

Similarly, you can read the same data with the following API calls:

- `nvm3_readData(nvm3_defaultHandle,0x44000,(void\*)read_buffer,maxlen);`

- `gecko_cmd_flash_ps_load(0x4000); //in Bluetooth SDK v2.x`

- `sl_bt_nvm_load(0x4000,maxlen,&read_len,(uint8_t\*)read_buffer); //in Bluetooth SDK v3.x`

## Native NVM3 API

For code accessing NVM3 objects that does not need to be compatible with the token or PS Store APIs, using the native NVM3 API to access NVM3 data is recommended to reduce code size and allow using the full feature set of NVM3. Any PS Store object or token can also be accessed through the native NVM3 API using the same NVM3 key. Complete documentation of this API is found online in the Drivers section of the [Gecko Platform Developer Documentation](https://docs.silabs.com/gecko-platform/latest/driver/api/group-nvm3). If you are using an earlier version of the GSDK, search that page's version history for the corresponding release.
