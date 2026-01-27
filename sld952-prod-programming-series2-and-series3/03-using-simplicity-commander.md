# Using Simplicity Commander

1. This application note uses Simplicity Commander v1.19.2. The procedures and console output may be different for the other versions of Simplicity Commander. The latest version of Simplicity Commander can be downloaded from [https://www.silabs.com/developers/mcu-programming-options](https://www.silabs.com/developers/mcu-programming-options).

    ```sh
    commander --version
    ```

    ```sh
    Simplicity Commander 1v19p2b1907

    JLink DLL version: 8.44

    Qt 5.15.2 Copyright (C) 2017 The Qt Company Ltd.

    EMDLL Version: 0v19p19b793

    mbed TLS version: 2.16.6

    Emulator found with SN=440329507 USBAddr=0

    DONE
    ```

2. The Simplicity Commander's Command Line Interface (CLI) is invoked by `commander.exe` in the Simplicity Commander folder. The location for Simplicity Studio 5 in Windows is `C:\SiliconLabs\SimplicityStudio\v5\developer\adapter_packs\commander`. For ease of use, it is highly recommended to add the path of `commander.exe` to the system PATH in Windows.

3. If more than one Wireless Starter Kit (WSTK) is connected via USB, the target WSTK must be specified using the `--serialno <J-Link serial number>` option.

4. If the WSTK is in debug mode OUT, the target device must be specified using the `--device <device name>` option. For more information about Simplicity Commander, see [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).
