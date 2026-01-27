# Running Host Applications with Pre-Built Packages

## Running Host Applications with Debian-Bookworm Packages

As mentioned before, you can install pre-built host applications. The packages can be acquired from the Silicon Labs Github Page: https://github.com/SiliconLabsSoftware/sisdk-release/releases in the debian-bookworm.zip folder.

Install all packages, making sure to match the right version to your system architecture.

First, install the cpc packages.

```bash
apt-get install -y ./debian-bookworm/deb/libcpc3_<version>.deb
apt-get install -y ./debian-bookworm/deb/libcpc-dev_<version>.deb
apt-get install -y ./debian-bookworm/deb/cpcd_<version>.deb
```

Then, install the ot-br-posix and zigbeed packages.

> **Note**: When installing the ot-br-posix .deb file on 32-bit bookworm systems such as armhf (arm32v7), make sure the dhcpcd package on your target system is higher than the recommended bookworm version of 9.4.1-24, which has a [known startup issue](https://github.com/NetworkConfiguration/dhcpcd/issues/323). The bookworm-backports apt source provides version 10.1.0 which is highly recommended. If not, Silicon Labs recommends updating your dhcpcd package version to at least 9.5.1. To use the backports source, you can create the following two files and then run `sudo apt update`.

```bash
/etc/apt/sources.list.d/bookworm-backports.list:

deb [trusted=yes] http://deb.debian.org/debian bookworm-backports main

/etc/apt/preferences.d/bookworm-backports:

Package: dhcpcd
Pin: release n=bookworm-backports
Pin-Priority: 1001

Package: dhcpcd-base
Pin: release n=bookworm-backports
Pin-Priority: 1001
```

```bash
apt-get install -y ./debian-bookworm/deb/ot-br-posix_<version>.deb
apt-get install -y ./debian-bookworm/deb/zigbeed_<version>.deb
```

## Running Host Applications with OpenWRT Packages

Silicon Labs provides evaluation multiprotocol host application packages for OpenWRT. These packages are pre-built for Raspberry Pi 4 – bcm2711 devices running OpenWRT version 23.05.3.

### Installation

To use these packages, download the release tar file containing the ipk packages from https://github.com/SiliconLabsSoftware/sisdk-release/releases.

The openwrt.tar.gz should contain:

- openthread-br_\<version\>.ipk
- cpcd_\<version\>.ipk
- libcpc_\<version\>.ipk
- zigbeed_\<version\>.ipk

Install the packages with `opkg install silabs/\*.ipk`.

To register a uart device with a `/dev/ttyACM` device filepath, it is recommended to download the kmod-usb-acm package.

`opkg update`

`opkg install kmod-usb-acm`

As described in [Running Local Processes Concurrently: Running Zigbeed](running-local-processes-concurrently.md#running-zigbeed), the socat utility is needed for zigbeed. Make sure it is also installed on the system.

`opkg install socat`

#### Install CPCD

```
opkg update
opkg install libcpc_<version>.ipk
opkg install cpcd_<version>.ipk
```

#### Install OTBR

```
opkg update
opkg install openthread-br_<version>.ipk
```

#### Install Zigbeed

```
opkg update
opkg install zigbeed_<version>.ipk
```

## Build Bluetooth and Zigbee Host Applications

Both the Bluetooth host and Z3Gateway host applications can be built for OpenWRT. Instructions to build applications for OpenWRT can be found on the OpenWRT.org website.

The `cpcd.conf` and `zigbeed.conf` files (described in section [Configuring CPCD](building-cpcd-locally.md#cpcd-configuration)) are both populated during installation.

There are two setup instructions recommended for the `openthread-br` package, updating the firewall options, and updating the radio-url in the `otbr-agent` configuration file.

OpenWRT has a firewall that is independent of the Silicon Labs multiprotocol host apps. However, the firewall can interfere with the host apps as far as the generic processing of IP traffic is concerned, such as with the openthread border router application. One simple way to ensure that all IP traffic can be processed is by setting the policy for each firewall chain to ‘ACCEPT’; other alternatives can be configured on a case by case basis using the OpenWRT Firewall configuration user guide: https://openwrt.org/docs/guide-user/start#firewall_configuration.

The firewall service can be updated through the OpenWRT Unified Configuration Interface (UCI) with the following commands:

`uci set firewall.@defaults[0].input=’ACCEPT’`

`uci set firewall.@defaults[0].output=’ACCEPT’`

`uci set firewall.@defaults[0].forward=’ACCEPT’`

`uci commit`

`service firewall restart`

The other recommended setup step needed for the openthread-br package is to update the radio-url in the otbr-agent configuration file, /etc/init.d/otbr-agent with the radio-url specified in [OpenThread Host Applications](building-otbr-locally.md#configuration-of-the-otbr) in order to use the multiprotocol supported spinel interface using cpc-daemon.

## Startup

To start up the installed and configured multiprotocol host applications, perform the following sequence:

1. Run the cpc-daemon with `/usr/bin/cpcd`.

   1. Bind to the RCP if not previously done with `cpcd –bind ecdh`.

   2. Launch the `cpcd` program in the background or in a separate terminal with `cpcd&`.

2. For `openthread-br`:

   1. Launch the otbr-firewall service with `service otbr-firewall start`.

   2. Launch the otbr-agent service with `service otbr-agent start`.

   3. Launch the `ot-ctl` program to connect to the OpenThread cli through the agent socket.

3. For Zigbee Host applications:

   1. Run zigbeed with `/usr/bin/zigbeed`.

   2. Run Z3Gateway as in any other native configuration ie. `{Z3Gateway_build_path}/zigbee_z3_gateway -p ttyZigbeeNCP`.
