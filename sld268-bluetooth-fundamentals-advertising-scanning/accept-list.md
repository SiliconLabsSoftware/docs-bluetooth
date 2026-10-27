# Filter Accept List

## Introduction

Filter Accept List (old terminology: whitelisting) filters devices based on a list of Bluetooth addresses on both Central and Peripheral side. If the Accept List is on the Central side, scan results from non-listed devices are dropped and connecting to non-listed devices is not enabled. If the Accept List is on the Peripheral side, only listed devices can connect and receive scan responses (scan requests and connection requests from non-listed devices are dropped).

This feature was introduced in Bluetooth SDK version 2.11 and, and has been heavily revised in Bluetooth SDK version 6.0.0.

Accept List currently supports public, static, non-resolvable, and resolvable private random Bluetooth addresses. For the latter, the device has to be added to the [Resolving List](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-resolving-list) as well. Anonymous address is supported also. A Filter Accept List entry with this type matches all advertisements sent with no address. Additionally, if a device using a random address decides to change it after a power cycle, that will put the device out of the Accept List.

## Using Accept List

To add the Accept List feature to your application, follow the instructions below. For details about command parameters, see the Bluetooth Software API Reference Manual.

### Adding the Feature

 By default, this feature is not included in the stack and the **Device filtering with Bluetooth controller's Filter Accept List (bluetooth_feature_accept_list)** software component must be installed to get it work:

![Filter Accept List component](resources/v3-add-al.png?darkModeUrl=resources/v3-add-al.png)

### Configuring Filter Accept List

The only parameter to be configured, is the size of the list (at most, 127 records supported as of now). It can be modified from the component GUI or in the **sl_bt_accept_list_config.h** header file:

![Filter Accept List config](resources/v3-feature-config.png?darkModeUrl=resources/v3-feature-config.png)

```C
// <o SL_BT_CONFIG_ACCEPT_LIST_SIZE> Bluetooth Controller Filter Accept List Size <0..32:1>
// <i> Default: 2
// <i> Define the number of Filter Accept List entries to be used in doing device address filtering when scanning
#define SL_BT_CONFIG_ACCEPT_LIST_SIZE     (2)
```

### Utilizing Filter Accept List

Using the Filter Accept List is possible for both advertising and scanning. You may configure the advertising with the following API and two *flags*, as shown below:

```C
sl_status_t sl_bt_advertiser_configure(uint8_t advertising_set, uint32_t flags);

/**
 *
 * Use the Filter Accept List to filter scan requests received while performing
 * scannable advertising with this advertising set. By default, this flag is not
 * set and scan requests from all devices are processed. If the application sets
 * this flag, scan requests are processed only from those devices that the
 * application has added to the Filter Accept List.
 *
 * This configuration is supported only when the application has included the
 * Bluetooth component bluetooth_feature_accept_list.
 *
 * */
#define SL_BT_ADVERTISER_USE_FILTER_FOR_SCAN_REQUESTS       0x20      

/**
 *
 * Use the Filter Accept List to filter connection requests received while
 * performing connectable advertising with this advertising set. By default,
 * this flag is not set and connection requests from all devices are processed.
 * If the application sets this flag, connection requests are processed only
 * from those devices that the application has added to the Filter Accept List.
 *
 * This configuration is supported only when the application has included the
 * Bluetooth component bluetooth_feature_accept_list.
 *
 * */
#define SL_BT_ADVERTISER_USE_FILTER_FOR_CONNECTION_REQUESTS 0x40      
```

With scanning, you should use the API below, with the proper *filter_policy* configured:

```C
sl_status_t sl_bt_scanner_set_parameters_and_filter(uint8_t mode,
                                                    uint16_t interval,
                                                    uint16_t window,
                                                    uint32_t flags,
                                                    uint8_t filter_policy);

typedef enum 
{
sl_bt_scanner_filter_policy_basic_filtered      = 0x1, 
/**<
(0x1) Advertising and scan response PDUs are processed only from devices that the application has added to the Filter Accept List. For directed advertising, the target address must additionally match the identity address of the local device or be a Resolvable Private Address that is resolved to the local device by the Bluetooth controller.
*/

sl_bt_scanner_filter_policy_extended_filtered   = 0x3  
/**<
(0x3) Advertising and scan response PDUs are processed only from devices that the application has added to the Filter Accept List. For directed advertising, the target address must additionally match the identity address of the local device or be any Resolvable Private Address.
*/
} sl_bt_scanner_filter_policy_t;
```

### Adding devices manually to the Filter Accept List

There are two options adding a device to the Accept List, depending on whether we are bonded with it already or just scanned it.

```C
/***************************************************************************//**
 *
 * Add a device to the Filter Accept List based on its bonding handle.
 *
 */

sl_status_t sl_bt_accept_list_add_device_by_bonding(uint32_t bonding);

/***************************************************************************//**
 *
 * Add a device to the Filter Accept List based on its identity address.
 *
 */

sl_status_t sl_bt_accept_list_add_device_by_address(bd_addr address,
                                                    uint8_t address_type);
```

### Removing a device from the Filter Accept List

Deleting the bond will automatically remove the appropriate record from the Accept List as well. This is done with the command below:

```C
/***************************************************************************//**
 *
 * Delete the specified bonding or accept list filtering. The connection will be
 * closed if the remote device is connected currently.
 *
 */

sl_status_t sl_bt_sm_delete_bonding(uint8_t bonding);
```

You also have the option, to remove the device from the list by the bonding handle, but without deleting the bonding. Bondless removal is also possible as well:

```C
/***************************************************************************//**
 *
 * Remove a device from the Filter Accept List based on its bonding handle.
 *
 */

 sl_status_t sl_bt_accept_list_remove_device_by_bonding(uint32_t bonding);

 /***************************************************************************//**
 *
 * Remove a device from the Filter Accept List based on its identity address.
 *
 */

sl_status_t sl_bt_accept_list_remove_device_by_address(bd_addr address,
                                                       uint8_t address_type);
```

### Erasing the complete Filter Accept List

The command below deletes all bondings and clears the respective records from the Accept List.

```C
/***************************************************************************//**
 *
 * Delete all bondings, accept list filtering and device local identity
 * resolving key (IRK). All connections to affected devices are closed as well.
 *
 */

sl_status_t sl_bt_sm_delete_bondings();
```

Naturally, it is not necessary to delete all the bondings, in order to have the Accept List truncated:

```C
/***************************************************************************//**
 *
 * Remove all devices from the Filter Accept List.
 *
 */

sl_status_t sl_bt_accept_list_remove_all_devices();
```

## Test Setup

Two devices (WSTK radio boards) are used to demonstrate this feature.

**Device 1**: A radio board running the *Bluetooth - SoC Empty* example. This program advertises using the name **Empty Example**. The Bluetooth address used in the advertisement is public. In Simplicity Studio IDE, select the board and flash the *Bluetooth - SoC Empty* example from the demo section. Alternatively, use any device which is advertising using a public Bluetooth address.

**Device 2**: A radio board running a *modified version of the Bluetooth - NCP Empty* example. In Simplicity Studio IDE, create a project with the *Bluetooth - NCP Empty* example from the Software Examples section on the Launcher tab. Add the **Device filtering with Bluetooth controller's Filter Accept List (bluetooth_feature_accept_list)** software component to the project in the Software Components view, then build, and flash the project to the radio board.

Open the NCP Commander from the Tools Dialog. Connect to the NCP device. Click **Start Scan** under *Discover - Central* to start scanning. A list of advertising devices along with their addresses will populate, as shown in this figure. Make a note of the address of Device 1 or the device which you want to add to the accept list. Stop scanning.

![advertising extensions](resources/v3-scan-all.png?darkModeUrl=resources/v3-scan-all.png)

In the command prompt, execute the following commands, one at a time. Replace the Bluetooth address with the address of the device you want to add to the accept list.

```C
sl_bt_scanner_set_parameters_and_filter(0x0, 0x000F, 0x000F, 0, 0x1)
sl_bt_accept_list_add_device_by_address(00:0B:57:64:91:E2, 0x0)
```

Start scanning again. You will only see the device which was added to the Accept List, as shown in the figure below.

![device added to the accept list](resources/v3-scan-acceptlist.png?darkModeUrl=resources/v3-scan-acceptlist.png)
