# Wi-Fi Coexistence Fundamentals

> **Note: This section replaces *UG103.17: Wi-Fi Coexistence Fundamentals*. Further updates to this user guide will be provided here**.

These pages describe methods to improve coexistence of 2.4 GHz IEEE 802.11b/g/n Wi-Fi and other 2.4 GHz radios such as Bluetooth®, Bluetooth Mesh, Bluetooth Low Energy, and IEEE 802.15.4-based radios such as Zigbee® and OpenThread. These techniques are applicable to the EFR32MGxx and EFR32BGxx families.

For additional details about the implementation of managed coexistence for EFR32 devices refer to the following application notes:

- [Bluetooth Coexistence with Wi-Fi](https://docs.silabs.com/bluetooth/latest/bluetooth-coexistence-with-wifi/)

- [Zigbee and OpenThread Coexistence with Wi-Fi](https://docs.silabs.com/multiprotocol/latest/zigbee-openthread-coexistence-wifi/)

- *AN1243: Timing and Test Data for EFR32 Coexistence with Wi-Fi* (available under non-disclosure from Silicon Labs Sales).

For evaluating the Silicon Labs EFR32 software coexistence solution, order EFR32MG Wireless SoC Starter Kit (WSTK) #SLWSTK6000B and Coexistence Backplane EVB (#SLWSTK-COEXBP). Detailed instructions for using the Starter Kit and Backplane EVB are found in [Silicon Labs Coexistence Development Kit (SLWSTK-COEXBP)](https://docs.silabs.com/multiprotocol/latest/coexistence-development-kit/). To see a demonstration of Wi-Fi coexistence, access links for ordering the WSTK and EVB, and to download additional coexistence documentation, visit the [Silicon Labs Wi-Fi Coexistence Learning Center](https://www.silabs.com/products/wireless/learning-center/wi-fi-coexistence).

## Silicon Labs' Fundamentals Series

Silicon Labs’ Fundamentals series covers topics that project managers, application designers, and developers should understand before beginning to work on an embedded networking solution using Silicon Labs chips, networking stacks such as EmberZNet PRO or Silicon Labs Bluetooth®, and associated development tools. The documents can be used as a starting place for anyone needing an introduction to developing wireless networking applications, or who is new to the Silicon Labs development environment.

## Coexistence of Radio Standards

The 2.4 GHz Industrial, Scientific and Medical (ISM) band supports Wi-Fi, Bluetooth, and 802.15.4. The simultaneous and co-located operation of these different 2.4 GHz radio standards can degrade performance of one or more of the radios. To improve interference robustness, each of the 2.4 GHz ISM radio standards support some level of collision avoidance and/or message retry capability. At low data throughput rates, low power levels, and/or sufficient physical separation, these 2.4 GHz ISM standards can co-exist without significant performance impacts. However, recent customer trends are making coexistence more difficult:

- Increased Wi-Fi transmit power level for “extended range.”

   +30 dBm Wi-Fi Access Points are now common.

- Increased Wi-Fi throughput.

   Depending on achievable Signal-to-Noise Ratio (SNR), high throughput requirements for file transfers and/or video streaming may result in high Wi-Fi duty cycle within the 2.4 GHz ISM band.

- Integrating Wi-Fi, Bluetooth, and 802.15.4 into the same device for gateway functionality.

   This is required by Home Automation and Security applications and provides easier end-node commissioning.

>**Note**: Zigbee and OpenThread devices (802.15.4) operate at less than +20 dBm transmit power level. With normal network activity, Zigbee/Thread solutions have a relatively low RF duty cycle and Bluetooth solutions implement Adaptive Frequency Hopping (AFH). Silicon Labs’ testing of Bluetooth blocked by Zigbee/Thread shows low impact on Bluetooth with AFH enabled and normal Zigbee/Thread network activity. However, if 100% RF duty cycle, Zigbee/Thread can degrade co-located Bluetooth performance.
