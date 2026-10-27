# Co-Processors (NCP and RCP)

The Silicon Labs Bluetooth stack provides multiple APIs for the developer to access the Bluetooth functionality. Three modes are supported:

- Standalone mode, where both the Bluetooth stack and the application run in an EFR32SoC or module. The application can be developed with C programming language.
- Network Co-Processor (NCP) mode, where the Bluetooth stack runs in an EFR32 and the application runs on a separate host MCU. For this use case, the Bluetooth stack can be configured into NCP mode where the API is exposed over a serial interface such as UART.
- Radio Co-Processor (RCP) mode, where only the Link Layer of the Bluetooth stack runs on the EFR32, and the Host Layer of the stack, as well as the application, runs on a separate host MCU or PC. In this use case, the Host Layer is developed by a third party, since Silicon Labs’ Bluetooth stack is only built for EFR32 SoCs / modules. The Link Layer and the host layer communicate via HCI (Host-Controller Interface), which is a standard interface between the two layers. The HCI can be accessed via UART following the Bluetooth SIG's UART (H4) transport protocol.

This section provides additional detail on the last two.

- [**Using the Bluetooth Stack in Network Co-Processor Mode**](/bluetooth/{build-docspace-version}/bluetooth-network-coprocessor-mode): Describes how to configure the NCP target and how to program the NCP host when using the Bluetooth Stack in Network Co-Processor mode
- [**Enabling a Radio Co-Processor using the Bluetooth Controller and HCI Functions**](/bluetooth/{build-docspace-version}/enabling-rcp-using-bt-hci): Gives a short overview of the standard Host Controller Interface (HCI) and how to use it with a Silicon Labs Bluetooth LE controller.
