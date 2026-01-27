
# Service Change Indications

## Background

Attribute Caching is a mechanism of caching attribute handles, which allows clients to avoid the process of rediscovering services upon consecutive connections with paired and bonded devices. This saves power, time, and reduces the packet size.

## Problem

If the client caches attribute handles and the values change, it can lead to unpredictable values from those handles. This can happen when you add or remove characteristics in run time or between firmware updates (see the [Polymorphic GATT](./polymorphic-gatt) feature). The issue is how to notify the client to rediscover services upon a change in GATT server.

For more details about how Android platform and iOS behave and implements this, [see this article](https://punchthrough.com/pt-blog-post/attribute-caching-in-ble-advantages-and-pitfalls/).

## Solution

The standard **Generic Attribute Service** includes a characteristic called **Service Changed**, which can be used by the client to subscribe for service change indications and by the server to send out service change indications (BLUETOOTH CORE SPECIFICATION Version 5.4 | Vol 3, Part G, Section 7.1). The value sent out in the indication contains the range of handles which have been changed. This can help the client to rediscover that range of handles.

If the database has a Generic Attribute Service and Service Changed characteristic, the stack will monitor local database changes and will send out Service Changed indications for those clients that have subscribed for it.

Note that, by default, the Service Changed subscription is only valid for the lifetime of a connection. After the connection is closed, the subscription is removed. Upon the next connection, the client has to subscribe again. This also implies that if the GATT database changes between two connection, then the client will not be notified about this change. To avoid this, the two devices (client and server) have to create a bonding. Subscriptions of bonded devices are permanently stored, and after the client reconnects to the server, it will be notified about changes that happened since the last connection.

## Implementation

Unlike other services and characteristics, the Generic Attribute Service is not directly visible in the GATT configurator. For any Bluetooth project in Simplicity Studio, open the *Bluetooth GATT Configurator* and click on the profile in the custom GATT view. You can enable or disable the Generic Attribute Service using the slider, as shown in the image below. It is enabled by default for examples that come with the SDK.

![Enabling SCI](resources/v3-Enabling-SCI.png?darkModeUrl=resources/v3-Enabling-SCI.png)
