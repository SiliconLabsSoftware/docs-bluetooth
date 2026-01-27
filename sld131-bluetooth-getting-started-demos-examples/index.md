# More Demos and Examples

Because starting application development from scratch is difficult, the Bluetooth SDK comes with a number of built-in demos and examples covering the most frequent use cases, as shown in the following figure. Demos are pre-built application images that you can run immediately. Software examples (example projects) can be modified before building the application image. See [Getting Started with Application Development](/bluetooth/{build-docspace-version}/bluetooth-getting-started-app-dev) for more information about configuring and customizing examples.

Demos with the same name as examples are built from their respective example. Click **View Project Documentation** to see additional information about some examples. This is also displayed on a **readme** tab when you create a project based on the example.

Use the **Demos** and **Example Projects** switches to filter on only examples or only demos. Demos are also noted by the blue Demo tag in the upper left of the card. The Solution Examples are primarily for use with multiprotocol applications.

![Filtering examples and demos](resources/sld131-image15.png?darkModeUrl=resources/sld131-image15.png)

>**Note**: The demos and examples you see are determined by the part selected. If you are using a custom solution with more than one part, click on the part you are working with to see only the items applicable to that part.

## Demo and Example Descriptions

The following examples are provided as part of the Bluetooth SDK. Examples with (\*) in their names have a matching pre-built demo.

### Silicon Labs Gecko Bootloader Examples

See *UG266: Silicon Labs Gecko Bootloader User’s Guide for GSDK 3.2 and Lower*, [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](/bluetooth/{build-docspace-version}/bootloader-user-guide-gsdk-4), or [Silicon Labs Gecko Bootloader User’s Guide for Series 3 and Higher](/bluetooth/{build-docspace-version}/bootloader-user-guide-series3-and-higher), and [Using the Gecko Bootloader with Silicon Labs Bluetooth Applications](/bluetooth/{build-docspace-version}/using-gecko-bootloader-with-bluetooth-apps).

### Bluetooth Examples

- **Bluetooth – RCP(\*)**: Radio Co-Processor (RCP) target application. Runs the Bluetooth Controller (i.e. the Link Layer only) and provides access to it using the standard HCI (Host-Controller Interface) over a UART connection.

- **Bluetooth – RCP CPC(\*)**: Radio Co-Processor (RCP) target application. Runs the Bluetooth Controller (i.e. the Link Layer only) and provides access to it using the standard HCI (Host-Controller Interface) over CPC (Co-Processor Communication) protocol through a UART connection.

- **Bluetooth – NCP(\*)**: Network Co-Processor (NCP) target application. Runs the Bluetooth stack dynamically and provides access to it via the Bluetooth API (BGAPI) using a UART connection. NCP mode makes it possible to run your application on a host controller or PC.

- **Bluetooth – NCP Host**: Reference implementation of an NCP (Network Co-Processor) host, which typically runs on a central MCU without radio. It can connect to an NCP target via UART to access the Bluetooth stack of the target and to control it using BGAPI.

- **Bluetooth AoA – NCP Locator(\*)**: Network Co-Processor (NCP) target application extended with CTE Receiver support. It enables Angle of Arrival (AoA) calculation. Use this application with Direction Finding host examples.

- **Bluetooth AoA –Asset Tag(\*)**: Demonstrates a CTE (Constant Tone Extension) transmitter that can be used as an asset tag in a Direction Finding setup estimating Angle of Arrival (AoA).

- **Bluetooth – SoC Application OTA DFU**: A minimal project structure that serves as a starting point for custom Bluetooth applications providing Over-the-Air device firmware update in the user application runtime.

- **Bluetooth – SoC Application OTA DFU FreeRTOS**: Demonstrates the integration of FreeRTOS into Bluetooth applications. RTOS is added to the *Bluetooth - SoC Application OTA DFU* sample application that realizes over-the-air device firmware updates in user application scope.

- **Bluetooth – SoC Application OTA DFU MicriumOS**: Demonstrates the integration of MicriumOS into Bluetooth applications. RTOS is added to the *Bluetooth - SoC Application OTA DFU* sample application that realizes over-the-air device firmware updates in user application scope.

- **Bluetooth – SoC Blinky(\*)**: The classic blinky example using Bluetooth communication. Demonstrates a simple two-way data exchange over GATT. This can be tested with the Simplicity Connect mobile app.

- **Bluetooth – SoC Certificate-Based Authentication and Pairing**: Demonstrates Certificate-Based Authentication and Pairing over Bluetooth LE.

- **Bluetooth – SoC CSR Generator**: Certificate-generating firmware example. Software generates the device EC key pair, the signing request for the device certificate, and other related data. The generated data can be read out by the Central Authority. See *Bluetooth – SoC Certificate Based Authentication and Pairing.*

- **Bluetooth – SoC DTM**: This example implements the direct test mode (DTM) application for radio testing. DTM commands can be called via UART. See [RF PHY Layer Evaluation in Bluetooth SDK v3.x and Higher](/bluetooth/{build-docspace-version}/bt-rf-phy-evaluation-using-dtm-sdk-v3x) for more information.

- **Bluetooth – SoC Empty**: A minimal project structure that serves as a starting point for custom Bluetooth applications. The application starts advertising after boot and restarts advertising after a connection is closed.

- **Bluetooth – SoC Interoperability Test(\*)**: A test procedure containing several test cases for Bluetooth Low Energy communication. This sample app (also provided as a demo) is meant to be used with the Simplicity Connect mobile app, through the "Interoperability Test" tile on the Develop view of the app.

- **Bluetooth – SoC Thermometer(\*)**: Implements a GATT Server with the Health Thermometer Profile, which enables a Client device to connect and get temperature data. Temperature is read from the Si7021 digital relative humidity and temperature sensor of the WSTK or of the Thunderboard.

- **Bluetooth – SoC Thermometer Client**: Implements a GATT Client that discovers and connects with up to four Bluetooth LE devices advertising themselves as Thermometer Servers. It displays the discovery process and the temperature values received via UART.

   >**Note**: Some radio boards will exhibit random pixels in the display when this example is running because they have a shared pin for sensor- and display-enabled signals.

- **Bluetooth – SoC Thermometer FreeRTOS**: Demonstrates the integration of FreeRTOS into Bluetooth applications. RTOS is added to the Bluetooth - SoC Thermometer sample app.

- **Bluetooth – SoC Thermometer Micrium OS**: Demonstrates the integration of Micrium RTOS into Bluetooth applications. RTOS is added to the Bluetooth - SoC Thermometer sample app.

- **Bluetooth – SoC Throughput(\*)**: Tests the throughput capabilities of the device and can be used to measure throughput between two EFR32 devices, as well as between a device and a smartphone using the Simplicity Connect mobile app, through the Throughput demo tile.

- **Bluetooth – SoC Voice(\*)**: Voice over Bluetooth Low Energy sample application. It is supported by Thunderboard Sense 2 and Thunderboard EFR32BG22 boards and demonstrates how to send voice data over GATT, which is acquired from the on-board microphones.

- **Bluetooth – SoC iBeacon(\*)**: Sends non-connectable advertisements in iBeacon format. The iBeacon Service gives Bluetooth accessories a simple and convenient way to send iBeacons to smartphones. This example can be tested together with the Simplicity Connect mobile app.

- **Bluetooth – SoC Thunderboard Sense 2(\*),** and **Thunderboard EFR32BG22(\*)**: Demonstrate the features of the Thunderboard Kit. These can be tested with the Simplicity Connect mobile app.

## Dynamic Multiprotocol Examples

See [Dynamic Multiprotocol Development with Bluetooth and Proprietary Protocols on RAIL](/bluetooth/{build-docspace-version}/multiprotocol-dynamic-ble-proprietary-on-rail) for more information.

- **Bluetooth RAIL DMP – SoC Empty FreeRTOS**: A minimal project structure, used as a starting point for custom Bluetooth + Proprietary DMP (Dynamic Multiprotocol) applications. It runs on top of FreeRTOS and multiprotocol RAIL.

- **Bluetooth RAIL DMP – SoC Empty Micrium OS**: A minimal project structure, used as a starting point for custom Bluetooth + Proprietary DMP (Dynamic Multiprotocol) applications. It runs on top of Micrium OS and multiprotocol RAIL.

- **Bluetooth RAIL DMP – SoC Empty Standard FreeRTOS**: A minimal project structure, used as a starting point for custom Bluetooth + Standard DMP (Dynamic Multiprotocol) applications. It runs on top of FreeRTOS and multiprotocol RAIL utilizing IEE802.15.4 standard protocol.

- **Bluetooth RAIL DMP – SoC Empty Standard Micrium OS**: A minimal project structure, used as a starting point for custom Bluetooth + Standard DMP (Dynamic Multiprotocol) applications. It runs on top of Micrium OS and multiprotocol RAIL, utilizing IEE802.15.4 standard protocol.

- **Bluetooth RAIL DMP – SoC Light RAIL FreeRTOS(\*)**: A Dynamic Multiprotocol reference application demonstrating a light bulb that can be switched both via Bluetooth and via a Proprietary protocol. Can be tested with the Simplicity Connect mobile app and the **RAIL – SoC Switch** sample app.

- **Bluetooth RAIL DMP – SoC Light RAIL Micrium OS**: A Dynamic Multiprotocol reference application demonstrating a light bulb that can be switched both via Bluetooth and via a Proprietary protocol. Can be tested with the Simplicity Connect mobile app and the  **RAIL – SoC Switch** sample app.

- **Bluetooth RAIL DMP – SoC Light Standard FreeRTOS(\*)**: A Dynamic Multiprotocol reference application demonstrating a light bulb that can be switched both via Bluetooth and via a standard protocol. Can be tested with the Simplicity Connect mobile app and the **RAIL – SoC Switch** Standards sample app.

- **Bluetooth RAIL DMP – SoC Light Standard Micrium OS(\*)**: A Dynamic Multiprotocol reference application demonstrating a light bulb that can be switched both via Bluetooth and via a standard protocol. Can be tested with the Simplicity Connect mobile app and the **RAIL – SoC Switch Standards** sample app.

### NCP Host Examples

NCP host examples are located in \<GSDK-install-location>\app\bluetooth\example_host.

- **bt\_host\_empty:** Minimal host-side project structure, used as a starting point for NCP host applications. Use it with the **Bluetooth – NCP** target application flashed to the radio board.

- **bt\_host\_ota\_dfu:** Demonstrates how to perform an OTA DFU on a Silicon Labs Bluetooth Device. It requires a WSTK with a radio board flashed with NCP firmware to be used as the GATT client that performs the OTA.

- **bt\_host\_uart\_dfu:** Demonstrates how to perform a UART DFU on a Silicon Labs Bluetooth Device running NCP firmware.

- **bt\_host\_voice:** On a WSTK programmed with NCP firmware, it to connects to the **Bluetooth – SoC Voice** example, sets the correct configuration on it, receives audio via Bluetooth, and stores audio data into a file.

- **bt\_aoa\_host\_locator:** A locator host sample app that works together with a **Bluetooth AoA – NCP Locator** target app. It receives IQ samples from the target and estimates the Angle of Arrival (AoA). For more information see [Application Development with Silicon Labs’ RTL Library](https://docs.silabs.com/rtl-lib/latest/direction-finding-solution-guide/).

- **bt\_host\_positioning:** Connects to multiple **bt\_aoa\_host\_locator** sample apps (via MQTT) and estimates a position from Angles of Arrival (AoA). For more information, see *QS175: Application Development with Silicon Labs’ RTL Library.*

- **bt\_host\_positioning\_gui:** Connects to the **bt\_host\_positioning** sample app (via MQTT), reads out the position estimations and displays the tags and locators on a 3D GUI. This sample app is python based. For more information, see [Application Development with Silicon Labs’ RTL Library](https://docs.silabs.com/rtl-lib/latest/direction-finding-solution-guide/).

- **bt\_host\_throughput:** Tests the throughput capabilities of the device in NCP mode and can be used to measure throughput between two devices as well as between a device and a smartphone.

- **bt\_host\_cpc\_hci\_bridge:** A background application to be run when HCI interface is exposed via CPC. This application retrieves the HCI commands/events from the CPC messages and forwards them toward the Bluetooth host running on the PC. Similarly, it forwards the HCI commands from the host toward the target over CPC.

## Code Examples

Additional examples are provided in repositories on GitHub. See [Code Examples](/bluetooth/{build-docspace-version}/bluetooth-examples-overview) for more information.
