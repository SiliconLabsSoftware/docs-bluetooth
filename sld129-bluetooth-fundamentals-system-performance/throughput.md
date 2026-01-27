
# Throughput with Bluetooth Low Energy Technology

## Introduction

This page describes the maximum achievable bitrates on Blue Gecko devices (EFR32BGxx) with different Bluetooth SDK versions. However, the principles can also be applied to legacy devices (BLExxx), taking into account their parameters.

The application-level data, throughput with *Bluetooth Low Energy* is affected by many factors. In particular, two operations below have a different throughput calculation:

**1.** **Acknowledged data transfer**. In this use case, the reception of all data packets is
acknowledged by the receiver. The receiver sends a response for every read/write
request in the next connection interval. The connection is reliable but
the throughput is low.

![Acknowledged data transfer](resources/throughput-fig1.png?darkModeUrl=resources/throughput-fig1.png)

**2.** **Unacknowledged data transfer**. In this use case, packets can be sent sequentially without waiting for acknowledgment from the other
side. This ensures much higher throughput, but a less reliable connection.

![Unacknowledged data transfer](resources/throughput-fig2.png?darkModeUrl=resources/throughput-fig2.png)

Acknowledged and unacknowledged operation can be mixed on the same connection by sending packets of different types.
The following packet types result in acknowledged operation:

- Read (read request)
- Write (write request)
- Indication
- Prepare write
- Signed write

The following packet types result in unacknowledged operation:

- Write without response (write command)
- Notification

## Throughput Calculation for Acknowledged Data Transfer

For acknowledged data transfer, the throughput depends on the following parameters:

- Connection interval
- MTU size
- Attribute protocol operation

### Connection Interval

The connection interval specifies the frequency of sending data, which varies between 7.5 ms up to 4000 ms. After the sender sends data (or request), the sender has to wait for the receiver to send an acknowledgment. Therefore, one (GATT) operation takes two connection intervals.

The lower the connection interval, the higher the potential data rate. For example, using the smallest connection interval of 7.5 ms e.g., with 125-byte useful payload / connection interval, the radio can (assuming no lost packets and no re-transmissions) achieve a theoretical data rate, as follows:

**1000 ms / (2 * 7.5 ms) * 125 bytes = 8,333 bytes/sec = 66,666 bps**

On the other hand, when using the largest connection interval of 4000 ms e.g., with 22-byte useful payload, data rate will be reduced to:

**1000 ms / (2 * 4000) ms * 22 bytes = 2.75 bytes/sec = 22 bps**

### MTU Size

Maximum Transfer Unit (MTU) specifies the number of bytes that can be sent within one GATT operation. In other words, it's the number of bytes that can be sent within two connection intervals. MTU size can be set for each connection. However, it has an upper limit, which varies with Bluetooth stack versions. The maximum MTU size for each Bluetooth stack version is summarized in the following table

|Bluetooth Stack Version |Maximum MTU Size|
|:---|:---:|
|For legacy devices (BLExxx)||
|<= 1.5.0|23 |
|For Blue Gecko devices||
|1.0.x|23|
|2.0.x|58|
|2.1.x|126|
|2.3.x or later|250|

Note that the MTU size depends on both sides. For example, if the remote device supports a smaller MTU size, the smaller MTU size will be used. The higher the MTU size, the higher the throughput. Twice the MTU size doubles the throughput.

### Attribute Protocol (ATT) Operation

MTU size includes the GATT header, which has a variable length and means that the useful payload is a bit smaller than the MTU. The size of the GATT header depends on the operation type, hence the maximum useful payload is different for different operations. This is summarized in the following table:

|ATT Operation|Max Useful Data / ATT Operation|
|:---|:---:|
|Read |MTU - 1 bytes|
|Write |MTU - 3 bytes|
|Indication |MTU - 3 bytes|
|Prepare write |MTU - 3 bytes|
|Signed write |MTU - 15 bytes|

### Maximum Achievable Throughput with ACK

For acknowledged operations, the maximum throughput can be achieved with the following parameters:

- Connection interval: **7.5 ms**
- MTU size: **250 bytes**
- Attribute protocol operation used: **Read**

This results in a maximum throughput of

**1000 ms / (2 * 7.5 ms) * (250 - 1) bytes = 16,600 bytes/sec = 132,800 bps**

## Throughput Calculation for Unacknowledged Data Transfer

For unacknowledged data transfer, the throughput depends on the following parameters:

- Packet size
- Attribute protocol operation
- PHY (physical layer) bitrate
- Connection interval

### Packet Size

Data over Bluetooth is sent in packets. Multiple packets can be sent within one connection interval sequentially (with 150 µs inter-frame spacing). Packets have variable length, but their length has an upper limit. The maximum PDU (protocol data unit) size of a packet depends on the Bluetooth stack version, as summarized in the following table:

|Bluetooth Stack Version|Bluetooth Standard|Maximum PDU Size|
|:---|:---:|:---:|
|For legacy devices (BLExxx)|||
|<=1.5.0|4.0|27 byte|
|For Blue Gecko devices|||
|1.0.x| 4.0| 27 byte|
|2.0.x|4.2|38 byte|
|2.1.x| 4.2| 38 byte|
|2.3.x| 5.0| 128 byte|
|2.4.x| 5.0| 160 byte|
|2.6.x or later| 5.0| 251 byte|

Twice the packet size does not mean double throughput because the larger packets take more time to send. As a result, less packets can be sent during the same time interval. However, the larger the packet size, the smaller the overhead, which increases the application level data throughput.

For example, for a 27-byte PDU it takes the following time to send out a packet:

328 µs + 150 µs + 80 µs + 150 µs = 708 µs

328 µs is the effective transmitting time, 150 µs is IFS (interframe space), 80 µs is receiving, 150 µs is IFS. This results in a theoretical data rate calculation, as follows:

**27 byte / 708 µs = 38,135 bytes/sec = 305,080 bps**

For a 251-byte PDU, it takes the following time to send out a packet:

2120 µs + 150 µs + 80 µs + 150 µs = 2500 µs

This results in a theoretical data rate calculation, as follows:

**251 byte / 2500 µs = 100,400 bytes/sec = 803,200 bps**

Note that the packet size depends on both sides. If the remote device supports a smaller packet size, the smaller packet size will be used.

For more details about packet timing, see the blog [Exploring Bluetooth 5 - How Fast Can It Be](https://blog.bluetooth.com/exploring-bluetooth-5-how-fast-can-it-be).

### Attribute Protocol Operation

The PDU size includes the L2CAP header, which is 4 bytes long and the GATT header which has a variable length. This means that the useful payload of a packet is a bit smaller than the PDU size. The size of the GATT header depends on the operation type, hence the maximum useful payload is different for different operations. This is summarized in the following table:

|ATT Operation|Max Useful Data Payload of a Packet|
|:---|:---|
|Write without response |PDU - 4 - 3 bytes|
|Notification|PDU - 4 - 3 bytes|

The theoretical effective data rate when sending out notifications with 251-byte PDU is as follows:

**(251 – 4 – 3) byte / 2500 µs = 97,600 bytes/sec = 780,800 bps**

If the MTU is larger than the PDU, multiple packets can be sent within one GATT operation. In this use case, only the first packet contains the L2CAP header and the GATT header, and the following packets contain only data within the PDU.

### Physical Layer (PHY) Bitrate

Bluetooth 4.2 specifies the PHY bitrate to be exactly 1 Mbps.

Bluetooth 5 (introduced in Bluetooth SDK v2.3) specifies PHY bitrates as follows:

- 2 Mbps
- 1 Mbps.
- 500 kbps (1 Mbps with 1:2 convolution coding)
- 125 kbps (1 Mbps with 1:8 convolution coding)

These bitrates only define the symbol time and not the effective throughput. However, the higher the PHY bitrate, the higher the throughput. Double bitrate means nearly double throughput. It is not exactly double because of the constant IFS (IFS = 150 µs for both 1 Mbps and 2 Mbps).

The following table summarizes the time it takes to send out a packet with different PDU sizes and PHY bitrates. The times include transmitting + IFS + receiving + IFS.

||1 Mbps|2 Mbps|
|:---|:---|:---|
|27 byte |708 µs|---|
|38 byte|796 µs|548 µs|
|128 byte |1516 µs |908 µs|
|160 byte |1772 µs |1036 µs|
|251 byte |2500 µs |1400 µs|

For example, if you use 2 Mbps PHY with 251-byte PDU, the theoretical throughput is:

**(251 – 4 – 3) byte / (1060 + 150 + 40 + 150) µs = 174,285 bytes/sec = 1,394,280 bps**

For more details see the blog [Exploring Bluetooth 5 - How Fast Can It Be](https://blog.bluetooth.com/exploring-bluetooth-5-how-fast-can-it-be).

### Connection Interval

If acknowledgment is not required, any number of packets can be sent within one connection interval. As a result, the connection should not directly influence the throughput. Data packets, however, must be aligned so that the first packet starts at the start of the connection interval. For example, for a  7.5 ms connection interval, 251-byte PDU, 2 Mbps PHY, you can send 4 packets in one connection interval. The 5th packet does not fit and has to wait for the next connection interval to start.

![connection interval](resources/throughput-fig3.png?darkModeUrl=resources/throughput-fig3.png)

This means that the effective throughput with these parameters is as follows:

**4 * (251 – 4 – 3) byte / 7500 µs = 130,133 bytes/sec = 1,041,066 bps**

To avoid the overhead introduced by skipped packages, adjust the connection interval so that the remaining time at the end of the connection interval is minimal. Longer connection intervals usually mean smaller overhead because of less frequent re-adjustment.

![Connection Interval Adjustment](resources/throughput-fig4.png?darkModeUrl=resources/throughput-fig4.png)

However, if the connection interval is too long the following applies:

1. Too many packets may be queued while waiting for next connection interval and you can run out of memory.
2. You may have to wait much more to resynchronize if the connection breaks because of consecutive CRC errors. Long connection interval is hence not recommended in noisy environments where CRC errors are expected.

### Maximum Achievable Throughput without ACK

For unacknowledged operation, the maximum throughput can be achieved with the following parameters:

- Connection interval: **12.5 ms**
- PDU size: **251 bytes**
- Attribute protocol operation used: **Write without response / Notification**
- PHY bitrate: **2 Mbps**

In this use case, 8 packets fit into one connection interval (8*1400 us = 11200 µs) and the theoretical throughput is:

**8 * (251 – 4 – 3) byte / 12,500 µs = 156,160 bytes/sec = 1,249,280 bps**

## Considerations for Smart Phones

The connection parameters and the MTU size depend on both devices participating in the connection. When connecting to a smart phone, the devices start to negotiate the connection interval and the MTU size. Depending on the smart phone and OS version, the minimum connection interval can be much greater than 7.5 ms and the maximum MTU size can be less than the MTU size supported by the Blue Gecko. This has a critical effect on throughput. The negotiated parameters are signaled by the stack events *sl_bt_evt_connection_parameters* and *sl_bt_evt_gatt_mtu_exchanged*, so you can check what parameters are supported by your phone.

Reference: [Exploring Bluetooth 5 - How Fast Can It Be](https://blog.bluetooth.com/exploring-bluetooth-5-how-fast-can-it-be)
