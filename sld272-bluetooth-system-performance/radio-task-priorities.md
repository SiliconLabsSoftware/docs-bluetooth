# Bluetooth Radio Task Priorities

The Bluetooth stack often needs to run multiple tasks that use the radio at the same time. For example, you can simultaneously send out advertisements and scan for new devices, keep alive multiple connections at the same time, or initiate a new connection while still scanning for new devices. You can even initiate a new connection while you are scanning for new devices, you are advertising, and you keep alive an already established connection.

In reality, the stack can only perform one task at a time (even though the tasks appear to run simultaneously). At any given time, the radio can be only configured to do one of the following:

- Send a packet
- Receive a packet
- Listen on a channel

 As a result, simultaneous radio tasks need to be prioritized using the radio scheduler, which decides the order that tasks are run in based on the priorities.

## Default Priorities

The Bluetooth stack distinguishes four radio tasks:

- Scanning
- Advertising
- Connection initiation
- Connection maintenance

**Scanning:** Scanning involves listening for advertisement packets on the three advertisement channels and periodically switching the channel. Scanning is a low-priority task, since missing a single advertisement is not a serious problem. Other task may need to interrupt scanning at any time, for example to keep the connection alive.

**Advertising:** Advertising involves repeatedly sending out advertisement packets. Advertisement is also a low-priority task. Because the exact timing of advertisements is not critical, advertisement packets can be delayed to free the radio for higher-priority tasks.

**Connection initiation:** Connection initiation consists of scanning to find the advertisement of the device to connect to and sending the connection request in the connection request window right after the advertisement. Connection initiation is a high-priority task for the following reasons:

1. Finding the very first advertisement to connect to quickly.
2. The connection request needs to be sent accurately in the connection request window.

**Connection maintenance:** Connection maintenance requires the device sending out at least one packet to the peer device and receive at least one packet from the peer device in every connection interval (Except when `slave latency` is defined, in which case the slave can skip some connection intervals.) These packets need very strict timing, they cannot be delayed, and missing lots of them may lead to connection loss, hence sending/receiving packets on a connection has a high priority.

The default priorities for the different tasks are listed here. Note that every task gets the minimum priority first which can be increased up to the maximum priority during time. This is explained in the next section.

The lowest possible priority is 255 and the highest priority is 0.

Task Type | Min. Priority | Max. Priority
----------|---------------|---------------
Scanning         | 191 | 143
Advertising      | 175 | 127
Connection init  |  55 |  15
Connection       | 135 |   0

## Dynamic Priorities

The Bluetooth stack uses dynamic priorities, which means that every radio task starts with a lower priority, and every time the task fails to be scheduled, its priority increases. This ensures that simultaneous tasks are served fairly. As shown in the table of the previous section, all four radio task types have a minimum and a maximum priority. By default, the task-specific minimum priority is assigned to the tasks, which can be increased up to the task-specific maximum priority. For example, connections have a higher priority than advertisements. However, if many advertisement packets were not sent due to connection maintenance, their priority can raise over the minimum connection priority, and for a short time advertisement can have a higher priority.

Another example is when multiple connections are open and their transmit/receive windows overlap. In this case, the connection with the highest priority is served. The priority of the served connection drops back to the minimum priority and the priority of other colliding connections is increased. Hence, all connections are served within a reasonable time. The priority is always increased such that it reaches 0 (highest) priority before the supervision timeout. For example, if you have two connections with 100 ms and 10 s supervision timeouts respectively, the priority of the first one increases quickly while the priority of the second one increases slowly. This ensures that all connections are served before their supervision timeout.

For more information about Bluetooth priority handling, see [Dynamic Multiprotocol User’s Guide](/bluetooth/{build-docspace-version}/multiprotocol-dynamic-ug).

## Changing Priorities

The default priorities are chosen so that they can be used well in most use cases. However, default priorities may not always provide an optimal solution. For example, if in your application it is not important to connect quickly but it is very important to send data in every connection interval via an already established connection, you can lower the priority of the connection initiation process.

The minimum and maximum priorities of each task type can be defined in a `sl_bt_bluetooth_ll_priorities` structure (find the definition of the structure in sl_bt_ll_config.h). To overwrite the default priorities, add a new line to `SL_BT_CONFIG_DEFAULT` in sl_bluetooth_config.h, as you see below:

```c
sl_bt_bluetooth_ll_priorities ll_priorities = { 191, 143,  //scan_min, scan_max
                                             175, 127,  //adv_min,  adv_max
                                             135,   0,  //conn_min, conn_max
                                             55,  15,   //init_min, init_max
                                             16,        //rail_mapping_offset
                                             16,        //rail_mapping_range
                                             0,         //afh_scan_interval
                                             4,4        //adv_step, scan_step
                                           };

// Bluetooth configuration parameters (see sl_bt_stack_config.h)
#define SL_BT_CONFIG_DEFAULT
{
    //...
    .bluetooth.linklayer_priorities = &ll_priorities,
    //...
};

```

*afh_scan_interval and adv_step/scan_step are not discussed here. *rail_mapping_offset* and *rail_mapping_range* are discussed in the next section.

Alternatively the priority table can also be set with `sl_bt_system_linklayer_configure()`, using the config key **sl_bt_system_linklayer_config_key_set_priority_table**.

## Priorities in Bluetooth + RAIL DMP Application

In a Bluetooth + RAIL DMP application your proprietary protocol also needs to use radio priorities. In this case, the Bluetooth priorities are mapped to a small range of the RAIL priorities, so that you can define radio tasks with a priority higher than the highest Bluetooth priority and with a priority lower than the lowest Bluetooth priority. By default, Bluetooth priorities are mapped to the 16..32 RAIL priority range meaning that the 0 Bluetooth priority corresponds to RAIL priority 16, and the 255 Bluetooth priority corresponds to RAIL priority 32. This can also be changed in the Bluetooth configuration structure with the last two parameters of the linklayer_priorities. The *offset* defines the highest RAIL priority (that corresponds to 0 Bluetooth priority) and the *range* defines the difference between the lowest and highest priority (by default 32 - 16 = 16).

![Priority Mapping](resources/priority-mapping.png?darkModeUrl=resources/priority-mapping.png)
