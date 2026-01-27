# System Architecture

## Overview

The system architecture includes various software and hardware components:

- Radio Board: A co-processor image that runs on the EFR32 co-processor. This can be either a RCP (Radio Co-Processor) or an NCP (Networking Co-Processor).
- Co-Processor Communication Daemon (CPCd): A Linux host process that communicates with the co-processor over a Universal Asynchronous Receiver/Transmitter (UART) or Serial Physical Interface (SPI) physical link, and multiplexes protocol streams.
- Zigbeed: A Linux daemon that runs the Zigbee Networking stack and sends and receives Spinel messages to CPCd over a socket when running in RCP mode.
- Zigbee host application (Z3Gateway): An application that communicates with Zigbeed using EmberZNet Serial Protocol / Asynchronous Serial Host (EZSP / ASH) over a virtual serial port, or directly to CPCd in the case of the Zigbee NCP/OpenThread RCP.
- OpenThread host application, like OpenThread Border Router (otbr-agent, OTBR): An application that includes the OpenThread protocol stack, and which connects to CPCd over a socket and operates on a separate PAN from the Zigbee network.
- BlueZ: Official Linux Bluetooth protocol stack that runs on the Linux host and communicates with the Bluetooth LE Controller on the RCP via the Host Controller Interface (HCI). `bluetoothd` is the BlueZ daemon for GAP/GATT applications that must be running to manage the Bluetooth devices.
- Bluetooth host applications: In the Zigbee NCP + BLE NCP case, the `bt_host_empty` application communicates with the NCP via CPCd using BGAPI over CPC. In a multiprotocol RCP case, the BlueZ stack uses a BlueZ CLI tool (`bluetoothctl`) controls the RCP by communicating with the CPCd through the CPC-HCI bridge over a virtual serial port.
- CPC-HCI bridge application: The bt_host_cpc_hci_bridge is a host applicatiion that serves as a communication bridge between the BlueZ host stack and CPCd, and creates a virtual serial interface.

The following figure illustrates the system architecture for the RCP:

![System Architecture for the Multiprotocol RCP](./resources/figure-1-1-host-rcp-configuration.png)

The following figure illustrates the system architecture for the Zigbee NCP/OpenThread RCP:

![System Architecture for the Zigbee NCP + OpenThread RCP](./resources/figure-1-2-host-ncp-rcp-configuration.png)

The following figure illustrates the system architecture for the Dynamic Multiprotocol (DMP) Zigbee + Bluetooth NCP.

![System Architecture for the Zigbee NCP + BLE NCP](./resources/figure-1-3-host-ncp-configuration.png)

## EFR32 Software

The EFR32 co-processor image comes in four flavors:

1. Multi-PAN 802-15.4 RCP (Radio Co-Processor):

   - Protocols Run: Openthread + Zigbee
   - Project Name: rcp-uart-802154.slcp and rcp-spi-802154.slcp
   - Description: The multipan RCP is based on the OpenThread 802.15.4 RCP with added multi-PAN and CPC support. It has a small flash footprint (~150K) and uses the Spinel protocol to serialize commands. The Spinel messages are further encapsulated by the CPC protocol before being sent over the physical link. Both UART and SPI links are supported. This is illustrated in system-architecture for the Multiprotocol RCP (ignoring the Bluetooth components).

2. Multiprotocol RCP

   - Protocols Run: Openthread + Zigbee + Bluetooth
   - Project Name: rcp-uart-802154-blehci.slcp and rcp-spi-802154-blehci.slcp
   - Description: The multiprotocol RCP adds the Bluetooth Controller and FreeRTOS to the above 802.15.4 RCP. It has a larger flash footprint (~250k). HCI is used to serialize Bluetooth commands over CPC. Both UART and SPI links are supported. See system-architecture for the Multiprotocol RCP.

3. Zigbee NCP (Network Co-Processor) with OpenThread RCP

   - Protocols Run: Zigbee + Openthread
   - Project Name: zigbee_ncp-ot_rcp-uart.slcp and zigbee_ncp-ot_rcp-spi.slcp
   - Description: This configuration runs the Zigbee Networking stack on the EFR32 alongside the OpenThread RCP. The Zigbee application still runs on the host and uses EZSP to send commands to the Zigbee NCP over CPC. Note that this solution does not make use of Zigbeed, because the Zigbee networking stack is running on the EFR32, not on the host. OpenThread runs on the host as in the other cases and uses Spinel over CPC to communicate with the OpenThread RCP. Both UART and SPI links are supported. See system-architecture for the Zigbee NCP + OpenThread RCP. **Due to a larger application footprint, it is recommended to choose parts with sufficient RAM (>64kB).**

4. Zigbee NCP with BLE NCP

   - Protocols Run: Zigbee + BLE
   - Project Name: zigbee_ncp-ble_ncp-uart.slcp zigbee_ncp-ble_ncp-spi.slcp
   - Description: The Zigbee application (Z3GatewayCPC) runs on the Linux host and communicates with the NCP using EZSP over CPC (SPI and UART are both available options). The Bluetooth host app bt_host_empty is compiled with CPC=1 option to enable communication with the DMP NCP over CPC. See system-architecture for the Zigbee NCP + BLE NCP. **Due to a larger application footprint, it is recommended to choose parts with sufficient RAM (>64kB).**

## Host Software Overview

The Co-Processor Communication Daemon (CPCd) enables users to have multiple stack protocols interact with the radio co-processor over a shared physical link. CPCd is distributed as three components: the daemon binary cpcd; a library libcpc.so that enables C applications to interact with the daemon; and a configuration file cpcd.conf. In CPC, data transfers between processors are segmented in sequential packets. Transfers are guaranteed to be error-free and sent in order. Multiple applications can send or receive on the same endpoint without worrying about collisions. A library libcpc.so is provided to simplify the interaction between the user application and the daemon via Unix domain sockets. Code to interface the OpenThread Spinel driver to libcpc.so is provided as part of this solution. For more information on CPCd, see <https://github.com/SiliconLabs/cpc-daemon/blob/main/readme.md>.

The Zigbee Daemon (Zigbeed) runs the Zigbee networking stack on the host. It is built with a Spinel adaptation layer that translates between 802.15.4 MAC primitives and Spinel messages and uses the Spinel driver-to-libcpc.so interface code referred to above. Spinel messages from Zigbeed are sent to CPCd where they are encapsulated and forwarded to the RCP. Zigbeed also opens a virtual serial port for communicating with the host application using the EZSP/ASH protocol.

The socat command line utility (also known as zigbee-socat) is used to create two virtual serial ports (PTY) and link them to each other. This allows a Zigbee host application that was built for a Zigbee NCP co-processor to interface with Zigbeed unchanged, simply by supplying the proper device name to it.

Both Zigbeed and the OpenThread stack can connect to CPCd and use the multi-PAN RCP at one time. Spinel messages for each application are labelled with a Spinel Interface ID (IID) which is supplied to the application at startup via the OpenThread Radio URL command line argument. The fact that the RCP is being shared between multiple PANs is transparent to the host applications. The only requirement is that all PANs must operate on the same 802.15.4 channel, the only exception being if Concurrent Listening. Coordination must happen between applications and no mechanism is provided as part of this solution.

Zigbeed stores non-volatile Zigbee stack tokens in a file called host_token.nvm. This allows Zigbeed to retain network information across resets.

For the Zigbee NCP with OpenThread RCP, the Zigbee host app and the OpenThread stack can also connect to CPCd and use the co-processor at the same time. They must coordinate at the application layer to ensure that both operate on the same 802.15.4 channel, or enable Concurrent Listening to operate on different 802.15.4 channels.

## Concurrent Listening

Concurrent Listening allows the Zigbee and Thread stacks to operate on independent 802.15.4 channels when used with the 802.15.4 RCP. This feature works on xG21, xG24 and xG26 parts only.

It functions by using the RX antenna diversity hardware block to switch extremely rapidly between two channels, approximately every 48 µsec. If a preamble is detected, it stays on the channel until completion of the packet. Because the switching is so rapid, packets on either channel are received. When enabling the concurrent listening feature the PHY performance is slightly degraded. The sensitivity of the IEEE 802.15.4 PHY in this mode drops to around -98 dBm for the xG21 and xG24, xG26.

SiSDK 2024.12.x includes rail_util_ieee802154_fast_channel_switching and rail_util_dma components which allows Zigbee and Thread networks to operate on 2 different radio channels. Note that the concurrent listening feature becomes active only when both networks are up. This feature can result in a decrease in receive sensitivity.

If this feature is not required, these components can be removed. However, if this component is removed it is imperative to make sure that both Zigbee and OpenThread networks are on the same channel.

Because concurrent listening makes use of the antenna diversity hardware block, antenna diversity is not available for the RCP on xG21, xG24, and xG26.
