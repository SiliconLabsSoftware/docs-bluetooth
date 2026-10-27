
# Pairing Processes

## Introduction

Silicon Labs Bluetooth stack implements Bluetooth security features, as described in [Bluetooth Low Energy Application Security Design Considerations in SDK v3.x](/bluetooth/{build-docspace-version}/bluetooth-application-security-design-considerations).

This involves pairing processes to set up a secure connection between two devices. There are five different pairing processes depending on the I/O capabilities of each devices, as follows:

- Just works [this results in an unauthenticated but encrypted connection!]
- Numeric Comparison [authenticated]
- Passkey Entry (Initiator displays, Responder inputs) [authenticated]
- Passkey Entry (Responder displays, Initiator inputs) [authenticated]
- Passkey Entry (Responder and Initiator inputs) [authenticated]

The process is selected based on the I/O capabilities as summarized in the following table:

![Process Table](resources/process-table.png?darkModeUrl=resources/process-table.png)

Each use case has a different sequence of events and commands which has to be followed. When implementing an application, the developer has to consider all scenarios that may happen with the I/O capabilities of the device. For example, if you have a device with DisplayOnly capability (you can display the 6 digit passkey but you have not inputs such as keys or buttons) and you are supposed to be a Responder (e.g., you use a mobile application that will always initiate the bonding), you have to implement the Just Works and the Passkey Entry (R displays, I inputs) processes in your application, as these are the possible scenarios according to the table above.

The pairing process is an interactive process because the user has to confirm the identity of the connecting devices, for example by typing in the passkey. This interaction is realized with events raised by the stack and with commands sent to the stack.

Note: You can enable automatic bonding confirmation by setting bit 3 in sm configuration flags to 0 (see sl_bt_sm_configure(flags,io_capabilities) ). In this case, “event sm_confirm_bonding” and “call sm_bonding_confirm” steps are skipped in each processes. Do not confuse this with sl_bt_sm_set_bondable_mode(1), which has to always be set to 1 to enable bonding processes. To learn more about sm configuration, see [Bluetooth Low Energy Application Security Design Considerations in SDK v3.x](/bluetooth/{build-docspace-version}/bluetooth-application-security-design-considerations).

## Just Works

In this use case you can't confirm the identity of the connecting devices, so no interaction is needed. Devices will pair with encryption but without authentication.

![Just Works](resources/just-works.png?darkModeUrl=resources/just-works.png)

## Numeric Comparison

Both devices will display a 6-digit passkey. The user has to confirm that the two devices display the same passkey by pressing a button. If the passkeys match, then there is no Man in the Middle (MITM) and the devices can exchange keys securely. Note: to get the two passkeys on the two devices at the same time, disable automatic bonding confirmation (bit 3 in sm configuration flags).

![Numeric Comparison](resources/numeric-cmp.png?darkModeUrl=resources/numeric-cmp.png)

## Passkey Entry: Responder Displays, Initiator Inputs

A passkey is displayed on the responder device, which has to be typed in with the use of the numeric keyboard on the initiator device.

![PassKey RD-II](resources/pk-rd-ii.png?darkModeUrl=resources/pk-rd-ii.png)

## Passkey Entry: Initiator Displays, Responder Inputs

A passkey is displayed on the initiator, which has to be input on the responder device to confirm authentication.

![PassKey ID-RI](resources/pk-id-ri.png?darkModeUrl=resources/pk-id-ri.png)

## Passkey Entry: Initiator and Responder Input

In this scenario, both devices have to input a passkey. By contrast to other scenarios, where the passkey was generated with either one of the devices, in this case, the user has to find out a key and enter the same key in both devices.

![PassKey II-RI](resources/pk-ii-ri.png?darkModeUrl=resources/pk-ii-ri.png)

## Example

This guide has a related code example, here: [Pairing Processes Example](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/security/pairing_processes_example).
