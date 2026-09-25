# Secure NCP

Secure NCP secures communication between the NCP Host and target by encrypting the commands, events, and any data transmitted between the target and the host.

## Target Side

To enable this feature on the target side, install the NCP Security Interface component.

![NCP Security Interface](resources/an1259-secure-ncp-target.png)

By default, the NCP target boots without using this encryption. It will be requested by the Host part, and after the security is increased, only encrypted messages are sent and accepted by the target.

## Host Side

1. Create a new **Bluetooth - Host Empty** project in Simplicity Studio 6

    ![studio6 host app generation](resources/an1259-studio6-host-app-generation.png)

2. At the *Target Device* select the option *Part* and the desired OS
    ![studio6 select os](resources/an1259-studio6-select-os.png)

3. Add `Secure NCP communication layer for host projects` component to the project
4. Secure mode requires the openssl package to be installed. It can be installed to the MSYS2 environment if necessary with:

```C
pacman -S mingw-w64-x86_64-openssl
```

5. Build the project in MSYS2 MinGW 64-bit
    ```C
    make -f bt_host_empty.Makefile
    ```
6. The build output is created in a new *build/debug/* folder. After the project is built, the encryption can be enabled by calling the .exe file with the command line parameter `-s`:

```C
.\bt_host_empty.exe -s
```

```C
$ ./build/debug/bt_host_empty.exe -u COM<*> -s
[D] Timer function intialized
[I] NCP host initialised.
[I] Press Crtl+C to quit

[I] Rebooting NCP target (0)...
[I] Start encryption using OpenSSL 3.0
[I] Communication encrypted
[I] Bluetooth stack booted: v11.0.1+0e13429e
[I] Bluetooth public device address: 04:87:27:E7:07:5D
[I] Started advertising.
```

Running the exe file without `-s` parameter will start a normal NCP Host application without encryption.
