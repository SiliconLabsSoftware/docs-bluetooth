# Conclusions

Co-located, strong Wi-Fi can have a substantial impact on Bluetooth and 802.15.4 performance. It can be improved by using the following unmanaged coexistence techniques:

1. [Implement Frequency Separation](./03-unmanaged-coexistence#implement-frequency-separation)

2. [Operate Wi-Fi with 20 MHz Bandwidth](./03-unmanaged-coexistence#operate-wi-fi-with-20-m-hz-bandwidth)

3. [Increase Antenna Isolation](./03-unmanaged-coexistence#increase-antenna-isolation)

4. [Implement Protocol and Stack Retry Mechanisms](./03-unmanaged-coexistence#implement-protocol-and-stack-retry-mechanisms)

5. [Remove FEM (or Operate FEM LNA in Bypass)](./03-unmanaged-coexistence#remove-fem-or-operate-fem-lna-in-bypass)

With market trends toward higher Wi-Fi TX power, higher Wi-Fi throughput, and integration of Wi-Fi, Bluetooth, and other radios, into the same device, unmanaged techniques alone may prove insufficient, so a managed coexistence solution is required. Even with a managed coexistence solution, all unmanaged coexistence recommendations are still necessary.

Silicon Labs recommends the following managed coexistence strategies:

- Wi-Fi/PTA devices providing 802.15.4-derived Packet Traffic Arbitration.

- Silicon Labs’ EFR32 PTA solution:

  - Implement one to four GPIOs as a combination of REQUEST, GRANT, PRIORITY, and RHO (two additional GPIOs are required to implement the PWM with High-Duty Cycle Wi-Fi feature for multi-EFR32 configurations).

  - Supports both single-EFR32 and multi-EFR32 configurations with single Wi-Fi/PTA interface.

  - Silicon Labs’ coexistence library and coexistence-hal-config.h #define settings to configure EFR32 PTA support for available GPIO pins and for compatibility with the chosen Wi-Fi/PTA device.

  - Silicon Labs’ API, supporting run-time PTA reconfiguration. For more information, see [Zigbee and OpenThread Coexistence with Wi-Fi](https://docs.silabs.com/multiprotocol/latest/zigbee-openthread-coexistence-wifi/) or [Bluetooth Coexistence with Wi-Fi](https://docs.silabs.com/bluetooth/latest/bluetooth-coexistence-with-wifi/).

## Bluetooth

Wi-Fi/Bluetooth coexistence test results show substantial Bluetooth performance improvements when PTA is utilized:

- Connection stability

  - Prevent user frustration with unstable product function as Wi-Fi throughput varies.

- Substantially reduced message failure with associated throughput improvement:

  - Improves end-node battery life.

  - Reduces message latency.

  - Bluetooth remains operational, even during high Wi-Fi duty cycles.

## 802.15.4

Wi-Fi/802.15.4 coexistence test results show substantial 802.15.4 performance improvements when PTA is utilized:

- Improved device join success:

  - However, device join utilizes broadcast messages, which are not retried.

  - If possible, device join success can be further improved by temporarily reducing Wi-Fi traffic during devices joining 802.15.4 network.

- Substantially reduced MAC retries:

  - Reduces message latency.

  - Improves end-node battery life.

  - Frequency separation remains important, as best managed coexistence performance is for “far-away” channels.

- Substantially reduced message failure:

  - 802.15.4 network remains operational, even during high Wi-Fi duty cycles.
