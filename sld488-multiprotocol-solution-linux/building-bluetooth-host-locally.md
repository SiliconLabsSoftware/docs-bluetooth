
# Building the Bluetooth Hosts Locally

This page describes how to configure and build the Bluetooth Host applications on a Raspberry pi 4B or a similar ARM host platform.

## Prerequisites

Clone the Silicon Labs SDK source code on the Raspberry Pi:

git clone <https://github.com/SiliconLabsSoftware/sisdk-release.git>

## Bluetooth Host Configuration for Multiprotocol NCP

The default Bluetooth NCP host application (bt_host_empty) is configured to communicate with the NCP target using BGAPI via UART. In order to enable CPC, the application needs to be built with the following command line option:

```bash
make CPC=1
```

The makefile for `bt_host_empty` is in the following SiSDK directory: <path_to_sdk>/bluetooth_le_app/example_host/bt_host_empty.

>**Note**: CPCd must first be installed on the system before building the host application.  

## Bluetooth Host Configuration for Multiprotocol RCP

### BlueZ - Official Linux Bluetooth Stack

In newer versions of Raspberry Pi OS, BlueZ is already installed by default and the bluetooth service is enabled.

- Check the BlueZ stack version using the command `bluetoothd --version` and if not the latest, download the latest version from <https://www.kernel.org/pub/linux/bluetooth/> and follow the instructions in the README file.

- The bluetooth service status can be checked with the command `systemctl status bluetooth`, which should give the following output:

```bash
● bluetooth.service - Bluetooth service
     Loaded: loaded (/lib/systemd/system/bluetooth.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-10-15 20:24:27 BST; 4 weeks 2 days ago
       Docs: man:bluetoothd(8)
   Main PID: 932 (bluetoothd)
     Status: "Running"
      Tasks: 1 (limit: 3933)
        CPU: 5.179s
     CGroup: /system.slice/bluetooth.service
             └─932 /usr/libexec/bluetooth/bluetoothd
```

If the bluetooth service is disabled, manually start the service by running `systemctl start bluetooth`.

*As of Nov 2024, the latest BlueZ version is 5.79.

### CPC HCI Bridge

The CPC-HCI bridge application connects to the CPCd and exposes a virtual serial device on the Linux host in `/dev/pts`. The source code and makefile are located in the SiSDK at `<path_to_sdk>/app/bluetooth/example/example_host/bt_host_cpc_hci_bridge`. Go to the `bt_host_cpc_hci_bridge` folder and run the following command to build the CPC-HCI bridge application:

```bash
make
```
