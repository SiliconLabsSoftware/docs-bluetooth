# Generic Access Profile (GAP)

The Generic Access Profile or the GAP is one of the first layers every Bluetooth LE developer gets exposed to. This is because the GAP is used to control how a device is visible and connectable by other devices and also how to discover and connect to remote devices.

To put this simply, the GAP provides access to the link layer operations described in section [](04-link-layer.md#), which are related to the device discovery, connection establishment and termination, and connection timing control.

GAP defines device roles that provide specific requirements for the underlying controller. Roles allow devices to have radios that either transmit (TX) only, receive (RX) only, or do both.

- **Broadcaster (TX only)**: Sends advertising events and broadcast data.

- **Observer (RX only)**: Listens for advertising events and broadcast data.

- **Peripheral (RX and TX)**: Always peripheral, is connectable and advertising. Designed for a simple device using a single connection with a device in the Central role.

- **Central (RX and TX)**: Always central, never advertises. Designed for a device that is in charge of initiating and managing multiple connections.

A device can support more than one role, but only one role can be adopted at a given time.

GAP also defines modes and procedures for discovery, connection, and bonding. The terminology is the same for Bluetooth LE and BR/EDR, although underlying technology can differ.

Modes:

- **Connectable**: Can make a connection. State: Non-connectable, connectable.

- **Discoverable**: Can be discovered (is advertising). State: None, limited, general.

- **Bondable**: If connectable, will pair with connected device for a long-term connection. State: Non-bondable, bondable.

Procedures:

- **Name discovery**: Go into a menu and find the name of the other device. The name is shared with BR/EDR in a dual-mode device.

- **Device discovery**: Search for devices that are available for connection.

  - Find address and name of devices.

  - Define device role.

- **Link establishment**: After selecting an advertising device, connect to it.

  - Instruct Link layer to send a CONNECT_REQ.

  - Perform service discovery.

  - Request device authentication (not data authentication).

  - Request use of services.

- **Service discovery**: Used by devices in Central and Peripheral roles to find services available on peer devices.
