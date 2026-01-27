# Running Multiprotocol Solution on Docker

>**NOTICE: The Multiprotocol Docker Container is now deprecated**

## Raspberry Pi / ARM Host with Docker Setup

The quickest method to set up a multiprotocol host is to use the pre-configured Multiprotocol Docker container on the Raspberry Pi. This container includes everything necessary to easily run the Z3Gateway Zigbee application, the OpenThread Border Router (OTBR) and the ot-cli application, and the BlueZ Bluetooth stack and bluetoothctl, an open source interactive CLI utility for sending commands to the BlueZ stack.

The Multiprotocol Docker container is hosted on DockerHub (hub.docker.com) in the `siliconlabsinc/multiprotocol` repository. Pulling the latest tag is generally the best way to get the latest release by issuing this command:

```bash
docker pull siliconlabsinc/multiprotocol:latest
```

If necessary, you can replace the latest tag with a specific version. To run the Docker container, a helpful *run.sh* script is provided in the SDK in the `app/host/multiprotocol/zigbeed/multiprotocol-container` directory. The script is meant to simplify the docker setup process. Copy it to your Raspberry Pi’s home directory. Make sure the RCP device is flashed and attached to the Raspberry Pi before starting the Docker container.

The Docker container by default uses CPCd security, make sure your RCP has CPC Security Enabled. Before running the docker container, a CPCd security commissioning step is required. CPC is now configured by default to encrypt the data over the SPI or UART serial connection between the host and the EFR32. For detailed information, see https://github.com/SiliconLabs/cpc-daemon/blob/main/readme.md.

The *run.sh* script provides a convenient shortcut to perform the security commissioning. After flashing the EFR32 with the desired image, simply execute `run.sh -K` on the host. This will pull and run the docker container, use cpcd inside the container to commission security and generate a binding key file, then copy the binding key file outside of the container and place it at /etc/binding-key.key on the host filesystem. This file will then be used for all subsequent runs of the container to establish a secure connection with the EFR32.

After CPCd security is commissioned, to start the Docker container, execute the run.sh script without any arguments. The container will start systemd, which is a process management utility. This will make sure that all the components are started in the proper sequence and are kept running. At startup, only CPCd is started.

The `/tmp/multiprotocol-container/log/` directory on the host will be mounted to `/var/log/` inside the Docker container. It contains the file syslog, which will contain log output from cpcd, zigbeed, zigbeed-socat, otbr-agent, and ot-cli, among other programs. Monitor the file live by issuing the following command:

```bash
tail -f /tmp/multiprotocol-container/log/syslog
```

Use the standard `journalctl` utility to monitor the logs for a given systemd service. For example, to monitor the cpcd log from outside of the container, use the following command:

```bash
docker exec -it multiprotocol journalctl -fexu cpcd
```

As a convenience, the command can also be executed by typing `run.sh -l`. Note that by default, verbose cpcd logging is disabled. To enable it, see **TODO LINK TO CPCD FILE CONFIGURATION**. After starting the container, it is a good idea to verify that CPCd successfully connected to the EFR32 by running `run.sh -l` and looking for the log line “Daemon startup was successful”.

A special file `/accept_silabs_msla` will be created in the Docker container to indicate acceptance of the Silicon Labs Master Software License Agreement (MSLA) located at https://www.silabs.com/about-us/legal/master-software-license-agreement.

Once the Docker container is running, you can open a shell by issuing the command:

```bash
run.sh -o
```

The Zigbeed configuration file is located: `/usr/local/etc/zigbeed.conf` and the CPCd configuration file is located: `/usr/local/etc/cpcd.conf`. For more information, see **TODO Link to CPCD CONF**.

The cpcd, zigbeed, zigbee_z3_gateway, zigbee_z3_gateway_cpc, zigbee_host_xncp_led, otbr-agent, ot-ctl, ot-cli, and cpc-hci-bridge binaries are all installed in `/usr/local/bin`. There are systemd configuration files to start up CPCd, zigbeed and socat for Zigbee, OTBR for OpenThread, and cpc-hci-bridge and hciattach for Bluetooth. The definition files for those are in `/etc/systemd/system`. Use systemd commands to start the services from within the Docker container shell as follows. Note that the otbr service takes an iid argument.

```bash
systemctl start zigbeed
systemctl start otbr
systemctl start hciattach
```

From outside of the container, as follows:

```bash
docker exec -it multiprotocol systemctl start zigbeed
docker exec -it multiprotocol systemctl start otbr
docker exec -it multiprotocol systemctl start hciattach
```

For Bluetooth, to avoid interference with Bluetooth running in the container, first disable Bluetooth on the native Raspi host using the
following commands:

```bash
sudo systemctl stop bluetooth
sudo systemctl mask bluetooth.service
```

After starting the container and opening a shell, issue the command:

```bash
service hciattach start
```

This uses the systemd utility to start the necessary processes for the BlueZ stack to connect to the RCP via CPCd.

To interact with the Bluetooth network, issue the command:

```bash
bluetoothctl
```

Once at the bluetoothctl prompt, use the bluetoothctl commands such as list, advertise, connect, and scan to exercise the host Bluetooth stack and RCP. Additional documentation for bluetoothctl and BlueZ is available online.

Note that the zigbeed service automatically starts the zigbeed-socat service, and the hciattach service automatically starts the cpc-hcibridge service.

For convenience, the `run.sh` script has arguments to start each of these services after the container is running:

```bash
-Z starts zigbeed and launches a terminal with zigbee_z3_gateway
-T starts OTBR and opens a terminal with ot-ctl
-L starts Bluetooth and opens a terminal with bluetoothctl
-C launches a terminal with zigbee_z3_gateway_cpc (for use with the Zigbee NCP/OpenThread RCP)
```

You can see the processes by running `ps aux` inside the Docker container. Alternatively, use systemd commands to see the state of the various services as follows:

```bash
systemctl status zigbeed
```

To stop the docker container issue:

```bash
run.sh -s. 
```

## Zigbee Host Applications in Docker

For convenience, a precompiled zigbee_z3_gateway host sample application is included in the Docker container. To run it with Zigbeed, issue the following commands from inside the Docker container shell (or on the host, depending on your installation):

```
Systemctl start zigbeed
/usr/local/bin/zigbee_z3_gateway -p ttyZigbeeNCP
```

The ttyZigbeeNCP refers to the file `/dev/ttyZigbeeNCP`. The zigbee_z3_gateway application automatically prepends /dev to a relative path in the -p argument. The `run.sh -Z` is a script that will run these commands on an already running container. Any Zigbee host application built with EZSP/ASH for communicating with a Zigbee NCP over a UART can also be used with Zigbeed by passing it the PTY device name. This is because to the host application, the PTY device appears exactly like a normal serial device. If theZigbee host application was built for EZSP/SPI, it will have to rebuild for EZSP/ASH to work with Zigbeed.

Some EZSP commands that are not supported with zigbeed:

```
• Set/Get Manufacturing Tokens
• EZSP_SET_MFG_TOKEN
• EZSP_GET_MFG_TOKEN
• Secure EZSP Commands
• EZSP_SET_SECURITY_KEY
• EZSP_SET_SECURITY_PARAMETERS
• EZSP_RESET_TO_FACTORY_DEFAULTS
• EZSP_GET_SECURITY_KEY_STATUS
• Bootloader Commands
• EZSP_LAUNCH_STANDALONE_BOOTLOADER
• EZSP_SEND_BOOTLOADER_MESSAGE
• EZSP_GET_STANDALONE_BOOTLOADER_VERSION_PLAT_MICRO_PHY
• EZSP_INCOMING_BOOTLOAD_MESSAGE_HANDLER
• EZSP_BOOTLOAD_TRANSMIT_COMPLETE_HANDLER
• EZSP_AES_ENCRYPT
```

If using the Zigbee NCP/OpenThread RCP, do not run Zigbeed on the host. Instead use a host app that has been built with the zigbee_ezsp_cpc component. For convenience, a precompiled zigbee_z3_gateway_cpc host sample application has been included in the Docker container. No arguments are required to run it, as it connects directly to CPCd. An optional -c argument can be supplied to specify the CPC daemon instance name to connect to. The default instance connected to is cpcd_0.

## OT CLI Application in Docker

A precompiled OpenThread ot-cli sample application with multi-PAN and CPCd support is also included in the Docker container and SDK. ot-cli is a stand-alone sample host application that includes the OpenThread stack and exposes the standard OpenThread CLI. You can run by issuing this command:

```
/usr/local/bin/ot-cli 'spinel+cpc://cpcd_0?iid=2&iid-list=0'
```

The RCP uses the interface id parameter of the radio-url argument (iid=2) to distinguish between the OpenThread and Zigbeed applications. By default, the `/usr/local/etc/zigbeed.conf` file uses a radio-url argument with iid=1. The iid-list argument in radiourl is to allow spinel frames other than the host interface id. RCP uses the interface id zero to broadcast a spinel frame to all the hosts. The string ‘cpcd_0’ in the radio-url is the default CPCd instance name, which is defined in the `/usr/local/etc/cpcd.conf` file. To enable otcli debugging output, use the command line argument -d \<level\> to enable logging at the desired level 1-5. By default, log messages are printed to syslog. Add the argument -v as well to echo log messages to stdout as well. The `run.sh -O` command can be used as a shortcut to start this application on a running container and open a shell to it.

## OpenThread Border Router (OTBR) Application in Docker

The Multiprotocol Docker container is based on the OpenThread Border Router Docker container. It has all the necessary dependencies to run OTBR over CPC. To run OTBR with iid 2, issue the following commands from inside the Multiprotocol Docker container shell (or on the host, depending on your installation):

```
systemctl start otbr
ot-ctl
```

For convenience, this can be run with `run.sh -T` on a running container and open a shell to it. Note that OTBR uses ot-ctl instead of ot-cli. These two utilities offer similar CLI to the OpenThread network. But ot-ctl is a utility that connects to otbr-agent, which is the process that runs the OpenThread stack for OTBR. **TODO POINT TO WHERE WE DISCUSS CHANING OTBR RADIO URL**

To use OTBR with Zigbee NCP/OpenThread RCP application, OTBR must set iid value to zero in the radio-url. Otherwise, OpenThread RCP will reject the spinel frames for all non-zero iid values. To run OTBR with iid 0, issue the following command from inside the Multiprotocol Docker container:

```
systemctl start otbr
ot-ctl
```

For convenience, this can be run with `run.sh -T 0` on an already running container to run OTBR with iid value zero.

## Bluetooth Host Applications on Docker

The multiprotocol solution uses the standard Linux BlueZ Bluetooth stack on the host. BlueZ and associated utilities are documented extensively online. Once Bluetooth is running as described in section 8: Bluetooth Host Setup, you can use standard Bluetooth tools such as bluetoothctl for a CLI utility and btmon to view the traffic going through the Bluetooth device.
