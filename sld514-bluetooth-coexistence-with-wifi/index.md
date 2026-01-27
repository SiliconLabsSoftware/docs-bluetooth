# Bluetooth Coexistence with Wi-Fi

> **Note: This section replaces *AN1128: Bluetooth Coexistence with Wi-Fi*. Further updates to this application note will be provided here**.

These pages describe methods to improve coexistence of 2.4 GHz IEEE 802.11b/g/n Wi-Fi and Bluetooth® radios. These techniques are applicable to the EFR32MGx family and EFR32BGx family. This application note assumes you have a basic understanding of how Wi-Fi coexistence is implemented on EFR32 devices. For more information, see [Wi-Fi Coexistence Fundamentals](https://docs.silabs.com/multiprotocol/latest/multiprotocol-wifi-coexistence-fundamentals/).

Additional details about the implementation of managed coexistence are included in [Wi-Fi Coexistence Fundamentals](https://docs.silabs.com/multiprotocol/latest/multiprotocol-wifi-coexistence-fundamentals/) and *AN1243: Timing and Test Data for EFR32 Coexistence with Wi-Fi* (available under non-disclosure from Silicon Labs Sales).

- [PTA 3-Wire BLE Functional Overview](./02-pta-3-wire-ble-functional-overview) describes how to configure the Silicon Labs Packet Traffic Arbitration (PTA) for Bluetooth.

- [Support Software Setup](./03-pta-support-software-setup) describes how to configure the Silicon Labs Packet Traffic Arbitration (PTA) for Bluetooth.

- [Code Coexistence Extensions](./04-application-code-coexistence-extensions) describes how to use PRS for radio digital signal output.

- [Coexistence Backplane Evaluation Board (EVB)](./05-coexistence-backplane-evaluation-board-evb) explains how to order the EVB for evaluating the Silicon Labs EFR32 software coexistence solution.

**Notes**:

1. Not all coexistence support features are present in SDK versions earlier than Bluetooth 3.3.1.0 and Bluetooth Mesh 2.2.1.0. Users of Bluetooth SDK 2.13.7 or earlier and Bluetooth Mesh SDK 1.7.1 or earlier may see different features from those documented in this application note.

2. Throughput this application note “Bluetooth Low Energy” is referenced as “Bluetooth”.

3. This application note addresses Bluetooth coexistence applications using EFR32 devices as per Bluetooth Core Specification v5.0 Vol 6 “Low Energy Controller” (point-to-point) and as per Bluetooth Specification Mesh Profile v1.0 (mesh network). These two applications have different coexistence considerations and, where necessary, this application note differentiates using the following terms:

    - “Bluetooth device” to reference Bluetooth Core Specification v5.3 Vol 6 “Low Energy Controller” (point-to-point) operation.

    - “Bluetooth mesh device” or “Bluetooth mesh node” to reference Bluetooth Specification Mesh Profile v1.0 (mesh network) operation.
