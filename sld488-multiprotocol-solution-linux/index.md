# Running Zigbee, OpenThread, and Bluetooth Concurrently on a Linux Host with a Multiprotocol Co-Processor

> **Note: This section replaces AN1333: Running Zigbee, OpenThread, and Bluetooth Concurrently on a Linux Host with a Multiprotocol Co-Processor for SiSDK 2024.12.0 and up. Further updates to this application note will be provided here**.

This section describes how to run any combination of Zigbee EmberZNet, OpenThread, and Bluetooth networking stacks on a Linux host processor, interfacing with a single EFR32 Radio Co-processor (RCP) or Network Co-Processor (NCP) with multiprotocol and multi-PAN support. The intended use case is for gateway products that wish to run any combination of the three protocols on the Linux host processor with a single, shared EFR32 RCP/NCP. Each stack can use the co-processor to communicate simultaneously and independently. Key points covered include:

- **System Architecture**: Describes the components of the Multiprotocol and Multi-PAN RCP solution, including the RCP and NCP image, Coprocessor Communication Daemon (CPCd), Zigbeed, OpenThread and Bluetooth host applications. It explains how these components interact to enable concurrent operation of Zigbee, OpenThread, and Bluetooth on a single EFR32. Additionally, this section introduces various combinations of these three protocols, showcasing the flexibility and versatility of these multiprotocol solutions.

- **Co-Processor Configuration**: Instructions to set up the RCP and NCP using Simplicity Studio, such as building and flashing the RCP and NCP image for different boards.

- **Host Setup**: Guides you through setting up the required host code on a Raspberry Pi 4B with Raspbian OS. Although, these solutions can also run on similar ARM host platform only instructions for Raspberry Pi are provided. The document includes steps for installing dependencies, configuring CPCd, and starting the necessary services for Zigbee, OpenThread, and Bluetooth.

- **Running and Building Host Applications Locally**: Detailed instructions for running Zigbee, OpenThread and Bluetooth host applications. Explains how to build and run CPCd, Zigbeed, Z3Gateway, OpenThread Border Router, and a Bluetooth host.

- **Running Host Applications - Prebuilt Platforms**: Provides evaluation multiprotocol host application packages for OpenWRT, Debian, and Docker Multiprotocol Solution. These OpenWRT packages are pre-built for Raspberry Pi 4 running OpenWRT version 23.05.3.

- **Configuration Files**: Includes information on configuring CPCd and Zigbeed, how to modify UART or SPI configurations and enabling logging. It also covers the configuration of the OpenThread Border Router (OTBR) and detailed information on debugging techniques and tools available for these components.

## Pre-Built Host Application Distribution

The following pre-built host application distribution methods support the following architectures:

- Debian Packages (*.deb for debian-bookworm) support architectures: armhf (arm32v7), arm64 (aarch64,arm64v8), i386, amd64 (x86_64)

  > **Note**: When installing the ot-br-posix .deb file for on 32-bit bookworm systems such as armhf (arm32v7), make sure the dhcpcd package on your target system is higher than the recommended bookworm version of 9.4.1-24, which has a [known startup issue](https://github.com/NetworkConfiguration/dhcpcd/issues/323). The bookworm-backports apt source provides version 10.1.0 which is highly recommended. If not, Silicon Labs recommends updating your dhcpcd package version to at least 9.5.1.

- Multiprotocol Container support architectures: armhf **the Multiprotocol Container will be deprecated in 2025
- OpenWRT packages (*ipk for OpenWRT-23.05.3) support architectures: arm64

Below is a table that shows how each application is delivered:

| Applications          | Debian Packages (\*.deb) | Multiprotocol Container | OpenWRT Packages (\*.ipk) | Built Natively |
|-----------------------|-------------------------|-------------------------|--------------------------|----------------|
| CPCD                  | X                       | X                       | X                        | X              |
| Zigbeed               | X                       | X                       | X                        | X              |
| ot-br-posix           | X                       | X                       | X                        | X              |
| ot-cli                |                         | X                       |                          | X              |
| zigbee_z3_gateway*    |                         | X                       |                          | X              |
| zigbee_host_xncp_led  |                         | X                       |                          | X              |
| bt_host_empty         |                         | X                       |                          | X              |
| cpc-hci-bridge        |                         | X                       |                          | X              |
| pro-compliance-posix  |                         | X                       |                          | X              |
