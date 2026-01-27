# Building OTBR Locally

The purpose of this document is to provide comprehensive steps and guidance on building an OpenThread Border Router (OTBR) that communicates with the Co-Processor Communication Daemon (CPCd). This guide aims to assist developers and engineers in setting up and configuring an OTBR to enhance network communication and management. By following the outlined procedures, you can use the OTBR in a multiprotocol context. For reference on hardware or software requirements, refer to the local host configuration and co-processor configuration.

## Install OTBR Repository

OpenThread host applications can be built from Silicon Lab's SDK as well as GitHub repositories. Silicon Labs provides a vendor extension to build an OpenThread Linux host application with multi-PAN and CPC support.

Start by cloning the SDK source code on your Raspberry Pi:

```shell
git clone https://github.com/SiliconLabsSoftware/sisdk-release.git

```

```shell
sudo apt-get install bind9
```

For steps on setting up and running CPCd, refer to [Building CPCd locally](building-cpcd-locally.md).

## Building OTBR

Before proceeding to build the OTBR, first, ensure:

The OpenThread repo is in ~/simplicity_sdk/openthread_stack/util/third_party/openthread. This folder must be symlinked under ~/simplicity_sdk/openthread_stack/util/third_party/ot-br-posix/third_party/openthread/repo:

```shell
ln -s ~/simplicity_sdk/openthread_stack/util/third_party/openthread/ ~/simplicity_sdk/openthread_stack/util/third_party/ot-br-posix/third_party/openthread/repo
```

> **Note**: The specific flags shown in the following commands are recommended for 1.4 certifiable OTBR, but may require review for your use case.

Now, install the dependencies by running the bootstrap script:

```shell
cd ~/simplicity_sdk/openthread_stack/util/third_party/ot-br-posix/
sudo ./script/bootstrap
```

To configure the OTBR for multi-PAN and CPC support, you can use Silicon Labs specific configuration settings for border-router and ot-cli. Use the special configuration header hosted in the GSDK/SiSDK under `protocol/openthread/platform-abstraction/posix/openthread-core-silabs-posix-config.h`.

```shell
sudo cp ~/simplicity_sdk/openthread/platform-abstraction/posix/openthread-core-silabs-posix-config.h ~/simplicity_sdk/openthread_stack/util/third_party/openthread/src/posix/platform/
```

To build a Thread certifiable otbr-agent, use the following reference commands, which should be run from under the `util/third_party/otbr-posix` directory. Make sure to provide the absolute path to `$SDK_DIR` and `$CPCD_DIR` variables, for example:

```shell
export SDK_DIR=<An absolute path to SDK directory> 
export CPCD_DIR=<An absolute path to CPCD directory> 
```

Run the setup script to complete building OTBR:

```shell
sudo INFRA_IF_NAME=eth0 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.4 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DHCP6_PD=ON" ./script/setup
```

Some of the important Build Flags are the following:

- OT_POSIX_CONFIG_RCP_VENDOR_INTERFACE: This needs to point to vendor interface source files. For Multiprotocol solutions this is the cpc_interface.cpp file.

- OT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE: This points to the vendor interface CMake package file. For Multiprotocol solutions this is the posix_vendor_rcp.cmake file. (Note in earlier releases this was referred to as SilabsRcpDeps).

- DOT_MULTIPAN_RCP=ON: Compiles the multi-PAN RCP Code.

- OT_POSIX_RCP_VENDOR_BUS=ON

- DOT_PLATFORM_CONFIG: This should point to our openthread-core-silabs-posix-config.h file.

- DOT_THREAD_VERSION: This is the Thread Version. This can be modified for testing Thread 1.4.

OTBR supports a reference implementation of DHCPv6 Prefix Delegation client using the flag DHCPV6_PD_REF=1 which can be passed to the bootstrap and setup commands above. This utilizes PD implementation using external utilities like dhcpcd and radvd. Border router products may have their own implementation of this feature they might choose to use.

To link against the cpc library and header, it is recommended that cpcd is first built and installed. The openthread CMake projects will automatically resolve these dependencies using pkgconfig or by searching in standard installation directories. If cpcd is not installed, the library path and include directory can be specified as CMake arguments (-DCpc_LIBRARY and -DCpc_INCLUDE_DIR respectively), or the root path to the cpc-daemon project can be specified as a CMake argument (-DCPCD_SOURCE_DIR).

**Note**:

- Host applications can be built using 'CMake' only; 'make' is unsupported.

- If you are looking to clear or update your build space, ensure to do the following command `sudo rm -rf build/`.

### OTBR Configuration Changes for Child and Source Match Tables

Starting with the 2025.12.0 release, the sizing logic for child and source match tables has been **decoupled**. The number of entries in the source match table now depends on a combination of:

- The configured **maximum number of children** (`OPENTHREAD_CONFIG_MLE_MAX_CHILDREN`)

- The **build type** (single-instance, multi-instance, or CMP)

- The **number of OpenThread network instances or protocol stacks** included in the build (for example, OpenThread and Zigbee in a CMP setup)

The total number of source-match entries is sized to support all child devices across the enabled protocol stacks. In practice, this corresponds to the number of OpenThread children across all OT instances, plus any Zigbee end-device children when CMP is enabled:

``` shell
total entries ≈ (MLE children × OT instances) + (Zigbee children, only when CMP is enabled)
```

This can be adjusted at build time using:

```shell
-DOPENTHREAD_CONFIG_MLE_MAX_CHILDREN=10

-DSL_ZIGBEE_MAX_END_DEVICE_CHILDREN=6

-DOPENTHREAD_CONFIG_MULTIPLE_INSTANCE_ENABLE=1
-DOPENTHREAD_CONFIG_MULTIPLE_INSTANCE_NUM=2

```

**Note**: Use SL_ZIGBEE_MAX_END_DEVICE_CHILDREN only if Zigbee runs alongside OT (CMP).

**Note**: Use OPENTHREAD_CONFIG_MULTIPLE_INSTANCE_* only if you’re intentionally running multiple OT instances in the same host.

## Configuration of the OTBR

Once the OTBR has been successfully built, you need to modify the Radio URL to point to the correct CPCd iid. Note these instructions use: -DOT_MULTIPAN_RCP=ON meaning the OTBR was built for a multi-PAN RCP, this requires us to include both iid and iid-list values. Both Zigbeed and the OpenThread stack can connect to CPCd and use the multi-PAN RCP at one time. Spinel messages for each application are labeled with a Spinel Interface ID (IID), which is supplied to the application at startup via the OpenThread Radio URL command line argument. The fact that the RCP is being shared between multiple PANs is transparent to the host applications.

-Multi-PAN RCP (ex: Zigbee RCP + OT RCP):  `OTBR_AGENT_OPTS="-I wpan0 -B eth0 spinel+cpc://cpcd_0?iid=2&iid-list=0"`. This indicates the OT RCP interface will respond to IID=2.

- Zigbee NCP + OT RCP:  `OTBR_AGENT_OPTS="-I wpan0 -B eth0 spinel+cpc://cpcd_0?iid=0&iid-list=0"`. The Zigbee NCP does not use Spinel thus the multi-PAN interface (IID) can be set to 0. 

 `/etc/default/otbr-agent` is the configuration file where you can update the Radio URL as follows:

```shell
sudo vi /etc/default/otbr-agent 
OTBR_AGENT_OPTS="-I wpan0 -B eth0 spinel+cpc://cpcd_0?iid=2&iid-list=0"
```

To confirm that the OTBR is correctly running, you can access its CLI utility by executing `sudo ot-ctl`.

## OTBR Build Commands for SDK Versions

The following table contains the reference OTBR build commands for a few previous releases.

Common steps for following SDK releases:

### SiSDK 2025.12 - Thread 1.4

```shell
sudo INFRA_IF_NAME=eth0 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.4 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DHCP6_PD=ON" ./script/setup
```

### SiSDK 2025.06 - Thread 1.4

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=0 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NAT64=1 DHCPV6_PD=0 WEB_GUI=0 REST_API=0 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.4 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON -DOTBR_TREL=ON -DOTBR_DHCP6_PD=ON" ./script/setup
```

### SiSDK 2024.12 - Thread 1.4

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=0 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NETWORK_MANAGER=0 NAT64=1 DNS64=1 DHCPV6_PD=0 WEB_GUI=0 REST_API=0 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.4 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR  -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON -DOTBR_TREL=ON -DOTBR_DHCP6_PD=ON" ./script/setup
```

### SiSDK 2024.06 - Thread 1.3

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=0 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NETWORK_MANAGER=0 NAT64=1 DNS64=1 DHCPV6_PD=0 WEB_GUI=0 REST_API=0 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.3 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON -DOTBR_TREL=ON -DOTBR_DHCP6_PD=ON" ./script/setup
```

### GSDK v4.4.3

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=1 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NAT64=1 DNS64=1 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.3 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PROJECT_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON" ./script/setup
```

### GSDK v4.4.2

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=1 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NAT64=1 DNS64=1 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.3 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_CONFIG_RCP_BUS=VENDOR -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON" ./script/setup
```

### GSDK v4.4.1, v4.4.0

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=1 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NAT64=1 DNS64=1 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.3 -DOT_MULTIPAN_RCP=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_RCP_VENDOR_BUS=ON -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON" ./script/setup
```

### GSDK v4.3.3, v4.3.2

```shell
sudo INFRA_IF_NAME=eth0 RELEASE=1 REFERENCE_DEVICE=1 BACKBONE_ROUTER=1 BORDER_ROUTING=1 NAT64=1 DNS64=1 OTBR_OPTIONS="-DOT_THREAD_VERSION=1.3 -DOT_MULTIPAN_RCP=ON -DCMAKE_MODULE_PATH=$GSDK_DIR/protocol/openthread/platform-abstraction/posix -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_CONFIG_RCP_BUS=VENDOR -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=SilabsRcpDeps -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_CONFIG=openthread-core-silabs-posix-config.h -DOTBR_DUA_ROUTING=ON -DOTBR_DNSSD_DISCOVERY_PROXY=ON -DOTBR_SRP_ADVERTISING_PROXY=ON" ./script/setup 
```

## Building OT-CLI

This command will build the posix ot-cli app using CMake (which is a different OT Host application than the OTBR). The command to build the posix ot-cli from the `util/third_party/openthread` directory of the SDK using CMake:

```shell
sudo ./script/cmake-build posix -DOT_MULTIPAN_RCP=ON -DOT_POSIX_RCP_HDLC_BUS=ON -DOT_POSIX_RCP_SPI_BUS=ON -DOT_POSIX_RCP_VENDOR_BUS=ON -DCPCD_SOURCE_DIR=$CPCD_DIR -DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE=$GSDK_DIR/protocol/openthread/platform-abstraction/posix/cpc_interface.cpp -DOT_PLATFORM_CONFIG=openthread-core-silabs-posix-config.h -DOT_POSIX_CONFIG_RCP_VENDOR_DEPS_PACKAGE=$SDK_DIR/protocol/openthread/platform-abstraction/posix/posix_vendor_rcp.cmake
```

Build ‘otbr-agent’ and ‘ot-cli’ host applications using Silicon Labs SDK artifacts provides OT_PLATFORM_CONFIG, CPCD_SOURCE_DIR, and DOT_POSIX_CONFIG_RCP_VENDOR_INTERFACE configurations.
