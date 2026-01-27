# Bluetooth GATT Protocol

These pages provide information about the following topics:

- [**Server and Client Roles**](./gatt-server-client-roles): Reviews information needed for an effective custom profile implementation, such as how a BLE connection works, what roles are played by the devices involved, and how data is transferred from one device to the other over the air.
- [**Acknowledged vs. Unacknowledged GATT Operations**](./acknowledged-vs-unacknowledged): Explains the difference between acknowledged and unacknowledged GATT operations.
- [**Polymorphic GATT**](./polymorphic-gatt): Describes the Polymorphic GATT feature, which can be used to dynamically show or hide GATT services and characteristics.
- [**GATT Caching**](./gatt-caching): Explains how to use the GATT Caching feature so that every remote device can make sure it is storing the latest version of the GATT database structure.
- [**Service Change Indications**](./service-change-indication): Discusses using the service change characteristic to notify the client to rediscover services on a change in the GATT server.
- [**Different Value Types of Characteristics**](./characteristics-value-types): Discusses profiles, services, and characteristics, and describes the uses for different characteristic value types.
- [**GATT Operation Flowcharts**](gatt-flowcharts): Provides sequence diagrams and discussion of the procedures defined in the "Generic Attribute Profile" (GATT) of the BLE stack.
