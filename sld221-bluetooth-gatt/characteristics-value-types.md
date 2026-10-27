
# Different Value Types of Characteristics

## Introduction

Bluetooth LE (BLE) is used for wireless communication, which is achieved by operations on characteristics.

### Characteristics

Characteristics are a GATT Profile concept, which defines **Client** and **Server** roles. Server holds the state or data and client can request the state or data. Below is the definition of server and client from the Bluetooth SPEC.

- **Client** is the device that initiates commands and requests towards the server and can receive responses, indications and notifications sent by the server.
- **Server** is the device that accepts incoming commands and requests from the client and sends responses, indications and notifications to a client.

The GATT Profile specifies the structure in which profile data is exchanged. This structure defines basic elements, such as services and characteristics, which are used in a profile.

#### Profile

A profile is the top level of the hierarchy, which is composed of one or more services necessary to fulfill a use case. A service is composed of characteristics or references to other services. Each characteristic contains a value and may contain optional information about the value. The service and characteristic and the components of the characteristic (i.e., value and descriptors) contain the profile data and are stored in Attributes on the server.

#### Service

A service is a collection of data and associated behaviors to accomplish a particular function or feature of a device or portions of a device. A service may include other primary or secondary services and/or a set of characteristics that make up the service.
GATT also specifies the format of data contained on the GATT server. Attributes, as transported by the Attribute protocol, are formatted as Services and Characteristics. Services may contain a collection of characteristics. Characteristics contain a single value and any number of descriptors describing the characteristic value.

#### Characteristic

A characteristic is a value used in a service along with properties and configuration information about how the value is accessed and information about how the value is displayed or represented. A characteristic definition contains a characteristic declaration, characteristic properties, and a value. It may also contain descriptors that describe the value or permit configuration of the server with respect to the characteristic value.

![GATT-Based Profile Hierarchy](resources/0304-f1.png?darkModeUrl=resources/0304-f1.png)

## Characteristic Value Types

The above section introduces how to achieve communication among BLE devices. This section describes the differences among the types of a characteristic value. In the [Blue Gecko Bluetooth Profile Toolkit Developer's Guide](/bluetooth/{build-docspace-version}/bluetooth-profile-toolkit-developers-guide), three types of values can be used: hex, utf-8 and user.

**Available Types**:

**hex**: The characteristic value is a hexadecimal value. In the next figure, the initialization value has the format of 0xXXXX, the characteristic value has the length of 2 bytes, and is initialized with the value 0x1122.

![Hexadecimal Type](resources/v3-hex-char.png?darkModeUrl=resources/v3-hex-char.png)

**utf-8**: The characteristic is a string and it takes a string as its initialization value. In this example, the characteristic has 13 bytes length and is initialized with the string value "Empty Example".

![UTF-8 Type](resources/v3-string-char.png?darkModeUrl=resources/v3-string-char.png)

**user**: When the characteristic type is marked as type="user", the application has to initialize the characteristic value and also provide it when a read operation occurs, for example. The Bluetooth stack does not initialize the value nor automatically provide the value when it's being read. When this is set, the Bluetooth stack generates _gatt_server_user_read_request_ or _gatt_server_user_write_request_, which must be handled by the application.

**Default**: utf-8

## Comparison

The type "**hex**" and "**utf-8**" are the same in terms of user operations, but are not of the same length. The section below lists the differences between type "**user**" and "**hex**" to help users in choosing the right property of a characteristic value. The most significant difference between the type “user” and the other two types is where the data is stored.

### **Type – hex or utf-8**

If you use the property other than "user", the stack will allocate and maintain the buffer for the characteristic value automatically.

- When a read request is received, the read response will be filled with the characteristic value and sent back to the peer device by the stack without user involvement.

- When a write request is received, the stack will modify the characteristic value and send the write response to the peer device, then generate a `sl_bt_evt_gatt_server_attribute_value` event to inform users about the new value.

- When sending notification/indication, the characteristic value has to be read from the stack, then the read value needs to be placed into the API `sl_bt_gatt_server_send_notification` or `sl_bt_gatt_server_send_indication`.

- To read or write the characteristic value locally, use the below 2 APIs.

```c
sl_bt_gatt_server_read_attribute_type(uint16_t attribute,
                                      size_t max_type_size,
                                      size_t* type_len,
                                      uint8_t* type)

sl_bt_gatt_server_write_attribute_value(uint16_t attribute,
                                       uint16_t offset,
                                       size_t value_len,
                                       const uint8_t* value)
```

The following figures show the flow of typical GATT operations when using hex characteristic value type.

![Hex Type - GATT Read](resources/0304-f4.png?darkModeUrl=resources/0304-f4.png)

![Hex Type - GATT Write](resources/0304-f5.png?darkModeUrl=resources/0304-f5.png)

![Hex Type - GATT Notification/Indication](resources/0304-f6.png?darkModeUrl=resources/0304-f6.png)

### Type – user

If you use "user" as the value type, the value of the characteristic is stored in the **application layer**, which means the users should be responsible for allocating, maintaining, and freeing a suitable buffer for the characteristic value. Additionally, respond to the GATT write/read requests by sending write/read response back to the peer device via below APIs.

```c
sl_bt_gatt_server_send_user_read_response(uint8_t  	connection,
                                          uint16_t  characteristic,
                                          uint8_t  	att_errorcode,
                                          size_t  	value_len,
                                          const uint8_t*  	value,
                                          uint16_t* sent_len);

sl_bt_gatt_server_send_user_write_response(uint8_t connection,
                                           uint16_t characteristic,
                                           uint8_t  att_errorcode);
```

For example, if you have a characteristic with length 60, allocate a 60-byte buffer in the application layer to store its value.

- When a read request is received, a `sl_bt_evt_gatt_server_user_read_request` event will be generated from the stack to the application and you need to respond the characteristic value via `sl_bt_gatt_server_send_user_read_response`.

- When a write request is received, `sl_bt_evt_gatt_server_user_write_request` event is generated. At this point, users decide how to handle the value with the local buffer against the data in write request and respond it via `sl_bt_gatt_server_send_user_write_response`.

- When sending notification/indication, the application buffer can be directly put into the API `sl_bt_gatt_server_send_notification` / `sl_bt_gatt_server_send_indication`

The following figures show the flow of typical GATT operations when using hex characteristic value type.

![User Type - GATT Read](resources/0304-f7.png?darkModeUrl=resources/0304-f7.png)

![User Type - GATT Write](resources/0304-f8.png?darkModeUrl=resources/0304-f8.png)

![User Type - GATT Notification/Indication](resources/0304-f9.png?darkModeUrl=resources/0304-f9.png)

**Notes:**

1. **DO NOT** use the APIs `sl_bt_gatt_server_read_attribute_value(...)` and `sl_bt_gatt_server_write_attribute_value (...)` to read or write a characteristic value whose type is "user", because the data buffer is in your application code and stack will never know its value.

2. **DO NOT** forget to respond to the read and write requests if the characteristic value type is “user”. Otherwise, no other GATT operation can be issued until GATT timeout - typically 30 seconds.

## Conclusions and Suggestions

Using "user" and using "hex"

- Type hex/utf-8

  - Advantage – Stack will handle the read and write request to the characteristics, which is more effective and faster.
  - Disadvantage – Users can't access the characteristic buffer directly. Stack APIs are needed to read and write the value locally.

- Type user

  - Advantage – The buffer storing the characteristic value is in the application layer, which means it’s easier to read and modify the value.
  - Disadvantage – Users must respond to the read and write requests by calling stack APIs. If users don’t respond, no other GATT operation can be issued until GATT timeout. Because every GATT operation requires involving the application, it could be less effective and consume more power. Additionally, long-write, reliable-write and read-multiple-values requests are **NOT** supported if a characteristic is user type.

## Code Example

This guide has a related code example, here: [Different Characteristic Value Type Example](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/gatt_protocol/using_characteristics_value_types).
