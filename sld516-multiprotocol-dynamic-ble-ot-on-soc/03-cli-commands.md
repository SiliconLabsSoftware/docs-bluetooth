# CLI Commands

If you have used the **ot-cli-ftd** sample application, the OpenThread commands available in the **ot-ble-dmp** app are identical. Type `help` at the prompt to see a list. A complete OpenThread CLI reference is available here:

[OpenThread CLI Reference](https://github.com/openthread/openthread/blob/master/src/cli/README.md)

A quick tutorial on using the CLI to form a two-node OpenThread network and send a ping is available here:

[OpenThread CLI Reference](https://github.com/openthread/openthread/tree/master/examples/apps/cli)

The **ot-ble-dmp** app adds a set of Bluetooth commands that can be used to exercise the bluetooth stack. Type `ble` at the prompt to see a list of subcommands:

- `get_address`

- `create_adv_set`

- `set_adv_timing`

- `set_adv_random_address`

- `start_adv`

- `stop_adv`

- `start_discovery`

- `set_conn_timing`

- `conn_open`

- `conn_close`

These commands are implemented in the bluetooth_cli.c file, and each of them calls a corresponding Bluetooth C API function. For detailed documentation on the underlying functions, see [Developing with Silicon Labs Bluetooth Low Energy](https://docs.silabs.com/bluetooth/latest). Note that the C API prefixes for the Bluetooth SDK changed from `gecko_` to `sl_bt_` in version 3.0. See *AN1255: Transitioning from the v2.x to the v3.x Bluetooth® SDK* for more information about this and other BGAPI changes.

`ble get_address`

- Prints out the public Bluetooth address.

- Example: `ble get_address`

- Calls `sl_bt_system_get_identity_address()`

`ble create_adv_set`

- Create an advertising set. Must be called to obtain a handle for use in the other advertising commands.

- Example: `ble create_adv_set`

- Calls `sl_bt_advertiser_create_set()`

`ble set_adv_timing <handle> <interval_min> <interval_max> <duration> <max_events>`

- Set the advertising timing parameters of the given advertising set.

- Example: `ble set_adv_timing 0 160 320 0 0`

- Calls `sl_bt_advertiser_set_timing()`

`ble set_adv_random_address <handle>`

- Set the advertiser on an advertising set to use a random address.

- Example: `ble set_adv_random_address 1`

- Calls `sl_bt_advertiser_set_random_address()`

`ble start_adv <handle> <discoverableMode> <connectableMode>`

- Starts advertising on a given advertising set with specified discoverable and connectable modes.

- Example: `ble start_adv 0 2 2`

- Calls `sl_bt_advertiser_start()`

`ble stop_adv`

- Stops advertising on the given handle.

- Example: `ble stop_adv`

- Calls `sl_bt_advertiser_stop()`

`ble start_discovery <mode>`

- Scans for advertising devices.

- Example: `ble start_discovery 1`

- Calls `sl_bt_scanner_start()`

`ble set_conn_timing <min_interval> <max_interval> <latency> <timeout>`

- Sets the default Bluetooth connection parameters.

- Example: ble set_conn_timing 6 400 0 800

- Calls sl_bt_connection_set_default_parameters()

`ble conn_open <address> <address_type>`

- Connects to an advertising device. Address type 0=public address, 1=random address. Initiating phy argument hard coded to 1.

- Example: `ble conn_open 80fd34a198bf 0`

- Calls `sl_bt_connection_open()`

`ble conn_close <handle>`

- Closes a Bluetooth connection.

- Example: `ble conn_close 0`

- Calls `sl_bt_connection_close()`

## Establishing a Bluetooth Connection Between Two Nodes

To establish a Bluetooth connection, the client starts advertising on advertising set 0 with modes discoverable and connectable. The server connects using the client's public address.

CLIENT:

```text
> ble create_adv_set
ble create_adv_set
success handle=0
>
> ble start_adv 0 2 2
ble start_adv 0 2 2
success
>
> ble get_address
ble get_address
BLE address: 90fd9f7b5d39
```

SERVER:

```text
> ble conn_open 90fd9f7b5d39 0
ble conn_open 90fd9f7b5d39 0
success
>
> BLE connection opened handle=1 address=90fd9f7b5d39 address_type=1 master=1 advertising_set=255
BLE connection parameters handle=1 interval=40 latency=0 timeout=100 security_mode=0 txsize=27
BLE event: 0x40800a0
BLE event: 0x900a0
BLE connection parameters handle=1 interval=40 latency=0 timeout=100 security_mode=0 txsize=251
```
