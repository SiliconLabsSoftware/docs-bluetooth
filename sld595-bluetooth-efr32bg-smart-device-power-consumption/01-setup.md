# Setup

1. Obtain a mainboard and radio board.

2. Build and install a test application, as described below.

3. Configure the mainboard as described in *AN969: Measuring Power Consumption in Wireless Gecko Devices*.

To observe power consumption, the device must be programmed with a suitable test application. This application note addresses the power measurement of EFR32 devices during Bluetooth legacy advertising and Bluetooth extended advertising with Constant Tone Extension (CTE).

For legacy advertising, a beaconing application is an excellent reference for power consumption evaluation because it showcases the real-world transmit (TX) and sleep currents of the device by default. With minor modifications to the application code, the receive (RX) current can be verified as well. Similarly, for a CTE packet transmission, a CTE transmitter AoA asset tag application using extended advertising is an excellent reference for power consumption evaluation. For both cases, the application note demonstrates and analyzes the power transients of the device at 0 dBm TX power.

If you have not already done so, install Simplicity Studio and the Gecko SDK. The Gecko SDK includes several software examples to create Bluetooth application projects.

## Legacy Advertising

Select the **Bluetooth SOC – iBeacon** example. This provides an iBeacon device implementation that sends non-connectable advertisements in iBeacon format. The iBeacon Service gives Bluetooth accessories a simple and convenient way to send iBeacons to iOS devices. This example demonstrates the power consumption at 0 dBm TX power. Follow the directions in the [getting started guide](/bluetooth/{build-docspace-version}/bluetooth-getting-started-overview) applicable to your version of the SDK to build and flash the example project to the device.

The SoC – iBeacon application sets the device to broadcast in a non-connectable mode, which means that only TX and sleep events are observed in the current profile. To observe RX events, edit the application script to make the device broadcast in a connectable mode as follows:

If you use Simplicity Studio 4/Bluetooth SDK 2.x:

1. Open the **main.c** file in the SoC – iBeacon project by double-clicking it.

2. Edit the parameters for **gecko_cmd_le_gap_start_advertising** in the `bcnSetupAdvBeaconing` function.

    ```C
    gecko_cmd_le_gap_start_advertising(0, le_gap_user_data, le_gap_undirected_connectable)
    ```

3. Rebuild the project and program the test device with the new image.

If you use Simplicity Studio 5/Bluetooth SDK 3.x:

1. Open the **sl_bluetooth_config.h** file inside the config folder of the SoC – iBeacon project.

2. Change the maximum transmit power to 0 dBm.

    ```C
    #define SL_BT_CONFIG_MAX_TX_POWER		(0)
    ```

3. Open the **app.c** file in the SoC – iBeacon project by double-clicking it.

4. Edit the parameters for **sl_bt_advertiser_start** in the `bcn_setup_adv_beaconing` function.

    ```C
    sc = sl_bt_legacy_advertiser_start(
                              advertising_set_handle,
                              sl_bt_legacy_advertiser_connectable);
    ```

5. Build the project and program the test device with the new image.

## Extended Advertising with Constant Tone Extension (CTE)

Select the **Bluetooth SoC – AoA Asset Tag** example. This provides a CTE transmitter device implementation that sends CTE packets using extended advertisements (i.e Silabs enhanced mode). The Constant Tone Extension Service offers a simple and convenient way to send CTE packets using extended advertising. Follow the directions in [Getting Started](/bluetooth/{build-docspace-version}/bluetooth-getting-started-overview) to build and flash the example project to the device.

The Bluetooth SoC – AoA Asset Tag application enables the debug messages by default. Disable this feature to avoid the power consumption overhead of the UART. In addition, the example has two advertising sets: one connectable legacy advertising and another non-connectable extended advertising with CTE. For this project, disable the legacy advertising.

Make the following changes to the SoC – AoA Asset Tag project:

1. Open the **sl_bluetooth_config.h** file inside the config folder of the SoC – iBeacon project.

2. Change the maximum transmit power to 0 dBm.

    ```C
    #define SL_BT_CONFIG_MAX_TX_POWER		(0)
    ```

3. Disable UART debugging:

   - Open the project configurator (*.slcp) file in the project.

   - Select the **SOFTWARE COMPONENTS** tab.

   - Uninstall the **Application > Utility > Log** component.

   - Uninstall **Services > IO Stream > IO Stream: RETARGET STDIO**.

   - Uninstall the **Services > IO Stream > IO Stream: USART** component with the default instance name: **vcom**.

   - Go to **Platform > Board Control** and disable Virtual COM UART.

   - Open the **app.c** file and comment out `#include app_log.h` header file and all the calls to `app_log_info()` API.

4. Comment out the following code part in **app.c** which starts the legacy advertising.

    ```C
    sc = sl_bt_legacy_advertiser_start(
                              advertising_set_handle,
                              sl_bt_legacy_advertiser_connectable_scannable);
    ```

5. Build the project and program the test device with the new image.

Note that the CTE feature is only supported by specific series 2 devices. The measurement results in this documentation are from an EFR32BG22 device.
