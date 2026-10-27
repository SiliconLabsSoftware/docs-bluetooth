# Bluetooth Stack Event Handling

The Bluetooth stack for the Wireless Geckos is an event-driven architecture, where events are handled in the main while loop of the application on bare metal.

## Commands from Multiple Tasks

It is possible to send Bluetooth commands from multiple OS tasks. All BGAPI command functions have automatic locking to make them thread-safe. Using `sl_bt_bluetooth_pend()` and `sl_bt_bluetooth_post()` is therefore no longer required for individual calls to the BGAPI.

The application only needs to use `sl_bt_bluetooth_pend()` and `sl_bt_bluetooth_post()` to protect sections of code where multiple commands need to be performed atomically in a thread-safe manner. This includes cases such as using `sl_bt_system_data_buffer_write()` to write data to the system buffer followed by a call to `sl_bt_extended_advertiser_set_long_data()` to set that data to an advertiser set. To synchronize access to the shared system buffer, the application needs to lock by calling `sl_bt_bluetooth_pend()` before `sl_bt_system_data_buffer_write()`, and release the lock by calling `sl_bt_bluetooth_post()` after `sl_bt_extended_advertiser_set_long_data ()`.
