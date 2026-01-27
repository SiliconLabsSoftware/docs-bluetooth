# Types of Multiprotocol Implementations

Multiple protocols can be implemented on a single device in different ways:

- **Switched**: The application switches between protocols using a bootloader.

- **Dynamic**: The application time-slices between two protocols.

- **Concurrent**: The application is able to receive multiple protocols full time by:

  - Running on the same RF channel of a single radio,

  - Using Silicon Labs Concurrent Listening technology on a single radio, or

  - Using a multi-chip or multi-radio solution.

The following discusses these implementations in more detail.

## Switched Multiprotocol

In switched multiprotocol, a device is initially programmed with one protocol, and then uses a bootloader to switch to another protocol at some point in the future. Switched multiprotocol has two primary use cases.

**Future Proofing**: Device manufacturers may need to sell their devices into different protocol environments, or they may need to plan for an environment changing over time.

**Commissioning by Smartphone**: Device manufacturers who are committed to a single protocol environment, such as Zigbee Home Automation, may want to make the process of adding a new device more secure and user friendly. In this case, the device is initially programmed with a Bluetooth commissioning application. The Home Gateway/Trust Center is also programmed with a Bluetooth application, and the customer uses a commissioning app on their smartphone. Using the smartphone app they can join a new device onto the network, set up a pairing with other appropriate devices in the network, then switch over to the network protocol. Each wireless network protocol has its own mechanisms for joining and pairing devices, but all can be accommodated with this mechanism.

An effective switched multiprotocol implementation requires:

- A multiprotocol-enabled platform with sufficient memory.

- A consistent API to use the radio.

- A cross-compatible bootloader.

## Dynamic Multiprotocol

In a dynamic multiprotocol implementation, two protocols run concurrently, but the application time-slices the radio and rapidly changes radio configurations, such as channel, to enable different wireless protocols to operate reliably at the same time. In time-slicing, the software schedules tasks based around their priority and minimum duration and will default to a background listen task between scheduled tasks. The scheduler can interrupt or delay a lower priority task if a higher priority task is scheduled.

A dynamic multiprotocol implementation allows a device to perform multiple simultaneous functions. For example, an end user can use a smartphone app to connect to the device via Bluetooth to control it or to perform diagnostics, while at the same time the device is connected to the Zigbee network routing packets and acting on Zigbee Cluster Library commands.

In addition to a multiprotocol-enabled platform with sufficient memory, common code infrastructure, common radio interface, and common API to use the radio such as that provided by Silicon Labs RAIL, an effective dynamic multiprotocol implementation requires:

- An RTOS to support task switching and resource sharing.

- A radio scheduler to manage time-slicing. See [Radio Scheduler](./04-radio-scheduler) for background on the Silicon Labs radio scheduler.

See [Dynamic Multiprotocol User’s Guide](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ug/) for additional information about Silicon Labs dynamic multiprotocol implementation and [RAIL Fundamentals](https://docs.silabs.com/rail/latest/rail-fundamentals/) for more information about Silicon Labs RAIL.

## Single Channel Concurrent Multiprotocol

In a single radio, single channel concurrent multiprotocol implementation, two protocols are able to run simultaneously on a single radio by sharing the same radio channel.

For example, a Zigbee/Thread gateway or controller device could manage both a Zigbee-based and a Thread-based network at the same time. In the case of sharing a single channel, the networks must coordinate what channel they use. In addition, they must be able to accommodate reduced bandwidth either by scaling their traffic or by limiting the number of devices on the network.

## Concurrent Listening

Concurrent Listening, a Silicon Labs technology, allows the Zigbee and Thread stacks to operate on independent 802.15.4 channels when used with the multiprotocol radio co-processor (RCP). This feature works on EFR32xG21 and EFR32xG24 parts only. For detailed information on the multiprotocol RCP, see [Running Zigbee, OpenThread, and Bluetooth Concurrently on a Linux Host with a Multiprotocol Co-Processor](https://docs.silabs.com/multiprotocol/latest/multiprotocol-solution-linux/).

Concurrent Listening allows a single radio to listen on two channels simultaneously. This is accomplished by extremely rapid switching (on the order of tens of microseconds) that allows the radio to detect preambles for incoming packets on either channel. Once a preamble is detected, the radio stays on that channel until the packet is received, then resumes rapid switching.

Note that this fast switching differs from the time slicing used in dynamic multiprotocol, because the dwell on each channel is just sufficient to detect a preamble and shorter than the full preamble time, so incoming packets are not missed on either channel. In contrast, for dynamic multiprotocol the time slot is large enough to receive scheduled packets, which means that the other channel is not being listened to during that time.

When enabling the concurrent listening feature the PHY performance is slightly degraded. The sensitivity of the IEEE 802.15.4 PHY in this mode drops to around -98 dBm for both the EFR32xG21 and EFR32xG24.
