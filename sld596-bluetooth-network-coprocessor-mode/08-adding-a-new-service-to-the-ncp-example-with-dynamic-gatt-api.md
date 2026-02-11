# Adding New Attributes to the GATT Database

This page describes how to add a custom Bluetooth service to the **NCP** example using the dynamic GATT API. The service added here has one characteristic to receive data. When the central device (tablet/phone) writes this characteristic, the peripheral (starter kit – NCP Target) forwards this data to the NCP Host. The NCP Host prints out the actual data to the PC console.

![new service to NCP](resources/an1259-new-service-to-NCP-empty.png)

To implement this application, you need to make these changes:

- Modify the GATT database in the Host project

- Handle the GATT change event (sl_bt_evt_gatt_server_attribute_value) in the Host project

## Adding New Attributes Using the Configuration File

Beginning with SDK version 3.3, the NCP host sample applications contains a .btconf file with a basic GATT configuration. This configuration can be edited with the GATT Configurator. Although PC host examples are not handled by Simplicity Studio, .btconf files can still be edited individually. Open the Simplicity IDE perspective in Simplicity Studio and drag-and-drop the .btconf file onto the editor area. GATT Configurator will automatically open. Edit the file as described in [GATT Configurator User’s Guide for Bluetooth SDK v3.x](https://docs.silabs.com/bluetooth/latest/gatt-configurator-users-guide-ble-btmesh/) and save it. To be compatible with the code snippets, create a custom service and then add a custom characteristic with the following properties:

- ID:  my_data

- Read, Write, Indicate

- Value length: 20 bytes

Once the .btconf file is saved it must be turned into source code by running `make gattdb`. Run this command in the root folder of your example, where you can find the makefile. Note that the generator script requires installing Python 3 and the Jinja2 package by calling `pip install jinja2`.

![make gattdb](resources/an1259-v08-make-code-snippet.png)

The output (gatt_db.c / gatt_db.h) is located in the autogen folder. These values will be used as parameters for the dynamic GATT APIs. The database will be created (that is, built on the NCP target with the dynamic GATT API) automatically in the initialization phase before the boot event is sent to the application. The application is still able update the GATT database with the dynamic GATT commands, as described in the next section.

>**Note**: If your database contains included services, the included ones need to be defined before the ones including them.

## Adding New Attributes Using the Dynamic GATT API

The GATT database can be extended with the APIs provided by the Dynamic GATT Configurator component. For more details, see the Bluetooth API reference manual on [docs.silabs.com](https://docs.silabs.com/), section "GATT Database."

In the code snippet below, the custom service and characteristic is added to the database. The service will be a primary service, defined with a 16-byte long UUID, and it will be advertised.

The characteristic has the following properties:

- Read, Write, Indicate

- Valuelength: 20 bytes

- Valuemax length: 20 bytes

- 16-bytelong UUID

The service and the characteristic can be added any time after the boot event was received, but adding them in the boot event handler is suggested.

```C
uint8_t uuid_service[16] = {…} //define your 128bit service UUID, you can use a random number
uint8_t uuid_characteristic[16] = {…} //define your 128bit characteristic UUID

//create a session for the database update
sl_bt_gattdb_new_session(&session);
//add our service to the database, as an advertised primary service
sl_bt_gattdb_add_service(session, sl_bt_gattdb_primary_service, SL_BT_GATTDB_ADVERTISED_SERVICE, 16,
                 uuid_service, &service);
//define the following properties: read, write, indicate
property = (SL_BT_GATTDB_CHARACTERISTIC_READ | SL_BT_GATTDB_CHARACTERISTIC_INDICATE |
                 SL_BT_GATTDB_CHARACTERISTIC_WRITE);
//add our characteristic to the service
sl_bt_gattdb_add_uuid128_characteristic(session, service, property, 0, 0, uuid_characteristic,
                 sl_bt_gattdb_fixed_length_value, 20, 20, &value, &characteristic);
//activate the new service
sl_bt_gattdb_start_service(session, service);
//activate the new characteristic
sl_bt_gattdb_start_characteristic(session, characteristic);
//store the handle of the characteristic for future reference
gattdb_my_data = characteristic;
//save changes and close the database editing session
sl_bt_gattdb_commit(session);
```

>**Note**: The handle returned while adding a service or characteristic is not always the final one. It is only valid until `sl_bt_gattdb_commit` is called. You can get the actual final handle with the API call `sl_bt_gatt_server_find_attribute()`.

## Responding to the GATT Change Event

1. Add the callback function that reacts to the GATT change. In this case, it prints out the content of the characteristic.

   ```C
   void AttrValueChanged_my_data(uint8array *value)
   {
     uint8_t i;
     for (i = 0; i < value->len; i++){
         app_log("my_data[%d] = 0x%x \r\n",i,value->data[i]);
     }
     app_log("\r\n");
   }
   ```

2. Add the `sl_bt_evt_gatt_server_attribute_value_id` event to the switch case.

   ```C
   case sl_bt_evt_gatt_server_attribute_value_id:
     // Check if the event is because of the my_data changed by the remote GATT client
     if ( gattdb_my_data == evt->data.evt_gatt_server_attribute_value.attribute ){
       // Call my handler
      AttrValueChanged_my_data(&(evt->data.evt_gatt_server_attribute_value.value));
      }
   break;
   ```

>**Note**: If you edited the .btconf file, `gattdb_my_data` is defined in *gatt_db.h*. If you used the dynamic GATT API, `gattdb_my_data` is defined in the application as in the provided code snippet.

Now you can rebuild the host application. See the build process with MinGW in [Building the NCP Host Examples on Windows](./03-ncp-host-development.md#building-the-ncp-host-examples-on-windows).

## Testing

1. Start the host application from the *\exe* folder.

2. Once the PC is connected to WSTK (via UART), the WSTK starts advertising on Bluetooth.

3. If you connect via tablet/phone you can write the newly created `my_data` characteristic in the GATT. For this, you can use the Simplicity Connect app provided by Silicon Labs.

4. Browse to the `my_data` characteristic and write something to it. The data will be printed by the host application.

   ![my data characteristic](resources/an1259-my-data-characteristic.png)
