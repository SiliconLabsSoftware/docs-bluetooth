# Implementing Multiprotocol with Bluetooth

For details on how the RAIL/Bluetooth light/switch multiprotocol example was implemented, and for more information on developing a multiprotocol application with your own protocol on RAIL, see [Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL in GSDK v3.x and Higher](https://docs.silabs.com/multiprotocol/latest/multiprotocol-dynamic-ble-proprietary-on-rail/).

## Bluetooth Priorities

As opposed to Zigbee with statically defined priorities for different operation types, Bluetooth uses a range and offset approach to assign all tasks to a given area of the priority spectrum.

![Mapping of Bluetooth Priority Range to RAIL Priority Range](resources/sld485-image17.png)

In this example the Bluetooth priority range, which itself spans from 0 to 255, is mapped to a limited portion of the shared RAIL priority space.

Unlike Zigbee, Bluetooth has much more stringent timing requirements where missing a given slot may result in a connection terminating. Also Bluetooth has a range of different tasks like (potentially multiple) connections, advertisement, scanning, and Periodic Advertising with Responses (PAwR) transmissions and receptions.

| **No.** | **Name** | **Default Setting** | **Exit Criterion** |
|---------|----------|---------------------|--------------------|
| 1 | Connection | 135 to 0 | Connection Event Ends |
| 2 | Connection Initiation | 55 to 15 | Initiation Window Ends |
| 3 | Advertisement | 175 to 127 | Advertisement Event Ends |
| 4 | Scanner | 191 to 143 | Scan Window Ends |
| 5 | PAwR TX | 15 to 5 | Advertiser: PAwR Transmit Event Ends; Synchronizer: PAwR Response Slot Ends |
| 6 | PAwR RX | 20 to 10 | Advertiser: PAwR Response Slot Ends; Synchronizer: PAwR Response Slot Delay Ends |

In order to handle this the Bluetooth scheduler, whose priorities are mapped to the RAIL radio scheduler, takes into account the following parameters for each task:

1. Start Time

2. Minimum time

3. Maximum time

4. Priority

![Bluetooth Task](resources/sld485-image18.png)

If the start time is moved the total running time is reduced respectively, that is the slack is reduced. Also priorities can be dynamically adjusted.

### Connections

Connections have a relatively high priority. The start time of a connection cannot be moved.

The priority is dynamically increased by the Bluetooth scheduler the closer the connection gets to the supervision timeout, and reaches the maximum priority close to it. A TX packet in the TX queue also increases the priority of a connection.

### Connection Initiation

Connection initiation scans advertisements from target device to establish a connection. It has a higher priority compared to a scanner to allow more robust connection establishment.

### Advertisements

Advertisements by default have a lower priority and their start point can be moved. Start time and Maximum time are defined by the advertisement interval.

If an advertisement could not be sent out, the priority of advertisements increases slowly and is reset once an advertisement was successfully sent.

### Scanner

By default, these tasks have the lowest priority. Start, minimum and maximum time are defined by the scanning interval and window size. Scanning can continue even when interrupted by a higher priority task. If this happens the scan time is accumulated to make sure the desired scan window size is reached at each scanning interval.

As with advertisements, the priority is increased in case the desired scan interval or window size could not be previously met. It is reset back to its initial priority once the scan interval or window size has been met.

### Periodic Advertising with Responses (PAwR)

Sending Periodic Advertising with Responses has the highest priority by default over all other Bluetooth tasks, followed by receiving responses in PAwR to maintain synchronization in an electronic shelf label (ESL) network.

A PAwR task priority is increased if the task scheduling fails twice in a row. The priority is either increased by 1/6th of the priority range, or at least by one until the maximum priority has been reached. The task priority is reset back to the minimum after successful scheduling. The same procedure applies to both PAwR advertiser and synchronizer in both directions.

## Example of Bluetooth Scheduler Operation

This example illustrates how the Bluetooth scheduler will schedule three connection tasks and one advertisement task, each holding different priorities. In the following figures the gray part indicates the minimum runtime a task requires and the blue part indicates the maximum runtime the task can use and, if flexible, the region where the task can be moved. The following figure shows the initial setup.

![Task Scheduling Example: Setup](resources/sld485-image19.png)

As shown below, Conn1 is the first task to run as it does not overlap with any higher priority task.

![Task Scheduling Example: 1st Step](resources/sld485-image20.png)

Adv1 overlaps with the higher priority Conn2. Adv1 is flexible and therefore gets moved in, as illustrated in the following figure.

![Task Scheduling Example: 2nd Step](resources/sld485-image21.png)

Conn2 overlaps with higher priority task Conn4. As Conn2 is not flexible, the scheduling of Conn2 fails.

![Task Scheduling Example: 3rd Step](resources/sld485-image22.png)

Conn4 does not overlap with other tasks, therefore Conn1 end is adjusted to stop before Conn4 starts.

![Task Scheduling Example: 4th Step](resources/sld485-image23.png)

Finally, Adv1 is run. Conn4 is adjusted to end before Adv1 starts.

![Task Scheduling Example: 4th Step](resources/sld485-image24.png)

### Modifying Priorities

The "sl_bt_configuration_t" (v3.x)/"gecko_configuration_t" (v2.x) struct defines the sl_bt_stack_config_t struct, which contains the field “bluetooth.linklayer_priorities” that is a pointer to the priority configuration. If the pointer is NULL then the stack uses its default priorities as listed in [Bluetooth Priorities](#bluetooth-priorities), as well as this section.

In case the pointer is not null it must point to a struct of priority settings as defined below:

```text
typedef struct {
  uint8_t scan_min;
  uint8_t scan_max;
  uint8_t adv_min;
  uint8_t adv_max;
  uint8_t conn_min;
  uint8_t conn_max;
  uint8_t init_min;
  uint8_t init_max;
  uint8_t rail_mapping_offset;
  uint8_t rail_mapping_range;
  uint8_t _reserved;
  uint8_t adv_step;
  uint8_t scan_step;
  uint8_t pawr_tx_min;
  uint8_t pawr_tx_max;
  uint8_t pawr_rx_min;
  uint8_t pawr_rx_max;
} sl_bt_bluetooth_ll_priorities;
```

The parameters `scan_min`, `can_max`, `adv_min`, `adv_max`, `conn_min`, `conn_max`, `init_min`, and `init_max` define the minimum and maximum priorities for scanning, advertisement, connections, and initiations respectively. The priorities will move between the min and max boundaries as described in:

- [Connections](#connections)
- [Connection Initiation](#connection-initiation)
- [Advertisements](#advertisements)
- [Scanner](#scanner)

The RAIL mapping parameters, `rail_mapping_offset` and `rail_mapping_range`, define how the Bluetooth link layer priorities are mapped to the global RAIL radio scheduler priorities. The mapping of these values can be seen in [Bluetooth Priorities](#bluetooth-priorities). The default for both `rail_mapping_offset` and `rail_mapping_range` is 16.

The `adv_step` and `scan_step` parameters define the step size when the priority of scanning and advertising is changed dynamically.

Finally, the parameters `pawr_tx_min`, `pawr_tx_max`, `pawr_rx_min`, and `pawr_rx_max` define the priority range for the PAwR advertiser and synchronizer TX and RX events in each sub-event.
