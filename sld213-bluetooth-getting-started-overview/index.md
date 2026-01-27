# Getting Started with Silicon Labs Bluetooth LE Development

To get started with Bluetooth LE development, download the Simplicity Studio Development environment
as described in the [Simplicity Studio 5 User's Guide](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-getting-started/). Simplicity Studio 5 includes everything needed for IoT product development with Silicon Labs devices including a resource and project launcher, software configuration tools, full IDE with GNU toolchain, and analysis tools.

Once you have downloaded Simplicity Studio, you will be prompted to install the Gecko SDK (GSDK), which contains the Bluetooth Software Development Kit (SDK). The GSDK combines Silicon Labs wireless SDKs and Gecko Platform into a single, integrated package. The Bluetooth SDK comes with a number of example application that you can then modify to create your own applications.

See [Prerequisites](#prerequisites) for additional details.

These pages focus on use and development in the Simplicity Studio 5 environment. Alternatively, Gecko SDK may be installed manually by downloading or cloning the latest from GitHub. See the [GitHub site](https://github.com/SiliconLabs/gecko_sdk) for more information.

Silicon Labs provides two different starter kits. The links below provide specific instructions for getting started with each kit.

- [**Wireless Starter Kit**](/bluetooth/{build-docspace-version}/bluetooth-getting-started-wstk): The Blue Gecko Bluetooth Wireless Starter Kit (WSTK) helps you evaluate Silicon Labs' Blue Gecko Bluetooth modules and get started with software development. The kits come in different versions with different module radio boards. See the [Silicon Labs product page](https://www.silabs.com/products/development-tools/wireless/bluetooth/bluegecko-bluetooth-low-energy-module-wireless-starter-kit) for details on current configurations. If you are developing with an EFR32BG device and have purchased a EFR32BG Wireless Starter Kit (WSTK), you can use precompiled demos and an Android or iOS smartphone app to demonstrate Bluetooth software features.
- [**BGM220 Explorer Kit (part number: BGM220-EK4314A)**](/bluetooth/{build-docspace-version}/bluetooth-getting-started-bgm): The BGM220 is focused on rapid prototyping and IoT concept creation around the Silicon Labs BGM220P module.

Once you have set up development with your platform of choice and optionally experimented with precompiled demos, you can begin to customize example applications for your specific needs. See the [**Getting Started with Application Development**](/bluetooth/{build-docspace-version}/bluetooth-getting-started-app-dev) section for details.

## Prerequisites

Before beginning application development, you should have:

- Acquired a basic understanding of Bluetooth technology and terminology. [Bluetooth LE Fundamentals](/bluetooth/{build-docspace-version}/bluetooth-le-fundamentals) provides a good starting point if you have not yet learned about Bluetooth.

- Purchased an EFR32BG Wireless Starter Kit or BGM220 Explorer Kit.
- Created an account at Silicon Labs. You can register at [https://siliconlabs.force.com/apex/SL_CommunitiesSelfReg?form=short](https://siliconlabs.force.com/apex/SL_CommunitiesSelfReg?form=short).
- Downloaded Simplicity Studio 5 and the Silicon Labs Gecko SDK containing the Bluetooth SDK and become generally familiar with the SSv5 Launcher perspective. SSv5 installation and getting started instructions along with a set of detailed references can be found in the online *Simplicity Studio 5 User’s Guide*, available on [https://docs.silabs.com/](https://docs.silabs.com/) and through the SSv5 help menu.
- Obtained a compatible compiler (See the Bluetooth SDK’s release notes for the compatible versions):

  - Simplicity Studio comes with a free GCC C-compiler.
  - IAR Embedded Workbench for ARM (IAR-EWARM) can also be used as the compiler for Silicon Labs Bluetooth projects. Once IAR-EWARM is installed, the next time Simplicity Studio starts it will automatically detect and configure the IDE to use IAR-EWARM.

To get a 30-day evaluation license for IAR-EWARM:

- Go to the Silicon Labs support portal at [https://www.silabs.com/support](https://www.silabs.com/support).

- Scroll down to the bottom of the page, and click **Contact Support**
- If you are not already signed in, sign in.
- Click the Software Releases tab. In the View list, select **Development Tools**. Click **Go**. In the results is a link to the IAR-EWARM version named in the release notes.
- Download the IAR package (takes approximately 1 hour).
- Install IAR.
- In the IAR License Wizard, click **Register with IAR Systems to get an evaluation license**.
- Complete the registration and IAR will provide a 30-day evaluation license.
- Once IAR-EWARM is installed, the next time Simplicity Studio starts it will automatically detect and configure the IDE to use IAR-EWARM.

## Gecko Platform

The Gecko Platform is a set of drivers and other lower layer features that interact directly with Silicon Labs chips and modules. Gecko Platform components include EMLIB, EMDRV, RAIL Library, NVM3, and mbed TLS. For more information about Gecko Platform, see release notes that can be found in Simplicity Studio’s Documentation tab.

## Documentation

Hardware-specific documentation may be accessed through links on the part Overview tab in Simplicity Studio 5.

![Overview with hardware doc](resources/sld213-image3.png?darkModeUrl=resources/sld213-image3.png)

## Support

You can access the Silicon Labs support portal at [https://www.silabs.com/support](https://www.silabs.com/support) through Simplicity Studio 5’s Welcome view under Learn and Support. Use the support portal to contact Customer Support for any questions you might have during the development process.

![Learn and Support tab](resources/sld213-image2.png?darkModeUrl=resources/sld213-image2.png)
