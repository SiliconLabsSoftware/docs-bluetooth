# Bluetooth Advertising Data Basics

## Introduction

This document explains the basics of BLE advertising packet formatting to allow users to quickly learn how to “decode” the content of an advertising packet.

## Advertising Data Format

When a BLE device is advertising, it periodically transmits packets, which contain information such as the preamble, access address, CRC, Bluetooth sender address, and so on. Application developers are often interested in the **advertising payload** that is **0-31 bytes long** **on the primary channels** because it is controlled by the application. For extended advertising, the maximum length is 1650 bytes (non-connectable - for connectable maximum 191 bytes of data can be set, see `sl_bt_extended_advertiser_set_long_data()`), but advertising parameters may limit the amount of data that can be sent in a single advertisement. Other fields in the advertising packets are automatically filled by the Bluetooth stack.

In this document, the term **advertising data** refers to the 0..31 byte long payload that is available for application use. (In Bluetooth Core specification this field is referred to as AdvData).

![ADV PDU](resources/adv-pdu.png?darkModeUrl=resources/adv-pdu.png)

Advertising data consists of one or more **Advertising Data (AD) elements**. Each element is formatted as follows:

- 1st byte: **length** of the element (excluding the length byte itself)
- 2nd byte: **AD type** – specifies what data is included in the element
- **AD data** – one or more bytes - the meaning is defined by **AD type**

For the possible AD type values, which are listed in the Bluetooth SIG website, see the following link:
https://www.bluetooth.com/specifications/assigned-numbers/generic-access-profile

Some of the most commonly used data types are:

**0x01** = Flags

**0x03** = Complete List of 16-bit Service Class UUIDs

**0x09** = Complete Local Name

**0x08** =   Shortened Local Name

The following are real-world examples of the advertising data format.

**Example 1:**

The first example shows how to decode the advertisement data sent by the **Thermometer Example**. The payload consists of 28 bytes that are seemingly random but are simple to decode into advertising elements, as shown below.

![Example 1](resources/sld268-example1.png?darkModeUrl=resources/sld268-example1.png)

In the example shown above, the raw payload is split into three AD elements. The first byte is always the length indicator, which makes it easy to find the AD element boundaries.

The first element is flags. The payload is just one byte long, which means that there are up to 8 flags that can be set. In this example, two flags are set (bit positions 1 and 2) and the meaning is:

**Bit 1**: “LE General Discoverable Mode”

**Bit 2**: “BR/EDR Not Supported.”

The second element includes a list of adopted services (16-bit UUID). In this case, only one service (Health Thermometer, UUID 0x1809) is advertised. Note the reversed byte order (multibyte values in BLE packets are in little-endian order). A complete list of the adopted service UUID values can be found at:
[https://www.bluetooth.com/specifications/assigned-numbers/](https://www.bluetooth.com/specifications/assigned-numbers/)

The third element contains the device name *Thermometer Example*. This is the name that is displayed if you, for example, try to scan the device with your smart phone.

**Example 2:**

![Example 2](resources/sld268-example2.png?darkModeUrl=resources/sld268-example2.png)

In the second example, the advertising payload has the maximum length i.e., 31 bytes. It is split up into individual AD elements the same way as in the first example.

The first element is the flags byte, which has the same value as in the first example.

The second element has AD Type set to 0x07, which means *Complete List of 128-bit Service Class UUID*. In this case, the device is advertising the 128-bit UUID that has been allocated for a custom service. Again, note that the byte order is reversed.

The third element is the device name. The device name defined in the project is *"BGM111 SPP server"*. Because the advertising payload doesn't have enough space to fit the complete name, the name is truncated. The AD Type 0x08 is used to indicate that this is a shortened name. Only the first 8 characters of the name can fit in the advertising data to meet the 31-byte size limit.

## Setting Advertising Data

There are two possible ways to set the advertising data content using the Silicon Labs Bluetooth SDK.

- Let the stack fill the advertising data automatically, based on the GATT content
  - `sl_bt_legacy_advertiser_generate_data()`
  - `sl_bt_extended_advertiser_generate_data()`
- The application can set the advertising data content directly
  - `sl_bt_legacy_advertiser_set_data()`
  - `sl_bt_extended_advertiser_set_data()`
  - `sl_bt_extended_advertiser_set_long_data()`

The first option is the simplest one. The stack will automatically fill the advertising data content based on the services defined in the GATT database of the application. It is possible to select which services are included in the advertising packets by using the **Advertise service** switch in the Bluetooth GATT Configurator or the **advertise** attribute in gatt_configuration.btconf.

The logic of how the advertising data is automatically filled is described in the API Reference Manual at the command `sl_bt_legacy_advertiser_generate_data()`.

The second option to set advertising data is more flexible because it allows the application to have full control over data that is included in the advertising payload. This is especially useful if the application needs to advertise manufacturer-specific data. Note that a dedicated AD Type 0xFF is reserved for proprietary data. Advertisement data can be set with `sl_bt_legacy_advertiser_set_data()`, `sl_bt_extended_advertiser_set_data()` and `sl_bt_extended_advertiser_set_long_data()`. Note, that one of the `generate_data()` or `set_data()` API has to be called before starting advertising with either  `sl_bt_legacy_advertiser_start()` or `sl_bt_extended_advertiser_start()`.

Note that even when using custom data the AD elements must be formatted according to the Bluetooth specification. Using custom advertising data is beyond the scope of this document and is discussed separately.

## Conclusion

This document is a **quick introduction** to BLE advertising. Below are resources for more detailed information on the topic:

[**List of AD Types**](https://www.bluetooth.com/specifications/assigned-numbers/generic-access-profile)

[**List of adopted services**](https://www.bluetooth.com/specifications/gss)

[**Bluetooth API reference manual**](https://docs.silabs.com/bluetooth/latest/bluetooth-start)

[**Bluetooth Core Specification**](https://www.bluetooth.com/specifications/specs) - This is the golden reference, but not that easy to digest.

## Example

This guide has related code examples here:

- [Advertising Manufacturer Specific Data](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/advertising/advertising_manufacturer_specific_data)
- [Advertisement or Scan Response Constructor](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/advertising/adv_or_scan_response_constructor)
