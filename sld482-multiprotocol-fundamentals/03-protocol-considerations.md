# Protocol Considerations

It is important to recognize that a single radio cannot receive or transmit two packets for two different protocols simultaneously. To share a single radio, both protocols have to accept that they will not have 100% use of the radio. They must therefore be able to survive loss of radio without significantly decreasing performance or losing application messages. In this case, *significantly* means that the application is working outside of its intended operational parameters. Examples are higher than acceptable message latency or message loss.

While some protocols and applications will have strict timing requirements for packet transmission, in most cases the critical area of concern with multiprotocol is the receipt of incoming packets from the networks.

For a protocol to be a good candidate for a dynamic or concurrent multiprotocol implementation, the following considerations must apply:

- Neither protocol can take over the radio for long periods (longer than a few milliseconds at a time) under normal operating conditions, excluding commissioning, firmware upgrades, and other related operations.

- One or both protocols must have a robust mechanism for managing loss of incoming packets, such as MAC retries.

- One or both protocols must have very short packets and/or a short time required on the radio.

- For dynamic multiprotocol, one or both protocols must implement strict time slots/connection intervals.

>**Note**: When using dynamic multiprotocol, a radio has to switch between two different PHYs and it 'disappears' from one network or another. For protocols where the device might be the 'parent' of a sleepy end device, if the parent device is not available when the sleepy end device wakes up to send a message, regular dependency on retries will impact the sleepy end device’s battery life.

>**Note**: If you have a protocol that uses multiple radio configurations, you do not need to implement multiprotocol. For example, Silicon Labs' RAIL and the radio configurator support multiple radio configurations in the same protocol. See *AN971: EFR32 Radio Configurator Guide* for more details. Multiprotocol is used if protocols have mostly independent protocol stacks.

## Dynamic Multiprotocol Example

Bluetooth LE and Zigbee are suitable protocols for a dynamic multiprotocol implementation. Because of the low duty cycle for Zigbee traffic and the retry mechanisms in the Zigbee networking stack, a Zigbee Router can switch its radio to some other frequency or protocol for short periods without dropping any messages at an application level.

Bluetooth LE radio usage can be predicted and planned in advance. Bluetooth beacons are quite short packets, usually less than 30 bytes. The radio only needs about 1 ms to transmit the beacon, and the interval between beacons is typically no shorter than 100 ms, thus providing a duty cycle of just 1%. This means that the radio can devote at least 99% of its time to the main Zigbee network.

If you are using Silicon Labs products, adding Bluetooth LE to most proprietary networks is also possible because of the short Bluetooth LE packets and the long delay between communications. The proprietary stack must be updated to work with the Bluetooth stack, and all proprietary communications should be examined to determine the priority and required timing accuracy of it. For more details, see [Dynamic Multiprotocol User’s Guide](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ug/) and [Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ble-proprietary-on-rail/).

## Concurrent Multiprotocol Example

Zigbee and Thread are examples of suitable protocols for a concurrent multiprotocol implementation.

The multiprotocol device can control which IEEE 802.15.4 channel it is operating on to connect to other Zigbee and Thread devices. It only has a single IEEE 802.15.4 MAC/PHY to listen to or send on both networks concurrently, meaning that switching radio PHYs is not required.

The multiprotocol device manages incoming packets from either network by filtering both PAN IDs, directing to the appropriate networking stack.

Operational constraints include:

- The multiprotocol device must control selection of the same IEEE 802.15.4 channel for both Zigbee and Thread, which means that it is most likely a Zigbee Coordinator/Thread Leader, in practice the Gateway/Controller in both networks. If using Concurrent Listening on an EFR32xG21 or EFR32xG24, this constraint does not apply.

- The device cannot be required to receive Zigbee and Thread packets simultaneously. However, because of MAC retries, in most cases this should not be a limitation.

- The combined duty cycle of the Zigbee and Thread traffic to or from the device must not exceed what would normally be tolerated by a single Zigbee or Thread device.
