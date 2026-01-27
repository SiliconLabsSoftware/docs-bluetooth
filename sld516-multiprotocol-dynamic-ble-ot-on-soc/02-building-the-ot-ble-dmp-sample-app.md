# Building the ot-ble-dmp Sample App

Precompiled demo application images are provided with the Gecko SDK Suite 3.0, compatible with:

- brd4180a

- brd4186c

- brd2703a

- brd4116a

To get started quickly, in the Simplicity Studio 5 (SSv5) Launcher Perspective, go to the DEMOS tab. Find the **ot-ble-dmp** demo and click **RUN**. This uploads the application image to your board.

To build the **ot-ble-dmp** sample app from source, you will need SSv5 and the Gecko SDK 3.x development environment with the following packages installed:

- OpenThread SDK

- Bluetooth SDK

The GNU ARM toolchain is installed with SSv5. The IAR-EWARM toolchain is not compatible with OpenThread.

1. With your target development hardware connected, open SSv5’s File menu and select **New \> Silicon Labs Project Wizard**. The Target, SDK, and Toolchain Selection dialog opens. Your target hardware should be populated. Click **NEXT**.

2. The Example Project Selection dialog opens. Use the Technology Type and Keyword filters to search for a specific example, in this case **ot-ble-dmp**. Select it and click **NEXT**.

   Note that if you do not see the application, your connected hardware may not be compatible. To verify, in the Launcher Perspectives My Products view enter **EFR32MGxx** and select one of the boards. Go to the Examples tab, filter by Thread technology, and verify you can see the app.

3. The Project Configuration dialog opens. Here you can rename your project, change the default project file location, and determine if you will link to or copy project files. Note that if you change any linked resource, it is changed for any other project that references it. Unless you know you want to modify SDK resources, use the default selection. Click **FINISH**.

The Simplicity IDE opens with the **ot-ble-dmp** project open in the Project Configurator. You may now build the project. For those used to Simplicity Studio 4, no generation step is necessary because it is done automatically. The ot-ble-dmp.s37 image will be located in the GNU ARM directory and may be uploaded to your board using an SSv5 tool such as the flash programmer or Simplicity Commander.
