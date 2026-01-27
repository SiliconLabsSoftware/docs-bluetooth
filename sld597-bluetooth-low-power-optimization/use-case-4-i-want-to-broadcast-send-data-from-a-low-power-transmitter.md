# Use Case #4: I want to broadcast/send data from a low power transmitter

Bluetooth Low Energy is very often used to control devices around you, such as your lamp, LED strip, or garage door. Usually, this is achieved by creating a connection to the controlled device and sending different commands over this connection, discussed in [Use Case #5: I want to connect to a device occasionally to exchange some data](./use-case-5-i-want-to-connect-to-a-device-occasionally-to-exchange-some-data). However, in many cases the switch is running on a battery and requires very low power consumption. In this case, a good approach may be to use simple advertisements to send the commands instead of the complicated connection process. This not only gives the lowest possible power consumption, but also provides the lowest possible latency.

**Bluetooth features to be used**: legacy advertisements, extended advertisements.

If you have a controlled device (for example a lamp) that has a continuous power supply and a switch that runs on a battery, the simplest and probably best solution is to put the controlled device into continuous scanning mode, and send a single advertisement packet from the switch whenever an action is needed. Since the controlled device is scanning continuously, it will receive the command immediately. Also, the switch does not have to spend energy on creating a connection, since it sends a single packet only – maybe repeating it a few times for redundancy.

Obviously, this solution has drawbacks compared to Bluetooth connections:

- The devices cannot discover each other and pair.

- While the BT address of the switch is present in the advertisement, the address of the controlled device is not specified.

- Anyone can listen to the activities and anyone can control the controlled device.

However, these problems can be easily overcome.

- While the controlled device is scanning and the switch is waiting for a trigger during normal duty, both devices can be put into connectable advertising mode, and a connection created between them (or between them and a smartphone). This connection must be created only once. During this connection the switch learns the BT address of the controlled device, and they can also share a common secret which can be used later to secure the communication between them.

- Once the BT address of the controlled device is known, it can simply be added to the advertisement data.

- Once a common secret is shared, the commands provided in the advertisements can be encrypted and signed. Adding a counter to the command also protects against replay attacks. Applying rolling codes is also a good solution.

Since 31 B is usually not enough to send encrypted custom commands, it is recommended to use extended advertisements, described in [Use Case #2: I want to broadcast data (such as information about a product/artwork)](./use-case-2-i-want-to-broadcast-data-such-as-information-about-a-product-artwork).

Use the following advertising data types in the advertising data:

- Public Target Address (0x17)

- Service Data - 128-bit UUID (0x21) – define a random 128-bit UUID for your application, and add your custom data

- Manufacturer Specific Data (0xFF) – if your company has a manufacturer ID registered at Bluetooth SIG

If not only the switch but also the controlled device requires low power consumption, there are three solutions:

- Use periodic advertisements as described in [Use Case #3: I want to broadcast / send data to low power receivers](./use-case-3-i-want-to-broadcast-send-data-to-low-power-receivers).

- Let the controlled device advertise and create a connection as described in [Use Case #5: I want to connect to a device occasionally to exchange some data](./use-case-5-i-want-to-connect-to-a-device-occasionally-to-exchange-some-data).

- Use the same approach as described in this section, but set the scan window to a short interval while setting the scan interval to a long interval on the scanner device. On the switch, repeat the advertisement many times. For example, set the scan window to 30 ms, scan interval to 1 sec, set the advertisement interval to 20 ms and repeat the advertisements for 1.2 sec. This ensures that at least one advertisement will be received by the controlled device, while it is scanning with a 3% duty cycle.

**Bluetooth API to be used**:

- `sl_bt_advertiser_create_set()`

- `sl_bt_advertiser_set_timing()`

- `sl_bt_extended_advertiser_set_phy()`

- `sl_bt_advertiser_set_channel_map()`

- `sl_bt_legacy_advertiser_set_data()`

- `sl_bt_extended_advertiser_set_data()`

- `sl_bt_legacy_advertiser_start()`

- `sl_bt_extended_advertiser_start()`

- `sl_bt_system_set_tx_power()`

- `sl_bt_advertiser_configure()`

- `sl_bt_scanner_set_parameters()`

- `sl_bt_scanner_start()`

**Tips for low power consumption**:

- Consider the same recommendations that are described in [Use Case #1: I want to signal the presence of a device](./use-case-1-i-want-to-signal-the-presence-of-a-device), in [Use Case #2: I want to broadcast data (such as information about a product/artwork)](./use-case-2-i-want-to-broadcast-data-such-as-information-about-a-product-artwork), and in [Use Case #3: I want to broadcast/send data to low power receivers](./use-case-4-i-want-to-broadcast-send-data-from-a-low-power-transmitter).
