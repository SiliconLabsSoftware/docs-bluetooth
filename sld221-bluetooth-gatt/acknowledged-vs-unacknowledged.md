# Acknowledged vs Unacknowledged GATT operations

This page explains the difference between acknowledged and unacknowledged **GATT operations** in Bluetooth Low Energy.

The figures below show data flows between the **GATT client** and **GATT server** for a write operation **without** and **with** a response (**unacknowledged** and **acknowledged**).

![write without a response](resources/write-without-response.png?darkModeUrl=resources/write-without-response.png)

![write with a response](resources/write-with-response.png?darkModeUrl=resources/write-with-response.png)

Terms *acknowledged* and *unacknowledged* apply to **GATT operations** because they describe both data transmitted across the radio link and also data reaching the GATT/Application layer. On both acknowledged and unacknowledged GATT operations, data is **reliably transported** across the **radio link**. The difference between the two transfer types is in knowing if and when data sent has been **received by the application** on the other side.

For example, the response to write operations on *'user'*-type characteristics must be managed by the **application**. If the application can't process the write request, a response will not be sent and a timeout will occur after 30 seconds, indicating to the client that it wasn't possible to accept the write request. Applications which need synchronous operation and availability of other resources, such as UART in a cable replacement application, are required to use acknowledged operations.

Notifications and indications are another example of unacknowledged and acknowledged operations, respectively. Data flow is shown below.

![notification data flow](resources/notification.png?darkModeUrl=resources/notification.png)

![indication data flow](resources/indication.png?darkModeUrl=resources/indication.png)

To summarize, in both unacknowledged and acknowledged data transfers, data is reliably transported across the radio link but in the former there is no guarantee of delivery at the GATT/Application layer.
