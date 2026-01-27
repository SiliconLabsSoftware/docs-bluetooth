# Using 2M and LE Coded PHY

## Introduction

Bluetooth 5.0 introduces three new PHYs in addition to the default 1M symbol rate:

- 2M PHY ensures higher data throughput and
- A new long range PHY (LE Coded PHY) with 125 kbps and 500 kbps coding which gives range gains of 1.5-2x with improved sensitivity of 4 to 6dB.

The LE Coded PHY uses 1 Mbit PHY but payload is coded at 125 kbps or 500 kbps. It also adds Forward Error Correction and Pattern Mapper.

**Note:** 2M and LE Coded PHY are only supported on specific devices.

## Changing PHY

Changing to 2M or LE Coded PHY can be requested from either slave or master device using the command sl_bt_connection_set_preferred_phy(uint8_t connection, uint8_t preferred_phy, uint8_t accepted_phy), where the `preferred_phy` value maps as follows:

- 0x01: 1M PHY
- 0x02: 2M PHY
- 0x04: 125k Coded PHY (S=8)
- 0x08: 500k Coded PHY (S=2)

The `accepted_phy` signifies the accepted PHYs in remotely-initiated PHY update requests and its value maps as follows:

- 0x01: 1M PHY
- 0x02: 2M PHY
- 0x04: Coded PHY
- 0xff: Any PHYs (default)

Multiple PHYs can be selected both as preferred PHY and as accepted PHY. The connected devices will negotiate which PHY to use based on this information.

For compatibility reasons, the default PHY is always 1Mbit on a connection - unless it was opened from a Coded PHY advertisement. After the connection is formed, either side (master or slave) can request a new PHY to be used. After the PHY changes, it is reported in a *sl_bt_evt_connection_phy_status* event.

To learn more about how to open a (long range) connection from a Coded PHY advertisement, see the following code example: [Advertising and Scanning with LE Coded PHY](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/advertising/advertising_and_scanning_with_le_coded_phy).

**Note**: When a new PHY is requested, the device will take a **minimum** of six connection intervals to switch to the new PHY.

The event `sl_bt_evt_connection_phy_status` will be raised on both devices in the following situations (if the new PHY request is accepted by the receiving device):

- Switching **to/from** LE Coded PHY **from/to** another PHY (1 Mbit/2 Mbit)
- Switching from 1 Mbit to 2 Mbit PHY and vice-versa

The event `sl_bt_evt_connection_phy_status` will **only be raised on the requesting device** in the following use cases:

- The new PHY request is not accepted (e.g., the receiving device doesn’t support it)
- Switching LE Coded PHY **coding** (from S=2 to S=8 and vice-versa) when LE Coded PHY is already in use
  - Changing the coding is not negotiated over-the-air, it’s done **only locally on the requesting device** and the receiving device continues using the same coding. At the RX stage, the radio detects the coding based on the packet preamble. It can use a different coding for the TX stage.

Additionally, **default coding for LE Coded PHY is S=8** (125 kbit), which means that when an EFR32BG13 accepts a PHY change request to LE Coded PHY **it will default to S=8**. The application should change the coding if desired and the new coding will then be used if the PHY is again changed to 1Mbit/2Mbit and then back to LE Coded PHY.

## Demonstration

These are examples that show how to change between the PHYs using 2 EFR32xG13 devices in NCP mode and controlled with Bluetooth NCP Commander.

### Changing to LE Coded PHY (S=8, 125kbit)

![Changing to LE Coded PHY](resources/v3-changed-phy-2.png?darkModeUrl=resources/v3-changed-phy-2.png)

### Changing the Coding While in LE Coded PHY

As explained earlier, changing the coding is not negotiated over-the-air between the devices and for that reason the event `sl_bt_evt_connection_phy_status` is only raised on the requesting side and very shortly after the command is sent. The receiving device will continue using the same coding.

![Changing the coding while in LE Coded PHY](resources/v3-changed-on-one.png?darkModeUrl=resources/v3-changed-on-one.png)

### Attempted Change to LE Coded PHY

In the example below, a new PHY request is attempted but the other device is an EFR32BG1, which only supports 1Mbit PHY. As a result, the event `sl_bt_evt_connection_phy_status` is immediately raised on the requesting device to notify that the PHY has not been changed and no event is raised on the receiving device.

![Attempted change to LE Coded PHY](resources/v3-phy-not-changed-2.png?darkModeUrl=resources/v3-phy-not-changed-2.png)

## Impact on Energy Consumption

Using 2M PHY leads to smaller current consumption because of the shorter RX/TX periods. On the other hand, using LE Coded PHY leads to higher current consumption because of the longer RX/TX periods. The subsequent images show the current profile of a notification packet with 20 bytes of data payload on all PHYs. The red blocks are highlighting the duration of the entire wake-up period and the average current consumption for said period.

These measurements were made from the peripheral device (RX precedes TX) with TX Power of 0dBm. The images are sorted from shortest to longest RX/TX periods: 2 Mbit, 1 Mbit, LE Coded S=2 (500 kpbs) and LE Coded S=8 (125 kpbs).

**Note:** When using LE Coded PHY, the packet header is always coded with S=8 but the payload can be S=2 or S=8 coded and that is indicated by a bit in the header. This means that empty packets will not have a significant difference between S=8 and S=2 coding in terms of TX duration.

### 2 Mbit PHY

![2 Mbit PHY](resources/sld269-fig4.png?darkModeUrl=resources/sld269-fig4.png)

### 1 Mbit PHY

![1 Mbit PHY](resources/sld269-fig5.png?darkModeUrl=resources/sld269-fig5.png)

### LE Coded PHY (S=2, 500kbps)

![LE Coded PHY S=2](resources/sld269-fig6.png?darkModeUrl=resources/sld269-fig6.png)

### LE Coded PHY (S=8, 125kbps)

![LE Coded PHY S=8](resources/sld269-fig7.png?darkModeUrl=resources/sld269-fig7.png)
