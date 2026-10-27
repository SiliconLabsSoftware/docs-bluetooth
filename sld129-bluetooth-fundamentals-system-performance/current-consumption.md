# Optimizing Current Consumption in Bluetooth Low Energy Devices

## Introduction

Current consumption or, more generally, energy usage is a major concern in battery-powered products. Optimizing current consumption extends battery life and, as a result, makes better products. This document discusses how to optimize the current consumption.

## Description

The two main factors affecting current consumption in a Bluetooth Low Energy (BLE) device are the amount of power transmitted and the total amount of time that the radio is active (TX and RX).

The amount of transmit power required depends on the range required between central and peripheral. Range is greatly affected by the environment such as obstacles and the amount of 2.4 GHz traffic present. The first tip is not to transmit more power than required.

The amount of time that a radio is active is determined by how often the radio must transmit or receive and the length of time required to transmit or receive. The first, and probably most obvious, tip is to keep characteristics small. Do not use a 32 bit integer if 8 bits will do.

In general, the power consumption of a BLE device can be adjusted by fine-tuning parameters and configurations related to advertising and connection states.

## Advertising

### Advertising Interval

Advertising interval is adjustable, from 20 ms to 10.24 s (non-connectable: minimum is 100 ms). Increasing advertising interval can significantly decrease the average current consumption of a BLE device. For instance, increasing advertising interval from 100 ms to 1 s drops the average current consumption by 93%.

![Advertising Interval vs Current Consumption](resources/adv-int-vs-ua2.png?darkModeUrl=resources/adv-int-vs-ua2.png)

### Advertising TX Power Level

The transmit power is adjustable, from -26 dBm to +8 dBm (default is 8 dBm). 0 dBm is enough to cover about 10 to 15 m range, based on tests made with iBeacon example and Android phone. The transmission power can easily be changed in applications using the API call to `sl_bt_system_set_tx_power()`.

Changing the TX power from 8 dBm to 0 dBm can reduce the current consumption by more than 120% using 100 ms advertising interval, and 105% using 1 s advertising interval.

![TX power: 0 dBm vs 8 dBm](resources/tx-0dbm-vs-8dbm-2.png?darkModeUrl=resources/tx-0dbm-vs-8dbm-2.png)

Furthermore, if the LE Power Control feature is enabled (both on the central and the peripheral), the Bluetooth stack can automatically lower the TX power on connections, when the two devices are close to each other.

### Advertising Mode (Connectable / Non-Connectable)

Non-connectable mode supports only the TX operation, whereas connectable mode of advertising supports both TX and RX operations.

![Current Profile: Connectable vs Non-Connectable Advertising Mode](resources/connectable-vs-non-connectable.png?darkModeUrl=resources/connectable-vs-non-connectable.png)

![Average Current Consumption: Connectable vs Non-Connectable Advertising Mode](resources/conn-vs-non-conn-2.png?darkModeUrl=resources/conn-vs-non-conn-2.png)

### Deep Sleep Modes

If deep sleep is enabled (as in most of the examples), the device can enter **EM2** mode automatically between advertising events. Deep sleep is only disabled if a peripheral of software component (e.g., UART) disables it. For example, consider switching off debug logs via UART in your final code because UART may disable deep sleep.

![EM2 Sleep Mode Current Consumption](resources/em2.png?darkModeUrl=resources/em2.png)

In some cases, going to **EM3** and **EM4** between advertising may be possible to save energy. This, however, only applies to non-connectable advertisements, and should be solved by the application.

## Connection

### Connection Interval

As with advertising, the connection interval has a direct impact on the current consumption. The connection interval can be adjusted between 7.5 ms and 4 s and is an easy way to trade-off between latency/throughput and average current consumption.

![Connection with Empty Packet Transfer (Active Time 1.5 ms), and 15 ms Connection Interval](resources/connection-interval-1.png?darkModeUrl=resources/connection-interval-1.png)

The following graph shows the average current required to keep the connection up, with different connection intervals (at 0 dBm TX power). The RF duty cycle is calculated based on the 1.5 ms activity in each interval.

![average current required at different connection intervals](resources/connection-interval-2.png?darkModeUrl=resources/connection-interval-2.png)

### Peripheral Latency

Peripheral latency ensures that the peripheral device can skip N connection intervals if it does not have anything to transmit. Note, however, that the central device still needs to poll the peripheral at every connection interval. Peripheral latency can be set using `sl_bt_connection_set_parameters()` API.

![Average Current Consumption: Peripheral Latency OFF (Upper Graph) vs Peripheral Latency Value of 5 (Lower Graph)](resources/peripheral-latency.png?darkModeUrl=resources/peripheral-latency.png)

With a connection interval of 75 ms, in the above graphs, changing the peripheral latency value to 5 can drop down the average current consumption from 230 µA to 140 µA (40% saving).

### Connection TX Power Level

The same TX power level setting (`sl_bt_system_set_tx_power()`) applies to advertisements and connections.

![TX Power Level Settings](resources/tx-power-conn.png?darkModeUrl=resources/tx-power-conn.png)

### PHY (1M / 2M Coded PHY)

Bluetooth 5 introduced 2M PHY for faster throughput and higher energy efficiency. This can lower the average current by reducing the air time of the radio and allowing the MCU to sleep more. The following graph compares the current consumption for a *short packet* transmission over 1M (left) and 2M (right) PHYs connections with a connection interval of 25 ms, in both cases. Going from 1M to 2M PHY, the energy consumption is reduced by 15%. For larger data transmissions the gain can be even higher.

![Current Consumption: 1M (left) vs 2M (right) PHY ](resources/1m-vs-2m.png?darkModeUrl=resources/1m-vs-2m.png)

## Setting up

This simple example demonstrates the impact of advertising and connection parameters on the power consumption of a BLE device. Follow the instructions below and verify the result by using the Energy Profile perspective in SimplicityStudio.

1. Create a new `SOC - Empty` project in SimplicityStudio.
2. Open `app.c` file and replace the `system_boot` event handler with the following code.

    ```cpp
    case sl_bt_evt_system_boot_id:

    // Set TX power
      sc = sl_bt_system_set_tx_power(0, 80, &pwr_min, &pwr_max);
      app_assert_status(sc);
      // Extract unique ID from BT Address.
      sc = sl_bt_system_get_identity_address(&address, &address_type);
      app_assert_status(sc);

      // Create an advertising set.
      sc = sl_bt_advertiser_create_set(&advertising_set_handle);
      app_assert_status(sc);

      // Set advertising interval to 100ms.
      sc = sl_bt_advertiser_set_timing(
        advertising_set_handle,
        160, // min. adv. interval (milliseconds * 1.6)
        160, // max. adv. interval (milliseconds * 1.6)
        0,   // adv. duration
        0);  // max. num. adv. events
      app_assert_status(sc);
      // Start general advertising and enable connections.
      sc = sl_bt_advertiser_start(
        advertising_set_handle,
        advertiser_general_discoverable,
        advertiser_connectable_scannable);
      app_assert_status(sc);
      break;
    ```

3. Similarly, replace the `le_connection_opened` event handler with code below.

    ```cpp
    case sl_bt_evt_connection_opened_id:
        app_log_info("connection opened\r\n");
        /* go for a longer connection interval
         * latency = 0, timeout = 32000 ms
         *
         * timeout > (1+latency)*2*max_interval ms
         *  therefore, max latency <= [timeout/(2*max_interval)] - 1
         *
         *  should be possible to use latency = 15
         *
         * */

        //sl_bt_connection_set_parameters(evt->data.evt_connection_opened.connection, 700,760,1,600,0,0xffff);
    break;
    ```

4. Create `le_connection_parameters` event handler by adding following code under the comment `/* Add additional event handlers as your application requires */`

    ```cpp
    case sl_bt_evt_connection_parameters_id : {
        uint16_t interval = evt->data.evt_connection_parameters.interval, latency = evt->data.evt_connection_parameters.latency, timeout = evt->data.evt_connection_parameters.timeout;
        app_log_info("connection interval %d, latency %d, timeout %d ms\n", interval, latency, timeout * 10);
      }
    break;
    ```

5. The Bluetooth specification allows the advertising interval to be anywhere between 20 ms and 10.24 s. In this configuration, the application uses default settings for the advertising interval (100 ms), and +8 dBm on a BGM121. This configuration results in an average current of 419 µA.

    ![Current Consumption Advertising at +8 dBm and 100 ms Intervals](resources/advertising-100ms-8dbm.png?darkModeUrl=resources/advertising-100ms-8dbm.png)

6. Set the transmit power to 0 dBm by changing `sl_bt_system_set_tx_power(0, 80, &pwr_min, &pwr_max);` to `sl_bt_system_set_tx_power(0, 0, &pwr_min, &pwr_max);;` in `app.c`. Rebuild and download again to see the current consumption.

    ![Current Consumption Advertising at 0 dBm and 100 ms Intervals](resources/advertising-100ms-0dbm.png?darkModeUrl=resources/advertising-100ms-0dbm.png)

    By reducing the transmit power to 0 dBm while keeping the same 100 ms advertising interval, the average current is reduced to 190 µA.

7. Increase the advertising interval to 1000 ms by changing `sl_bt_advertiser_set_timing(advertising_set_handle, 160, 160, 0, 0);`; to `sl_bt_advertiser_set_timing(advertising_set_handle, 1600, 1600, 0, 0);` in `app.c`. Then, rebuild and download once more to see current consumption.

    ![Current Consumption Advertising at 0 dBm and 1000 ms Intervals](resources/advertising-1000ms-0dbm.png?darkModeUrl=resources/advertising-1000ms-0dbm.png)

    By increasing the advertising interval to 1000 ms, the average current is reduced to  21.5 µA.

10. Connect to the device using the Silicon Labs Simplicity Connect app and check the power consumption. The console will display a message indicating the actual connection parameters being used.

    ![Current Consumption Using Default Connection Parameters with an iPhone](resources/default-connection-parameters-withiphone.png?darkModeUrl=resources/default-connection-parameters-withiphone.png)

    The above figure shows the power consumption of the device when connected with an iOS phone. Using an Android device will produce slightly lower results since Android’s default connection interval is longer than that used by the iOS.

11. Another factor affecting average current consumption is the connection interval. A short connection interval results in higher throughput but also incurs an energy cost while a longer connection interval limits data throughput but also provides energy savings. The default connection intervals used by iOS and Android are currently 30 ms and 50 ms respectively. As shown below, the average current at 0 dBm and a 30 ms connection interval is approximately 228 µA.

    Change the connection interval to between 875 - 950 ms by uncommenting the line `//sl_bt_connection_set_parameters(evt->data.evt_connection_opened.connection, 700,760,1,600,0,0xffff);`. Rebuild and download and connect again with the mobile app. The console should show the new connection parameters in use.

    ![Current Consumption with 950 ms Connection Interval](resources/950ms-conn-interval.png?darkModeUrl=resources/950ms-conn-interval.png)

    By using a longer connection interval, 950 ms, the average current is to approximately 25 µA.

12. Change the peripheral latency to 5 by changing the code, as follows:

    ```cpp
    sl_bt_connection_set_parameters(evt->data.evt_le_connection_opened.connection, 700,760,1,600,0,0xffff);
    ```

    to

    ```cpp
    sl_bt_connection_set_parameters(evt->data.evt_le_connection_opened.connection, 700,760,5,3200,0,0xffff);;
    ```

Rebuild and download the application again and connect with the mobile app.
    ![Current Consumption with 950 ms Connection Interval and Latency of 5](resources/950ms-conn-interval-latency-5.png?darkModeUrl=resources/950ms-conn-interval-latency-5.png)

By increasing the peripheral latency, i.e., the number of intervals the peripheral can skip when it has no data to send, to 5 the average current drops below 10 µA.

 Select the connection parameters carefully so that several retries are possible to ensure stable connections. The supervision timeout must be greater than  `(1+latency)*2*max_connection_interval`. As a result, if the maximum connection interval is chosen to be 950 ms and the timeout is set to the maximum, 32 seconds then the maximum latency allowed is 15. However, this will result in an unstable connection, so the latency in this example is set to 5 to allow for 2 retries.

## Conclusion

There is always a trade off between current consumption and data throughput / latency. The application needs for battery life and throughput must be considered when choosing a connection interval. A longer connection interval will help improve battery life but will reduce throughput and may result in an unreliable or unstable connection unless the connection parameters are chosen carefully.
