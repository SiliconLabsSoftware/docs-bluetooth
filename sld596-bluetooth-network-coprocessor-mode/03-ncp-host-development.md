# NCP Host Development

This section introduces the Bluetooth NCP Commander tool, which can be used to send BGAPI commands from a graphical user interface. It then walks through the process of building the PC Host examples provided in the Bluetooth SDK. And finally, it describes using Python for host side development.

## Bluetooth NCP Commander

Bluetooth NCP Commander is an easy-to-use tool that can be used for testing different stack features, by sending BGAPI commands to the target device. The tool has two versions: a version built into Simplicity Studio, which makes it easy to connect to your development kit and start testing, and a standalone version to test a board in an environment where Simplicity Studio cannot be installed, or if you want to test a custom board that can be accessed on UART interface, but not through a Simplicity Studio supported debug adapter using VCOM.

### Built-in Version

1. To open the built-in Bluetooth NCP Commander, select the target board in the **Debug Adapters** view, and check that the preferred SDK is set to **Gecko SDK Suite vn.n.n**. Select the **Compatible Tools** tab, and click **Launch** next to Bluetooth NCP Commander.

   ![Compatible Tools](resources/an1259-v14-tools-tab.png)

   Alternatively, you can open the built-in Bluetooth NCP Commander from the **Tools** menu.

   ![Tools dialog](resources/an1259-v14-tools-menu.png)

2. Select the target device, and click **Connect**.

   ![Connection Manager](resources/an1259-v14-connection-manager.png)

### Standalone Version

1. To open the standalone tool, navigate to *C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\ncp_commander*, and start NcpCommander.exe.

2. In the standalone tool, provide the UART interface settings, and then select the COM port on which the device can be accessed.

   ![Serial Connection Manager](resources/an1259-v14-serial-connection-manager.png)

### Bluetooth NCP Commander Functions

The following procedure covers most NCP Commander functions.

1. After the device is connected, you should see the result of the ``sl_bt_system_get_identity_address`` command displayed in green:

   ![NCP Commander](resources/an1259-v14-ncp-commander-open.png)

2. Unlike SoC examples, the NCP demo does not have a built-in GATT database. It expects the host to build the GATT database using the dynamic GATT database BGAPI commands. To create a basic GATT database, select the Local GATT menu, and click **Create Basic GATT**. This triggers a series of BGAPI commands that will build a basic database. You can modify this GATT database as you want. You can also change the device name here by changing the value of the Device Name characteristic.

   ![Create Local GATT](resources/an1259-v14-ncp-commander-create-local-gatt.png)

3. To extend the database with new services, characteristics, and descriptors, click **Add Service**. You can then add new characteristics for the service.

   ![Commander add service](resources/an1259-v14-ncp-commander-add-service.png)

4. To read out the GATT database from the device, click **Local GATT Database**. The smart console also supports API calls for creating entries.

   ![Commander API method](resources/an1259-v14-ncp-commander-api-method.png)

5. To start advertising your device so that other devices can discover it and connect to it, in the Advertise menu click '**+**' (Create Set) to create an advertiser set.

   ![Commander start advertising](resources/an1259-v14-ncp-commander-start-advertising.png)

6. To populate the advertisement payload with the device name, set the Advertising Data Type to **Generated data** and click **Start** to start advertising.

   ![Advertising Data Type](resources/an1259-v14-ncp-commander-start-advertising-2.png)

7. When advertising, the NCP target example accepts Bluetooth connections. If you connect to the mainboard or with another central device (for example with your phone), you can see the events and commands on the log.

   ![Commander log view](resources/an1259-v14-ncp-commander-log-view.png)

8. You can also issue commands manually. For example, you can issue the 'system hello' command at any time to verify that communication between the host and the device is working. The Smart Console provides auto-completion and documentation for the possible commands. To open/close the documentation, click the arrows at the right side of the input field.

   ![Commander issue commands](resources/an1259-v14-ncp-commander-issue-commands.png)

9. To create periodic advertisement sets, select **Advertisement mode: Periodic**. To set the content of the packet, use the **Edit** option next to "Periodic Advertising Packets".

   ![Commander periodic advertising](resources/an1259-v14-ncp-commander-periodic-advertising.png)

     This opens a new dialog where you can edit the contents of the package. Click **Set Data** after the data is edited.
  
     ![Commander edit packet](resources/an1259-v14-ncp-commander-edit-packet.png)

10. It is also possible to synchronize to periodic advertisement trains. To do this, click **Synchronization** on the left menu, input the Advertiser Address and advertising Set identifier, and click **Open Synchronization**.

    ![Commander synchronize streams](resources/an1259-v14-ncp-commander-synchronize-streams.png)

11. NCP Commander also provides a simple scripting feature. You can create or import an existing script with the controls on the top right corner. You can use any BGAPI commands in the script, but there are no additional features, such as branching or error handling.

    ![Commander scripting](resources/an1259-v14-ncp-commander-scripting.png)

     You can export the commands sent in the Smart console with the **Export** control. This saves the sent commands to a file that can be imported back as a script. You can also export the raw script using the **Export** button under the editor.

    ![Commander export messages](resources/an1259-v14-ncp-commander-export-messages.png)

### Host Provisioner with Bluetooth NCP Commander

Bluetooth NCP Commander also supports Bluetooth mesh features. You can issue Bluetooth mesh commands manually in the command box of Smart Console or use the host provisioner feature from the left menu. You can use the feature to provision and configure mesh nodes and to manage mesh networks rather than using a Bluetooth Mesh mobile application.

To use the Bluetooth Mesh features, create, build, and flash the device with an NCP example supporting Mesh features. Otherwise, the provisioner initialization attempt returns SL_STATUS_NOT_SUPPORTED (0x000f).

![NCP example supporting Mesh features](resources/an1259-v14-mesh-example.png)

In **Settings**, if the **Reset Mesh Node before Initializing as Provisioner** option is enabled, the host provisioner does a factory reset (the **node_reset** command) on the NCP target device before initializing the node. Clicking **Clear Data** next to **Remove all locally saved mesh data** removes the network and application keys that were configured during initialization.

![mesh initialization](resources/an1259-v14-ncp-commander-mesh-initialization.png)

1. To start using the host provisioner, select either **Provision** or **Networks & Nodes** on the left menu, and click **Initialize as Provisioner**.

    ![Initialize as Provisioner](resources/an1259-v14-ncp-commander-mesh-start-provisioning.png)

2. To provision devices, select **Provision** on the left menu and click **Start Scan** in the right panel. The devices that are transmitting unprovisioned beacons are shown in the **Discovered Devices** section. If you do not have a network from a previous configuration or have reset the provisioner node, you must create a new network with **Create New Network**.

    ![mesh provision](resources/an1259-v14-ncp-commander-mesh-provision.png)

3. Enter the name of the new network and click **Confirm**.

    ![create new network](resources/an1259-v14-ncp-commander-mesh-create-network.png)

4. Click **Provision** next to the device you want to provision.

    ![provision](resources/an1259-v14-ncp-commander-mesh-provision-device.png)

5. Before configuring devices, you may need to create application keys and groups. Application keys, groups, and other network settings can be managed in the **Settings** tab of the **Networks & Nodes** menu item. To create an application key, click **Create App Key**, name the key, and click **Confirm**. You can create as many application keys as you need. If you have created any application keys before, you can click **Get App Keys** to retrieve them. To create a group, click **Add Group**, name the group, and click **Confirm**. You can create as many groups as you need.

    ![create keys groups](resources/an1259-v14-ncp-commander-mesh-create-keys-groups.png)

6. In the same tab, you can induce a full network-wide key refresh or exclude nodes.

    ![other settings](resources/an1259-v14-ncp-commander-mesh-other-settings.png)

7. To configure a provisioned device, select **Networks & Nodes** on the left menu. The devices you provisioned are shown in the **Nodes (Provisioned Devices)** section of the **Settings** tab. Click **Configure** and a **Mesh Node** tab opens in which you can configure the device.

    ![configure provisioned device](resources/an1259-v14-ncp-commander-mesh-configure-provisioned-device.png)

8. In the **Application Keys** section of the **Mesh Node** tab, select an application key from the drop-down list and then click **Add**.

    ![mesh device keys](resources/an1259-v14-ncp-commander-mesh-device-keys.png)

9. Click **Get DCD** to configure all the Models available on your node(s), bind to app keys, set publishing or subscription to groups, fine tune parameters, and so on.

    ![node configuration](resources/an1259-v14-ncp-commander-mesh-node-configuration.png)

10. To configure the Provisioner, the Models must first be initialized using **Initialize Client Models**.

    ![initialize models](resources/an1259-v14-ncp-commander-mesh-initialize-models.png)

11. The Application Key must be bound to the Models in order for them to decrypt received messages. Press **Bind**.

    ![initialize models 2](resources/an1259-v14-ncp-commander-mesh-initialize-models-2.png)

12. To subscribe the Models to the messages of the recently created Group (optional), select the chosen Group from the dropdown and click **Subscribe**.

    ![subscribe models](resources/an1259-v14-ncp-commander-mesh-subscribe-models.png)

13. When configuration is complete, click **Done**. The **Show Nodes** tab is displayed, where you can **Get DCD** of the provisioned Node(s).

    ![Get DCD](resources/an1259-v14-ncp-commander-mesh-get-dc.png)

14. After listing, you can **Get** or **Set** the Server states of the Node(s).

    ![get or set server states](resources/an1259-v14-ncp-commander-mesh-get-set-server-states.png)

15. Click Set to set the current state of the selected Server of the selected Node.

    ![set server state](resources/an1259-v14-ncp-commander-mesh-set-server-state.png)

16. Click **Get** to get the current state of the selected Server of the selected Node.

    ![get server state](resources/an1259-v14-ncp-commander-mesh-get-server-state.png)

17. On the **Show Groups** tab, you can **Set** or **Get** the Server Model(s) states on a Group level.

    ![group level](resources/an1259-v14-ncp-commander-mesh-get-set-model-state.png)

## Building the NCP Host Examples on Windows

The Silicon Labs v3.x Bluetooth SDK contains a generic NCP Host example project for the PC. This example can be compiled on Windows or any POSIX OS. This section goes through the build process on Windows.

>**Note**: The host example projects in the SDK use the dynamic GATT database feature. They are to be used with the **Bluetooth – NCP** target application.

1. To build the examples properly, the MSYS2 development toolchain must be installed on your PC. Download MSYS2 at [MSYS2](https://www.msys2.org/).

2. After MSYS2 is installed, update the package database as described at [MSYS2](https://www.msys2.org/).

3. Start MSYS2 bash and install mingw-64 with the following command:

    ```C
    pacman -S make mingw-w64-x86_64-gcc
    ```

4. Close MSYS2 and start MSYS2 MinGW 64-bit.

    ![msys2 mingw 64-bit](resources/an1259-msys2-mingw-64-bit.png)

5. Change to the NCP Host example folder, where \<version> varies by SDK version:

    ```C
    cd c:\SiliconLabs\SimplicityStudio\v5\developer\sdks\gecko_sdk_suite\v3.x\app\bluetooth\example_host\bt_host_empty\
    ```

    or

    ```C
    cd c:\Users\<username>\SimplicityStudio\SDKs\gecko_sdk\app
    ```

6. Create an export of the example with the command `make export`. After the project files are exported, the export directory will be a working directory that is completely detached from the SDK but has the same folder structure inside. The benefit of using an export is that changes in the (config) files during development will not affect the SDK content, and multiple instances can coexist, for example for testing different variants. You can also use `make export EXPORT_DIR=/my/custom/export/path` to export the example to a custom directory.

7. Within the export folder navigate to the */app/bluetooth/example_host/bt_host_empty* folder.

8. If you want to add any service/characteristic to the GATT database, edit the */config/btconf/gatt_configuration.btconf* file. Edit it either with a text editor or drag-and-drop the file onto Simplicity Studio to edit it with the GATT Configurator. Do not forget to save the file after editing.

9. Generate GATT database source files from the .*btconf* file by running `make gattdb` (in the */bt_host_empty* folder). Note: The generator script requires installing Python 3 and the Jinja2 package by calling `pip install jinja2`.

10. Build the exported project with the command: `make`. (Run it in the */bt_host_empty* folder, where you can find the makefile).

11. The build output is created in a new *exe* folder. Go to this folder with `cd exe`, and then run`bt_host_empty.exe`. The COM port and the IP address of the target are passed as command line parameters. The COM port should be the same as the one used by the JLink CDC UART Port, as shown in [NCP Host Development](./03-ncp-host-development). To see how to pass the different parameters, first run the exe with the `-h` (help) switch.

    ```C
    .\bt_host_empty.exe -h
    ```

12. Once the UART connection with the device is established, you should see the following:

    ![started advertising message](resources/an1259-figure-3-6.png)

13. Now you can connect to the device over Bluetooth.

## Using Python for Host Side Development

You can also implement a host application using Python. A Python package is available at [PyBGAPI](https://pypi.org/project/pybgapi/). This package parses the API description file of the Bluetooth SDK and makes it possible to issue BGAPI commands and get BGAPI events in the Python environment. See the referred website for further documentation.
