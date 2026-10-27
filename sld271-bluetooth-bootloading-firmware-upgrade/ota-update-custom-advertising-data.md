# Bluetooth OTA Updates Using Customized Advertising Data

## Introduction

When a device is booted into OTA DFU mode, the default advertising data used by the BLE stack is very minimal. Below is an example that was created by booting the ***Bluetooth - SoC Empty*** example (from SDK 3.1.1) into OTA mode:

```c
// raw advertising data
0x02010604094F5441081B00428928E20A68020A00
```

The advertising packet includes four elements:

- Flags (value 0x06)
- Complete local name ("OTA")
- BLE device address (00 (public) 428928E20A68)
- TX Power (0 dBm)

For details about decoding the advertising packet content, see the following page:

[Bluetooth advertising data basics](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-advertising-scanning/advertising-data-basics)

To implement a robust OTA update procedure, use a customized advertising data format, for example:

- Include a unique device name in the OTA advertising packets.

- Indicate the current SDK/stack version in the advertising packets.

These customizations are left for the application designer to decide. Silicon labs Bluetooth stack includes a few simple yet powerful options to customize the advertising packets used in OTA mode.

Since SDK v3.x, OTA related configurations are included in **`Bluetooth > OTA > OTA`** software component. By installing this component, you can achieve different kinds of custom OTA configuration options using the APIs explained below.

## Setting the Device Name in OTA Advertising

The device name to be used during OTA can be set using the run-time command `sl_bt_ota_set_device_name(name_len, name)`, where `name` is the chosen name to be used during OTA mode and `name_len` is the exact number of characters in the name string. The name is stored in the persistent store. The maximum name length is 17 bytes.

The device name used during OTA does not have to be static. The string can be dynamically generated while the Bluetooth stack is running, for example, based on the serial number of the device or some other value that uniquely identifies the device. The OTA name can be, for example, negotiated between the OTA client and the OTA target device over a Bluetooth connection.

## Setting OTA Flags

You can use the run-time command `sl_bt_ota_set_configuration(flags)` to set OTA flags. The setting is stored in the persistent store.
`flags` is a 32-bit unsigned integer variable. Flags are defined as follows.

**Bit 0: Advertising address**
0: use public address.
1: use static random address.

**Bit 1: Application update version check**
0: disable application version check.
1: enable application version check.

**Bits 2-31: reserved.**

`flags` value is given as a bitmask. Flags values are defined as follows.

- 0: use public device address and disable application version check.
- 1: use static random address and disable application version check.
- 2: use public device address and enable application update version check.
- 3: use static random address and enable application update version check.

Default value 0 is used if the user application does not set the flags, in which case the public device address is used and AppLoader does not perform any application version checking during OTA mode.

## Setting the OTA Advertising Packet Content Manually

It is also possible to define the whole content of the OTA advertising packets from the application using the `sl_bt_ota_set_advertising_data(packet_type, adv_data_len, adv_data)` API.

This method can be used to include the OTA service UUID in the advertising packets that are sent during OTA mode. There are basically no limitations on what advertising data can be used, as long as it conforms to the Bluetooth specification (for instance, a maximum of 31 bytes of advertising data can be set).

Setting the `packet_type` to value 2 indicates the data is intended for advertising packets. Whereas, a `packet_type` value 4 indicates the data is intended for a scan response packets.

The following code snippet shows how to set custom advertising packet used during OTA mode. In this example, a simple packet format is used that includes three advertising elements: a `flag`, `complete local name`, and one `manufacturer-specific` advertising element. The custom advertising element is used to expose the SDK version and Bluetooth address of the device. The SDK version can be retrieved from the boot event as shown in the code snippet.

```c
// struct type to store SDK version in the boot event.
typedef struct {
  uint16_t major;
  uint16_t minor;
  uint16_t patch;
  uint16_t build;
}vSDK_t;

typedef struct
{
  /* First AD element: Flags*/
  uint8_t len_flags;
  uint8_t type_flags;
  uint8_t val_flags;

  /* Second Advt element: name*/
  uint8_t len_name;
  uint8_t type_name;
  uint8_t name[5];

  /* Third Advt elemnt: custom data. Include SDK version*/
  uint8_t len_manuf;
  uint8_t type_manuf;
  uint8_t vSDK_Major_LO;
  uint8_t vSDK_Major_HI;
  uint8_t vSDK_Minor_LO;
  uint8_t vSDK_Minor_HI;
  uint8_t vSDK_Patch_LO;
  uint8_t vSDK_Patch_HI;
  uint8_t bt_address[6];

} tsCustomAdv;

....
case sl_bt_evt_system_boot_id:
  vSDK_t vSDK;
    vSDK.major = evt->data.evt_system_boot.major;
    vSDK.minor = evt->data.evt_system_boot.minor;
    vSDK.patch = evt->data.evt_system_boot.patch;
    vSDK.build = evt->data.evt_system_boot.build;

    // Build the advt packet and configures OTA Advt data.
    configure_OTA_adv(vSDK);
  ...
break;

...
// fill advt data and configure OTA advertisement data
void configure_OTA_adv(vSDK_t sdk)
{
  tsCustomAdv sData;
  sl_status_t sc;
  char* ota_name = "Silab";

  /* fill the first Advt element (flags) */
  sData.len_flags = 0x02;
  sData.type_flags = 0x01;
  sData.val_flags = 0x06;

  /* fill the second Advt element (complete local-name)*/
  sData.len_name = 0x06;
  sData.type_name = 0x09;
  memcpy(sData.name, ota_name, 5);

  /* fill the third Advt element (manufacture-specific) */
  sData.len_manuf = 1+6+6; // type + (major, minor,patch) + addr
  sData.type_manuf = 0xFF;
  sData.vSDK_Major_LO = sdk.major & 0xFF;
  sData.vSDK_Major_HI = (sdk.major >> 8) & 0xFF;
  sData.vSDK_Minor_LO = sdk.minor & 0xFF;
  sData.vSDK_Minor_HI = (sdk.minor >> 8) & 0xFF;
  sData.vSDK_Patch_LO = sdk.patch & 0xFF;
  sData.vSDK_Patch_HI = (sdk.patch >> 8) & 0xFF;

  /* Check our own BT address and add it to the advertising data */
 sc = sl_bt_system_get_identity_address(&pAddr, &addrType);
 sl_app_assert(sc == SL_STATUS_OK,
                    "[E: 0x%04x] Failed to get Bluetooth address i OTA config\n",
                    (int)sc);
  memcpy(sData.bt_address, pAddr.addr, 6);

  sc = sl_bt_ota_set_advertising_data(2, sizeof(tsCustomAdv), (const uint8_t*)&sData);
  sl_app_assert(sc == SL_STATUS_OK,
                     "[E: 0x%04x] Failed to set OTA advertising data\n",
                     (int)sc);

  return;
}
```

The structure **tsCustomAdv** defines custom advertising data packet content. It is formatted according to the Bluetooth specification. Each advertising element begins with a length indicator byte followed by the data type indicator and the actual data value bytes.

Function **configure_OTA_adv()** fills the advertising data content and then tells the stack to use this data during OTA mode by calling `sl_bt_ota_set_advertising_data` API. Most of the advertising data content in this example is static, except for the Bluetooth device address that is queried from the stack and the SDK version. The SDK version can also be defined statically by copying the content in the `sl_bt_version.h` file.

Below is raw OTA advertising data from the above OTA configuration:

```c
//raw custom OTA advertising data
0x020106060953696c61620dff030001000100d02d61CCCCCC
```

| Length  | Type                             | Value                                                     |
|---------|----------------------------------|-----------------------------------------------------------|
| 2       | 0x01 (flags)                     | `0x06` (LE General discoverable mode, BR/EDR not supported) |
| 6       | 0x09 (complete local-name)       | `0x53696C6162` (Silab)                                      |
| 13      | 0xFF (Manufacture specific data) | `0x030001000100` (sdk 3.1.1) `Addr:D02D61CCCCCC`              |

Note the byte order in the `manufacturing-specific data` (that is the data containing SDK version and BT address) is the least significant byte first.

OTA advertising data set, as shown above, is stored permanently in Persistent Storage. Therefore, data will remain even if you remove or disable the call to **configure_OTA_adv()**  in the application code. To remove the custom advertising data, it must be wiped from the persistent storage. One way to do this is to set a custom OTA advertising data with length of zero bytes as follows:

```c
// Remove previously configured OTA advertising content:
// This will reset the OTA advt data to default
uint8_t dummy;
sl_bt_ota_set_advertising_data(2, 0, &dummy);
```
