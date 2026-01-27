# Unmanaged Coexistence

The unmanaged coexistence recommendations that follow provide guidance on how to maximize the EFR32MGxx/EFR32BGxx message success with strong nearby Wi-Fi.

## Implement Frequency Separation

You can minimize the blocking effect of high-power Wi-Fi on other 2.4 GHz radios by moving the Wi-Fi to one end of the pass band. This impacts fewer Bluetooth channels and allows 802.15.4 channels to move to the other end of the pass band where Wi-Fi will have minimal blocking effect.

### Bluetooth

Bluetooth co-channel operation with Wi-Fi has the most impact on Bluetooth communication. For Bluetooth devices (point-to-point) and Bluetooth mesh devices using Generic Attribute Profile (GATT) bearer communication, at least one ADV channel is minimally blocked and supports establishing a connection via Advertising, Scanning, and Initiating link-layer states. While establishing a connection, the Bluetooth connection master specifies the channel map, but the connection master can also update the channel map during connection. However, the Bluetooth connection slave must follow the channel map provided by master.

If EFR32 becomes the connection master, the Bluetooth channel map can be specified via:

`sl_status_t sl_bt_gap_set_data_channel_classification (uint8 channel_map_len, const uint8\* channel_map_data)`

This command can be used to specify a channel classification for data channels. This classification persists until overwritten with a subsequent command or until the system is reset.

`channel_map` is 5 bytes and contains 37 1-bit fields. The *n*th such field (in the range 0 to 36) contains the value for the link layer channel index *n*:

   0: Channel *n* is bad.

   1: Channel *n* is unknown.

The most significant bits are reserved and shall be set to 0 for future use. At least two channels shall be marked as unknown.

For Bluetooth mesh devices using Advertising bearer communication, at least one ADV channel is minimally blocked and supports establishing communication via Advertising and Scanning.

### 802.15.4

From the observations in [802.15.4](./02-wi-fi-impact-on-bluetooth-and-802-15-4-radios#802154) co-channel operation of 802.15.4 with 100% duty cycle Wi-Fi blocks most of the 802.15.4 messages and must be avoided. Also, EFR32MGxx tolerates up to 20 dB stronger Wi-Fi signal in “far-away” channel case than in adjacent channel case. The 802.15.4 network performance is improved by maximizing the frequency separation between the Wi-Fi network and the 802.15.4 network.

If the Wi-Fi and 802.15.4 radios are implemented with a common host (MCU controlling both radios), then the host should attempt to maximize the frequency separation. For Wi-Fi networks, the Access Point (AP) establishes the initial channel and, in auto channel configuration, is free to move the network to another channel using the Channel Switch Announcement (CSA), introduced in 802.11h, to schedule the channel change.

For OpenThread networks, frequency separation implementation depends on the application layer. For Zigbee networks, the Coordinator establishes the initial channel.

## Operate Wi-Fi with 20 MHz Bandwidth

Because Wi-Fi 802.11n uses OFDM (Orthogonal Frequency-Division Multiplexing) sub-carriers, third-order distortion products from these sub-carriers extend one bandwidth on each side of the Wi-Fi channel. 802.11n can operate in 20 MHz or 40 MHz modes. If operated in 40 MHz mode, 40 MHz of the 80 MHz ISM band is consumed by the Wi-Fi channel. However, an additional 40 MHz on each side can be affected by third-order distortion products. These third-order products can block the Bluetooth and 802.15.4 receiver and is the primary reason adjacent channel performance is up to 20 dB worse than “far-away” channel performance.

In proposing 40 MHz mode for 802.11n, the Wi-Fi standard anticipated potential issues with other 2.4 GHz ISM devices when Wi-Fi operated in 40 MHz mode. During association, any Wi-Fi station can set the **Forty MHz Intolerant** bit in the HT Capabilities Information. This bit informs the Wi-Fi access point that other 2.4 GHz ISM devices are present, forcing the entire Wi-Fi network to 20 MHz mode.

### Bluetooth

If the Wi-Fi and Bluetooth radios are implemented with a common host, then the host should have the Wi-Fi radio set the 40 MHz Intolerant bit during association to force the Wi-Fi to 20 MHz mode, increasing the number of channels available to Bluetooth and improving the Bluetooth performance.

If the application requires Wi-Fi to operate in 40 MHz mode, frequency separation can be maximized by placing Wi-Fi channel at upper or lower end of 2.4 MHz ISM band, minimizing the adjacent channels.

### 802.15.4

If the Wi-Fi and 802.15.4 radios are implemented with a common host, then the host should have the Wi-Fi radio set the Forty MHz Intolerant bit during association to force the Wi-Fi to 20 MHz mode, improving the 802.15.4 performance.

If the application requires Wi-Fi to operate in 40 MHz mode, frequency separation must be maximized by placing Wi-Fi channels and 802.15.4 channel at opposite ends of the 2.4 GHz ISM band.

From Silicon Labs’ managed coexistence testing, 802.15.4 performance with 40 MHz Wi-Fi, for the same Wi-Fi RF duty cycle, is comparable to 802.15.4 performance with 20 MHz Wi-Fi. While 802.15.4 performance with 100% Wi-Fi RF duty cycle is inherently impaired, 40 MHz Wi-Fi, for the same target Wi-Fi data rate, has a lower RF duty cycle than 20 MHz Wi-Fi, providing the 802.15.4 radio more frequent and longer time gaps for successful transmits and receives.

## Increase Antenna Isolation

From the observations in [Wi-Fi Impact on Bluetooth and 802.15.4 Radios](./02-wi-fi-impact-on-bluetooth-and-802-15-4-radios), minimizing the Wi-Fi energy seen by the EFR32 RF input improves the EFR32 receive range. For example, in the “far-away” channel case for Zigbee (Wi-Fi channel 1 and Zigbee channel 25) with 100% Wi-Fi duty cycle, a -96 dBm 802.15.4 message can be received when the average Wi-Fi energy at EFR32MGxx input is -30 dBm or less. If the Wi-Fi average transmit power level is +10 dBm, 40 dB or more antenna isolation between the Wi-Fi transmitter and 802.15.4 RF input is required to always receive a -96 dBm 802.15.4 signal, Wi-Fi ON or OFF.

Increased antenna isolation can be achieved by:

- Increasing the distance between antennas. In open-space, far-field, power received is proportional to 1/R2, where R is the distance between antennas.

- Taking advantage of antenna directionality. A monopole antenna provides a null along the axis of the antenna, which can be directed toward the Wi-Fi antenna(s).

## Implement Protocol and Stack Retry Mechanisms

You can maximize the use of built-in protocol MAC and stack retry mechanisms to minimize missed messages in the presence of high-power Wi-Fi.

### Bluetooth

Bluetooth (point-to-point) messages requires responses. If a response is not received within programmable time, the application can re-send the message up to a programmable limit.

Bluetooth mesh (mesh network) messages are sent via ADV payloads and responses are received during SCAN. Bluetooth mesh specifies:

- Optional relay nodes, which after a programmable time-out with no responses, can retransmit the original message for a programmable number of hops.

- Originator, after a programmable time-out with no response, can retransmit the original message for a programmable number of time-outs.

Both mechanisms improve Bluetooth mesh message success but should be used with caution. More relays nodes, shorter time-outs, and more retries may improve an individual message’s success, but these mechanisms can stress the mesh network by flooding too many identical messages. See [Bluetooth Mesh Network Performance](https://docs.silabs.com/btmesh/latest/btmesh-11-network-performance/) for details on these considerations.

### 802.15.4

The 802.15.4 specification requires retries at the MAC (Media Access Control) layer, which are implemented in Silicon Labs’ EmberZNet PRO stack. To further improve message delivery robustness, Silicon Labs EmberZNet PRO stacks also implements NWK (Zigbee net-work layer) retries, wrapping the MAC retries. The user application can also take advantage of APS (Zigbee Application Support (APS) Sub-Layer) retries, wrapping the NWK retries. More information on the retry mechanisms can be found at:

- [Zigbee Fundamentals](https://docs.silabs.com/zigbee/latest/zigbee-fundamentals/)

- [How does the EmberZNet stack retry work?](https://www.silabs.com/community/wireless/zigbee-and-thread/knowledge-base.entry.html/2012/06/29/how_does_the_emberzn-po1M)

These retry mechanisms are effective at improving message delivery. However, under high interference conditions, message latency increases.

For OpenThread networks, 802.15.4 retries at the MAC layer still apply. However, other message and retry mechanisms depend on the application layer.

## Remove FEM (or Operate FEM LNA in Bypass)

EFR32xGxx can deliver nearly +20 dBm transmit power and has excellent receiver sensitivity without an external FEM (Front End Module). However, an external FEM can increase transmit power to +20 dBm for increased range (in regions where this is permitted, for example, the Americas). The additional FEM LNA receive gain also improves sensitivity. However, this additional gain also degrades the EFR32xGx linearity performance in the presence of strong Wi-Fi.

For best receive sensitivity in the presence of strong Wi-Fi blockers, either eliminate the FEM or operate the FEM LNA in bypass mode. This recommendation is a trade-off as receive sensitivity without Wi-Fi blockers is improved with FEM LNA gain enabled.
