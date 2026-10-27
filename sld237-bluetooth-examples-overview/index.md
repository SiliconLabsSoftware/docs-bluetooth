# Code Examples

## SDK Examples

SDK examples are not only available through Simplicity Studio but also in the [GSDK GitHub repo](https://github.com/SiliconLabs/gecko_sdk). Both embedded application examples and host examples are available. Select the version of interest and open 'app/bluetooth/example' or 'app/bluetooth/example_host'. SDK examples are described in [Getting Started - More Demos and Examples](/bluetooth/{build-docspace-version}/bluetooth-getting-started-demos-examples)

## GitHub Examples

The Bluetooth SDK provides examples that demonstrate basic Bluetooth features (such as a simple Bluetooth server, Bluetooth client, OTA DFU) and serve as a starting point for your (RCP, NCP, SoC, RTOS or DMP based) development. Often, however, developers need guidance on how to use additional Bluetooth features (such as extended advertisements, and so on). Also, it may be very useful to see full solution examples that cover a common use case and demonstrate how to integrate peripheral programming with Bluetooth.

To satisfy these needs, additional examples are provided in repositories on GitHub. The repositories have detailed readme files that instruct the developer on how to set up the project. To make the setup even easier, some repositories also provide .slcp files that make it possible to automatically generate the project with Simplicity Studio. The examples in these repos are not maintained to the same degree as are the examples provided with the SDK. If you find an issue, report it on [https://community.silabs.com/](https://community.silabs.com/).

### Application Examples

This repository provides full application examples, which demonstrate how to add Bluetooth communication to an application that provides solution for a real problem. It is worth checking this repository both to learn how a full Bluetooth application should look like, and also because you might find an example that is already very similar to the one you want to implement. The repository is available here: [https://github.com/SiliconLabs/bluetooth_applications](https://github.com/SiliconLabs/bluetooth_applications).

### Stack Feature Examples

This repository demonstrates the different features of the Bluetooth stack. Each example focuses on one feature, and only a minimal application logic is added to demonstrate it. This repository provides .slcp files. The repository is available here: [https://github.com/SiliconLabs/bluetooth_stack_features](https://github.com/SiliconLabs/bluetooth_stack_features).

### Python-Based Host Examples

Python-based NCP host examples can be accessed at [https://github.com/SiliconLabs/pybgapi-examples](https://github.com/SiliconLabs/pybgapi-examples). These examples are meant to be used with PyBGAPI ([https://pypi.org/project/pybgapi/](https://pypi.org/project/pybgapi/)), which enables implementing NCP host codes using Python.

### Generating GitHub Example Projects with Simplicity Studio

To generate GitHub example projects with Simplicity Studio (only applicable to repositories with slcp files):

1. Open Simplicity Studio and navigate to Window \> Preferences \> Simplicity Studio \> External Repos.
2. Click **Add**.
3. In the URI field enter the repository’s HTTPS clone address, for example [https://github.com/SiliconLabs/bluetooth_stack_features.git](https://github.com/SiliconLabs/bluetooth_stack_features.git)

    Alternatively, clone the repository onto your hard drive, and provide the path to the .git folder in your local repository, for example C:\MyRepositories\bluetooth_stack_features\.git.

4. Enter an arbitrary name (such as ‘Bluetooth Stack Features’) and description for the repository, which will be displayed later.
5. Click **Next**. If you have entered the repository’s clone address, Simplicity Studio will clone the repository for you.
6. Click **Finish,** then click **Apply and Close.**
7. If you are not on the Launcher perspective, open it from the Perspectives toolbar in the upper-right corner.
8. Select your board in the Debug Adapters view or in the My Products view.
9. On the General card, verify the Gecko SDK version and change if necessary.

    Note: Bluetooth stack feature examples are currently compatible with Gecko SDK v4.1 only!

10. Go to the Example Projects & Demos tab.
11. Now you should see your repository listed under the Provider filtering class. Select this filter.

    Note: The repository only shows up if it contains at least one example that is compatible with your device.

12. All the examples contained in the repository that are compatible with your device are displayed. Click **Create** on any of them to create a new example project. The example project installs all the software components necessary to demonstrate the given feature, and all the needed code is automatically copied into your project. Additional configuration might be needed, so read the readme file of the example carefully.

    ![Creating an example](resources/sld237-image16.png?darkModeUrl=resources/sld237-image16.png)
