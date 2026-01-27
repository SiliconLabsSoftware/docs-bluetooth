# GATT Operation Flowcharts

## Introduction

Bluetooth Low Energy (BLE) defines a framework for a wide variety of communication schemes. It allows devices to discover each other, broadcast data, establish connections, and many other fundamental operations.

The objective of this page is to focus on the procedures defined for the "Generic Attribute Profile" (GATT) of the BLE stack.

Sequence diagrams are used to describe these procedures comprehensively, focusing on the following items:

* BGAPI function calls.
* Messages exchanged over the air.
* Events raised by the BLE stack.

This content does not expose the Bluetooth stack packet management nor does it describe the host controller interface (HCI).

This content assumes that the necessary Bluetooth hardware is used, such as an EFR32 SoC or a BGM module.

## Generic Attribute Profile (GATT) Operations

GATT provides a framework for all profiles defined either by the Bluetooth SIG or by the user. Bluetooth profiles are implemented using a hierarchical structure:

* Services: A collection of GATT entries, grouping together attributes that are related to each other.
* Characteristics: Data containers. A characteristic consists of a *declaration* (a label) and a *value*.
* Descriptors: Placed under a characteristic, they provide additional information about the characteristic and its values.

```bash
GATT database
*--> Services
        |
        --> Characteristics (always placed under a service)
                  |
                  --> Descriptors (always placed under a characteristic)
```

The BLE specification refers to all the attributes within a single service as the *service definition*.

### Primary Service Discovery

A primary service is the standard type of GATT service that includes relevant, standard functionality exposed by the GATT server. In other words, a service is a collection of characteristics. As a result, when a connection is established, it is necessary for the GATT client to be able to discover the available services.

![GATT Service Discovery](resources/gatt-service-discovery.png?darkModeUrl=resources/gatt-service-discovery.png)

The procedure iterates through all available services of the GATT database, an `sl_bt_evt_gatt_service` event is generated for each primary service discovered. After the end of the list is reached, an `sl_bt_evt_gatt_end_procedure` is raised by the stack.

After a client has obtained the handle range for a service, a similar procedure exists to retrieve a full list of the characteristics under that service.

### Characteristics

As mentioned earlier, characteristics are data containers. They consist of a *declaration* (a label) and a *value*.

There are multiple types of characteristics:

* 'hex' or 'utf-8' characteristics, which consist of hex or utf-8 values maintained internally by the BLE stack.
* 'user' characteristics, which are maintained at the application level. In other words, it is the  responsibility of the application to perform the appropriate actions when a Read/Write command is received.

Moreover, on a given characteristic, different types of read/write operations can be performed. The following procedures are discussed here:

* Read/Write on user characteristic.
* Long read/write on user characteristic.
* Notifications and Indications

The following sub-procedures are not discussed:

* Read multiple characteristics
* Write without response
* Signed write without response
* Reliable writes

For more information on these procedures, see the BLE [core specification](https://www.bluetooth.com/specifications/bluetooth-core-specification) and [Different Characteristic Value Types](./characteristics-value-types).

#### Read/Write Hex and utf-8 Type Characteristics

The BLE stack manages the read and write operations for these characteristics. Upon reception of the "Read request", the server's stack sends back a "Read response" containing the characteristic value, as shown below:

![Read response](resources/read-hex-utf.png?darkModeUrl=resources/read-hex-utf.png)

The following sequence diagram describes the write operation:

![write sequence diagram](resources/write-hex-utf.png?darkModeUrl=resources/write-hex-utf.png)

The "Write Response" contains only an error code indicating whether the write was successful or not.

Note that if the maximum length of the payload is reached, the read/write long characteristic is automatically used by the BLE stack.

#### Read User Characteristics

The "Read Characteristic Value" procedure is implemented via the `sl_bt_gatt_read_characteristic_value()` BGAPI command. In this procedure, the characteristic length must be less or equal to the maximum payload, that is (ATT_MTU-1) bytes. In other words, the length of the characteristic has to fit in one "Read Response".

![Read Characteristics](resources/read-characteristic.png?darkModeUrl=resources/read-characteristic.png)

Data is sent from the server to the client via the "Read Response". Then, the client application can retrieve the data via the "evt\_gatt\_characteristic_value" event.

If the characteristic does not fit in one "Read Response", the "Read Long Characteristic Value" procedure implemented via the `sl_bt_gatt_read_characteristic_value()` routine can be used. If the payload is bigger than (ATT_MTU-1), the stack will automatically pack and send the data using `sl_bt_gatt_read_characteristic_value_from_value()`. It is not the responsibility of the application to call `sl_bt_gatt_read_characteristic_value_from_value()`. Instead, the application will still use the `sl_bt_gatt_read_characteristic_value()` routine. The following sequence diagram describes the "Read Long Characteristic Value" mechanism using the offset routine.

![Read long characteristics](resources/read-long-characteristic.png?darkModeUrl=resources/read-long-characteristic.png)

The client reads the characteristic by chunks of (ATT_MTU-1) bytes. The client stack keeps and updates the byte offset used for the read request.

For more extensive description on long characteristic operations, see the example: [Working with Long Characteristic Values](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/gatt_protocol/working_with_long_characteristic_values).

#### Write User Characteristics

The "Write Characteristic Value" procedure is implemented via the `sl_bt_gatt_write_characteristic_value()` BGAPI command. In this procedure, the characteristic length must be less or equal to the maximum payload, that is (ATT_MTU-3) bytes. In a similar way to the "Read Characteristic", the length of the characteristic has to fit in one "Write Request".

![Write characteristic](resources/write-characteristic.png?darkModeUrl=resources/write-characteristic.png)

For longer characteristics, the same command can be used to write a characteristic in a remote GATT database. If the payload does not fit in one request, the "write long" procedure is automatically used when `sl_bt_gatt_write_characteristic_value()` is called. The value is buffered on the server side until all data is sent over. Upon reception by the server of all data, the client can issue a "write request execute" to trigger the effective write in the remote GATT database.

![Write long characteristic](resources/write-long-characteristic.png?darkModeUrl=resources/write-long-characteristic.png)

If the operation is successful, an `sl_bt_evt_gatt_procedure_completed` is sent by the server.

For more extensive on long characteristic operations, see the example: [Working with Long Characteristic Values](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/gatt_protocol/working_with_long_characteristic_values)

#### Notifications and Indications

Server-initiated updates can be sent from the GATT server to the client without the client having to request it. This has the advantage to save both power and bandwidth. Notifications and Indications are two types of server-initiated updates. In both cases, the procedure has to be enabled first on the remote GATT server using the `sl_bt_gatt_set_characteristic_notification()` beforehand.

Notifications includes the handle of the characteristic (i.e., its identifier) and the value. The client receives the notifications but does not send any acknowledgment back to the server.

![notifications](resources/notifications.png?darkModeUrl=resources/notifications.png)

The indications, on the other hand, have the same behavior but require an explicit acknowledgment from the client in form of confirmation. If the confirmation is not sent by the client, the server will not send further indications.

![indications](resources/indications.png?darkModeUrl=resources/indications.png)

The indications uses the same attribute protocol feature as the notifications.
