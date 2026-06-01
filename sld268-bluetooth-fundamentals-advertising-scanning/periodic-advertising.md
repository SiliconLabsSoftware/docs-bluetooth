
# Periodic Advertising

## Introduction

Periodic advertising is a Bluetooth 5.0 feature based on extended advertisements. It allows non-connectable advertisements to be sent at a fixed interval where advertising data can change between those intervals. One or more scanners can then listen for these advertisements, which is a form of multicast. The fact that, due to the fix interval, the scanners can go to sleep between two advertisement events instead of being in receive mode all the time is a huge advantage compared to other advertising modes.

Important points about Periodic Advertising are as follows:

1. You do not need to explicitly enable the extended advertiser when periodic advertising is enabled. The only way to get the pointer to the Periodic Advertising train is through the SyncInfo field of the extended advertiser. As a result, starting extended advertisement is mandatory and it is enabled automatically.
2. At least one advertisement packet needs to be sent to enable periodic advertising.
3. It will use the same PHY as the auxiliary packet. Each periodic advertiser has the same parameters as connection. Additionally, the channel is also determined the same way as connection using the Channel Selection Algorithm #2.
4. Because it is based on extended advertisements, it uses data channels as opposed to advertisements channels.
5. Periodic advertising supports Adaptive Frequency Hopping (AFH). Similar to Bluetooth connections, the advertiser and synchronized scanners can dynamically avoid channels experiencing high interference, improving robustness and reliability in congested RF environments.


## Concept

The overall process is shown in the figure below.

![Periodic Advertising](resources/sld268-figure-6.png?darkModeUrl=resources/sld268-figure-6.png)

Periodic advertising mode is indicated with the ADV_EXT_IND packets (legacy advertisement) sent on primary (advertising) channels, which point to AUX_ADV_IND packets (extended advertisement), containing the information about the periodic advertisement, such as interval, hopping sequence, advertiser address, and so on. The advertiser will also send AUX_SYNC_IND packets (periodic advertisement) at the identified interval containing the actual periodic advertisement data.

![Timing of Periodic Advertising](resources/sld268-figure-1.jpg?darkModeUrl=resources/sld268-figure-1.jpg)

If the data for a periodic advertisement does not fit in a single packet, the AUX_SYNC_IND packet can be followed by one or more AUX_CHAIN_IND packets. Together, AUX_SYNC_IND and AUX_CHAIN_IND form a sequence of advertisements known as a periodic advertising train.

Periodic advertising also supports AFH. Because periodic advertisements use data channels and Channel Selection Algorithm #2, the channel map can be updated to exclude channels affected by interference. This allows the periodic advertising train to adapt to changing RF conditions, similar to Bluetooth LE connections.


![Periodic Advertising Train](resources/sld268-figure-4.png?darkModeUrl=resources/sld268-figure-4.png)

The advertiser will periodically send new AUX_ADV_IND packets, so that new scanners can synchronize to the data stream or existing scanners can resume a lost sync. Silicon Labs' Bluetooth stack sends AUX_ADV_IND packet before every periodic advertisement so that new scanners can quickly synchronize.

## Periodic Advertising Modes

- **Advertiser** - Advertising device is sending periodic advertisements. The device also sends extended advertisements to enable syncing on the periodic advertising at any time. This is shown in the [Concept](#concept) section.

- **Scanner** - Users will get the address and SID (advertising set ID) values by scanning for extended advertisements. The device will synchronize on periodic advertising with user-specified address and SID. Synchronization will only happen when scanning is enabled. When the device has received at least one AUX_SYNC_IND packet, it is in synchronized mode. The device will then report received AUX_SYNC_IND packets. After the device has synchronized, you can stop scanning. From this point on, the device switches to receive mode only when a AUX_SYNC_IND packet is expected.

> **Note**: Periodic advertising is a new feature, which is not yet implemented on Phones. Hence, periodic advertisement packets will not be visible in mobile platforms and in Bluetooth scanning apps.

The following diagram summarizes the procedure involving both the advertiser and scanner.

![Flowchart of Periodic Advertiser and Scanner](resources/sld268-figure-7.png?darkModeUrl=resources/sld268-figure-7.png)

## Advertiser Implementation

### Configuration

The *max_advertisers* in the Bluetooth configuration structure also configures the maximum number of periodic advertisers. You can set the number of advertisers by opening the Bluetooth Core configurator tab from the Software Components:

![Bluetooth Core Configurator](resources/v3-add-adv.png?darkModeUrl=resources/v3-add-adv.png)

### Enabling the Feature

You can add the periodic advertising feature from the Software components for the advertiser:

![add the periodic advertising feature](resources/v3-add-feature.png?darkModeUrl=resources/v3-add-feature.png)

This will also initialize the feature.

### Enable/Disable Periodic Advertising

This command can be used to start periodic advertising on the given advertisement set. Note, that the set must be created beforehand with `sl_bt_advertiser_create_set()`. The stack will automatically start the extended advertisement containing the sync info along with the periodic advertisement.

```C
// Enable periodic advertisement
sl_bt_advertiser_start_periodic_advertising (uint8_t   	handle,
                                             uint16_t  	interval_min,
                                             uint16_t  	interval_max,
                                             uint32_t  	flags)

// Disable
sl_bt_advertiser_stop_periodic_advertising(uint8_t  handle)
```

### Set Periodic Advertisement Data

This command sets data that will be sent in periodic advertisement. Execute this command every time you want to change the periodic advertisement data. If periodic advertising is currently disabled for the specified advertising set, data is kept by the Controller and used after periodic advertising is enabled for that set. Data is discarded when the advertising set is removed.

```C
sl_bt_advertiser_set_data (uint8_t  handle,
                           uint8_t  packet_type,
                           size_t   adv_data_len,
                           const uint8_t *  adv_data)
```

The value of *packet_type* for Periodic advertisements is 8.

The command above can be used to set an advertisement data with a maximum length of 191 bytes. It is also possible to set long advertisement data for periodic advertisements with `sl_bt_advertiser_set_long_data()`. For details about the usage of this function see the following code example: [Chained Advertisements](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/advertising/chained_advertisement).

> Note: Periodic advertisement has the highest priority in the current implementation. For collisions, Periodic advertisements always overrides other advertisements. If you see collisions, try increasing the interval of advertisements.

## Scanner Implementation

### Configuration

*max_periodic_sync* in the Bluetooth config configures the maximum number of synchronizations the Bluetooth stack needs to support. The value can be set in the Bluetooth Core configuration, as shown below:

![configure maximum synchronizations](resources/v3-add-adv.png?darkModeUrl=resources/v3-add-adv.png)

### Enabling the Feature

To enable Periodic Advertisement Synchronization in the Bluetooth stack, add the feature to your project, as shown below:

![add the synchronization feature](resources/v3-add-sync.png?darkModeUrl=resources/v3-add-sync.png)

This will also initialize the feature.

### Establish Synchronization

This command can be used to establish a synchronization with periodic advertisement from the specified advertiser and begin receiving periodic advertisement packets. 

```C
sl_status_t sl_bt_sync_open (bd_addr  address,
                            uint8_t  address_type,
                            uint8_t  adv_sid,
                            uint16_t *  sync)
```

The *adv_sid* parameter value must match the Advertising SID subfield in the ADI (Advertise Data Info) field of the received advertisement for it to be used to synchronize. It can be obtained from the event *sl_bt_evt_scanner_scan_report_id*.

```C
evt->data.evt_scanner_scan_report.adv_sid
```

The timing parameters can be set with the command:

```C
sl_status_t sl_bt_sync_set_parameters(uint16_t  	skip,
                                      uint16_t  	timeout,
                                      uint32_t  	flags)
```

The *skip* parameter specifies the maximum number of consecutive periodic advertisement events that the receiver may skip after successfully receiving a periodic advertisement packet.
The *timeout* parameter specifies the maximum permitted time between successful receives. If this time is exceeded, synchronization is lost.

The advertiser *address_type* and advertiser *address* parameters specify the periodic advertising device to listen to. It can be obtained from the event *sl_bt_evt_scanner_scan_report_id*

```C
evt->data.evt_scanner_scan_report.address,
evt->data.evt_scanner_scan_report.address_type
```

This event is the result of successful synchronization:

```c
sl_bt_evt_sync_opened_id
```

Whenever a periodic advertisement packet is received, this event is generated:

```c
sl_bt_evt_sync_data_id
```

### Close Synchronization

This command closes a synchronization with periodic advertisement or cancels an ongoing synchronization establishment procedure.

```C
sl_bt_sync_close(uint8 sync)
```

*sync* is the periodic advertisement synchronization handle.

This event is generated when the synchronization is closed, lost, or when an ongoing sync process is canceled:

```c
sl_bt_evt_sync_closed_id
```

## Example

This guide has a related code example, here: [Periodic Advertisement Example](https://github.com/SiliconLabs/bluetooth_stack_features/tree/master/advertising/periodic_advertisement).
