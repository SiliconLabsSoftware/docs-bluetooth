# Implementing Multiprotocol with an 802.15.4-Based Stack

This section offers general information about implementing an 802.15.4-based stack, such as Zigbee or Connect, as part of a multiprotocol applications. For specifics on how to configure plugins and other details specific to a particular protocol, see one of the following application notes:

- [Dynamic Multiprotocol Development with Bluetooth and Zigbee EmberZNet SDK 6.x and Lower](https://docs.silabs.com/multiprotocol/latest/dynamic-multiprotocol-bluetooth-zigbee-sdk-7x-higher/)

- *AN1209: Dynamic Multiprotocol Development with Bluetooth and Connect*

## Wireless Protocol Support

Different wireless protocols have different characteristics that have been leveraged with the design of Dynamic Multiprotocol. For example, Bluetooth Low Energy is very strict and predictable in its schedule of radio operations; advertisement and connection intervals occur at set times. In contrast, an 802.15.4 protocol is more flexible in the timing of many message events; CSMA (carrier sense multiple access) in IEEE 802.15.4 adds random backoffs so that event delays are on the order of milliseconds. This allows IEEE 802.15.4 messages to be sent around the Bluetooth Low Energy events and still be reliably received.

## 802.15.4 RAIL Priority

802.15.4 protocols currently have three RAIL priorities.

| **No.** | **Name** | **Default Setting** | **Exit Criterion** |
|---------|----------|---------------------|--------------------|
| 1 | Active TX | 100 | MAC ACK received (or not) |
| 2 | Active RX | 255 | Packet filtered or MAC ACK sent |
| 3 | Background RX | 255 | Task with higher Priority present |

If an Active TX gets executed the radio will be released at the time the corresponding MAC acknowledgement was received (or a time-out occurred).

Background RX will leave the radio in receive state ready to receive asynchronous messages. If the active RX priority is different than the background RX priority, the receive priority will be raised whenever a sync word is detected and only lowered once that packet is filtered or completed and its ACK is sent if one was requested.

### Balancing Priorities

As explained in [Implementing Multiprotocol with Bluetooth](./06-implementing-multiprotocol-with-bluetooth), by default the Bluetooth priority range is mapped into the RAIL priority range 16 - 32. In general, Bluetooth starts out using low priority (32) and dynamically increases the priority up to the maximum (16) as needed if messages are not succeeding.

As described in [Radio Scheduler Examples](./03-radio-scheduler-examples), an 802.15.4-based stack such as Zigbee or Connect uses default RAIL priority values of 255 for background RX, 255 for active RX, and 100 for active TX.

As a result of these default RAIL priorities, in an 802.15.4 protocol-Bluetooth multiprotocol application, by default Bluetooth traffic will always take priority over 802.15.4 protocol traffic. This is a good choice for many applications, because Bluetooth traffic has stringent timing requirements, unlike 802.15.4 protocols. However, if Bluetooth traffic load is very high (for example, sending lots of data using a very small connection interval), it is possible for 802.15.4 protocol traffic to be completely blocked from access to the radio because of its lower priority and the very small windows of available radio time left by the Bluetooth traffic.

>**Note**: The following information is currently only applicable to the EmberZNet Zigbee stack. Silicon Labs Connect does not yet have the API needed to change the priorities.

If you are developing an 802.15.4-based dynamic multiprotocol application, and it is important for that traffic to succeed in the presence of very high load Bluetooth traffic, you can adjust the default priorities as shown in the table below using the following API:

| **No.** | **Name** | **Default Setting** |
|---------|----------|---------------------|
| 1 | Active TX | 23 |
| 2 | Active RX | 24 |
| 3 | Background RX | 255 |

Because the Bluetooth initially sets its RAIL priority to 32, these 802.15.4 priority settings give 802.15.4 traffic higher priority than Bluetooth initially, which gives the 802.15.4 protocol a chance to transmit or receive traffic successfully even in the presence of a very high load of Bluetooth traffic. On the other hand, Bluetooth will dynamically increase its priority if it is bumped from the scheduler by the 802.15.4 traffic, up to a high priority of 16. Thus, after allowing the 802.15.4 protocol access to the radio initially, Bluetooth will take priority on subsequent retries if necessary.

This approach allows both protocols to compromise on their use of the radio without one being able to completely dominate over the other.
