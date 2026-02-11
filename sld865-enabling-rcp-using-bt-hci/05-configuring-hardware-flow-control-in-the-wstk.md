# Configuring Hardware Flow Control In the WSTK

Hardware flow control can be enabled or disabled between the UART controller of the WSTK and Silicon Labs SoC. In the SoC, hardware flow control can be configured with the configuration parameter shown in Section [Using an Example Application](./02-enabling-hci-functionality.md#using-an-example-application). In the WSTK the hardware flow control can be configured as described in this section. The example below shows how to enable hardware flow control.

>**Important**: If the hardware flow control settings are not the same in the SoC and WSTK, the HCI will not work.

1. Open Simplicity Studio and, in the Debug Adapters view, right-click the target device.

2. Select **Connect**.

3. Right-click the device again and select **Launch Console**.

4. Select the admin tab.

5. Set flow control with the following command:

    ```sh
        WSTK\> serial vcom config handshake rtscts
        RTS handshake enabled
        CTS handshake enabled
        Serial configuration saved
    ```

6. Check the configuration with the following command:

    ```sh
    WSTK\> serial vcom
    ----- Virtual COM port -----
    Stored port speed  : 115200
    Active port speed  : 115226
    Stored handshake   : rtscts
    Actual handshake   : rtscts
    RTS Asserted - Ready to Receive.
    ```

The flow control can be disabled by setting *handshake* parameter to *none* in step 5 above.
