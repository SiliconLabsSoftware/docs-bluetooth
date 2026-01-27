# Use Case #1: I want to signal the presence of a device

Although Bluetooth is primarily used for creating a point-to-point wireless connection between two devices, and Bluetooth advertisements are mainly used to make this possible, there are use cases where an advertisement is used for simply advertising the presence of the device. This may be useful if you want to:

1. Find the device or

2. Mark a spot in space or signal that you entered a specific zone.

In both cases, the only thing you want to broadcast in the advertisements is an ID that identifies the device or the spot/zone.

**Bluetooth feature to be used**: Legacy non-connectable advertisements (beacons).

Legacy Bluetooth advertisements can transmit up to 31 bytes of data with a minimum interval of 20 ms. The two main types of advertisements are connectable and non-connectable advertisement. Connectable advertisements typically advertise the device name and optionally a service UUID that describes the main functionality of the device. Non-connectable advertisements (Bluetooth beacons) typically advertise an ID or URL and sometimes their TX power (or the RSSI level at 1 m distance) so that a receiver can roughly calculate their distance based on the measured RSSI.

The format of the payload is specified by the Bluetooth [advertising data format](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-advertising-scanning/advertising-data-basics).

The advertising data types are specified by the Bluetooth SIG here: [https://btprodspecificationrefs.blob.core.windows.net/assigned-numbers/Assigned%20Number%20Types/Generic%20Access%20Profile.pdf](https://btprodspecificationrefs.blob.core.windows.net/assigned-numbers/Assigned%20Number%20Types/Generic%20Access%20Profile.pdf).

Although the content of the advertisement can be anything as long as it complies with the advertising data format, two formats are widely used.

- iBeacon is a standard format defined by Apple: [https://developer.apple.com/ibeacon/](https://developer.apple.com/ibeacon/).

- Eddystone beacon is a standard format defined by Google: [https://github.com/google/eddystone](https://github.com/google/eddystone).

**Bluetooth API to be used**:

- `sl_bt_advertiser_create_set()`

- `sl_bt_advertiser_set_timing()`

- `sl_bt_advertiser_set_channel_map()`

- `sl_bt_legacy_advertiser_set_data()`

- `sl_bt_legacy_advertiser_start()`

- `sl_bt_system_set_tx_power()`

**Tips for low power consumption**:

- Set the advertising interval to as long as your application allows. For example, if you want to signal that you entered a zone, a one-second interval is more than enough. Obviously, shorter intervals result in lower latency, but the shorter the interval the higher the power consumption. Advertising interval can be set with `sl_bt_advertiser_set_timing()`.

- Make sure that you put the device into deep sleep mode (EM2) between two advertisements by installing the *Power Manager* software component and not installing any component that requires EM1 (such as UART or logging). If you start with the **Bluetooth – SoC iBeacon** or **Bluetooth – SoC Empty** example applications, this is pre-configured.

- If you plan to wait many seconds between beacons, consider putting the device into EM4 mode, which saves even more energy. See the following software example:

   [https://github.com/SiliconLabs/bluetooth_applications/tree/master/bluetooth_using_em4_energy_mode_in_bl_ibeacon_app](https://github.com/SiliconLabs/bluetooth_applications/tree/master/bluetooth_using_em4_energy_mode_in_bl_ibeacon_app).

- Make sure that you use non-connectable mode (use *sl\_bt\_advertiser\_non\_connectable* as the *connect* parameter when you call `sl_bt_legacy_advertiser_start`). Although connectable advertisements can also be used for broadcasting an ID, they consume a bit more energy since they are listening for connection requests after each advertisement.

- By default Bluetooth advertisements are sent on 3 channels, so that they can still be received if there is interference on one or two of the channels. If you know which Bluetooth channels are free of interference and which are blocked (for example because you know the Wi-Fi channels used in your area), you can set the channel map so that advertisements are only sent on free channels. This saves a lot of energy. Use `sl_bt_advertiser_set_channel_map()`.

- Set the TX power so that the beacon can be seen in the area you want it to be seen. Do not set it too high if not necessary. TX power can be set with `sl_bt_system_set_tx_power()`.
