# Running CPCd, Zigbeed, Z3Gateway, OTBR Concurrently on Raspbian

The purpose of this guide is to provide a hands-on example of running the previously discussed multiprotocol processes concurrently. This guide aims to demonstrate the process of setting up and configuring the Co-Processor Communication Daemon (CPCd), Zigbee Daemon (zigbeed), Zigbee Host application (Z3Gateway), and the OpenThread Border Router (OTBR). This guide assumes that the previous sections: *Building CPCd on Raspbian*, *Building zigbeed and Z3Gateway on Raspbian (UART)*, *Building Zigbee host*, and *Building OTBR on Raspbian* have been completed.

## Running CPCd

This step must be done before before running the Zigbee or OpenThread processes. CPCd can either be run from *systemd* or via the binary in the build folder, this guide will cover both. Note that by default CPCd encryption is turned on. This means you will have an additional CPCd binding step. **The steps below have disabled CPC Security on both CPCd and the RCP**.

### Running CPCd from the Binary

This method runs the CPCd binary directly in the terminal. If logging is enabled, debug logs will appear as shown below. The log level can be adjusted in the `cpcd.conf` file.

```bash
From the cpcd/build/debug/out folder:

./cpcd -c [/path_to_cpcd_conf] -[options]

or

cpcd/build/debug/out/cpcd -c [/path_to_cpcd_conf] -[options]
```

**If you are running CPCd with security enabled you will first need to run:**

```bash
./cpcd -c cpcd.conf --bind ecdh
```

### Running CPCd as a Service in the Background

If you chose to run CPCd as a service in the background, you can create a `cpcd.service` file and move it to the following location: `/etc/systemd/system`. Now with this file installed, you can simply start CPCd by running the following command:

```bash
systemctl start cpcd
```

To view the status of this process:

```bash
systemctl status cpcd
```

Should yield an output like the following:

```bash
● cpcd.service - Cpcd service
     Loaded: loaded (/etc/systemd/system/cpcd.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-09-23 20:31:18 UTC; 4s ago
   Main PID: 1170 (cpcd)
      Tasks: 6 (limit: 9238)
        CPU: 32ms
     CGroup: /system.slice/cpcd.service
             └─1170 /usr/local/bin/cpcd
```

### CPCd Log Example

```bash
[2024-09-20T19:28:38.039762Z] Info : [CPCd v4.5.0.0] [Library API v3] [RCP Protocol v5]
[2024-09-20T19:28:38.040027Z] Info : Git commit: f7cb761e18f338f09b4a59f07f6458aefb0a0f05 / branch: refs/heads/main
[2024-09-20T19:28:38.040047Z] Info : Reading cli arguments
[2024-09-20T19:28:38.040067Z] Info : ./cpcd 
[2024-09-20T19:28:38.040999Z] Info : Reading configuration
[2024-09-20T19:28:38.041019Z] Info :   file_path = /usr/local/etc/cpcd.conf
[2024-09-20T19:28:38.041029Z] Info :   instance_name = cpcd_0
[2024-09-20T19:28:38.041038Z] Info :   socket_folder = /dev/shm
[2024-09-20T19:28:38.041048Z] Info :   operation_mode = MODE_NORMAL
[2024-09-20T19:28:38.041057Z] Info :   use_encryption = false
[2024-09-20T19:28:38.041066Z] Info :   binding_key_file = /home/orkevlar/.cpcd/binding.key
[2024-09-20T19:28:38.041075Z] Info :   stdout_tracing = false
[2024-09-20T19:28:38.041084Z] Info :   file_tracing = false
[2024-09-20T19:28:38.041092Z] Info :   lttng_tracing = false
[2024-09-20T19:28:38.041100Z] Info :   enable_frame_trace = false
[2024-09-20T19:28:38.041108Z] Info :   traces_folder = /dev/shm/cpcd-traces
[2024-09-20T19:28:38.041117Z] Info :   bus = UART
[2024-09-20T19:28:38.041125Z] Info :   uart_baudrate = 115200
[2024-09-20T19:28:38.041134Z] Info :   uart_hardflow = true
[2024-09-20T19:28:38.041142Z] Info :   uart_file = /dev/ttyACM0
[2024-09-20T19:28:38.041151Z] Info :   fwu_recovery_pins_enabled = false
[2024-09-20T19:28:38.041160Z] Info :   fwu_connect_to_bootloader = false
[2024-09-20T19:28:38.041168Z] Info :   fwu_enter_bootloader = false
[2024-09-20T19:28:38.041177Z] Info :   restart_cpcd = false
[2024-09-20T19:28:38.041185Z] Info :   application_version_validation = false
[2024-09-20T19:28:38.041194Z] Info :   print_secondary_versions_and_exit = false
[2024-09-20T19:28:38.041202Z] Info :   use_noop_keep_alive = false
[2024-09-20T19:28:38.041210Z] Info :   reset_sequence = true
[2024-09-20T19:28:38.041218Z] Info :   stats_interval = 0
[2024-09-20T19:28:38.041226Z] Info :   rlimit_nofile = 2000
[2024-09-20T19:28:38.041236Z] Info : ENCRYPTION IS DISABLED 
[2024-09-20T19:28:38.052580Z] Info : Starting daemon in normal mode
[2024-09-20T19:28:38.063960Z] Info : Connecting to Secondary...
[2024-09-20T19:28:38.230299Z] Info : RX capability is 256 bytes
[2024-09-20T19:28:38.230329Z] Info : Connected to Secondary
[2024-09-20T19:28:38.236297Z] Info : Secondary Protocol v5
[2024-09-20T19:28:38.249294Z] Info : Secondary CPC v4.5.0
[2024-09-20T19:28:38.261343Z] Info : Secondary bus bitrate is 115200
[2024-09-20T19:28:38.273286Z] Info : Secondary APP vUNDEFINED
[2024-09-20T19:28:38.273546Z] Info : Daemon startup was successful. Waiting for client connections
```

When either the `otbr-agent` or `zigbeed` are connected to `cpcd`, the following will appear in the `cpcd` logs:

```bash
[2024-09-23T18:09:26.299013Z] Info : New client connection using library v4.5.2.0
[2024-09-23T18:09:26.305764Z] Info : Opened connection socket for ep#12
[2024-09-23T18:09:26.313604Z] Info : Endpoint socket #12: Client connected. 1 connection(s)
```

When both are connected, the logs will show:

```bash
[2024-09-23T20:01:07.851455Z] Info : New client connection using library v4.5.2.0
[2024-09-23T20:01:07.858154Z] Info : Endpoint socket #12: Client connected. 2 connection(s)
```

When the CPC-HCI bridge is connected to CPCd, the log will show:

```bash
[2024-11-15T20:43:46.043660Z] Info : New client connection using library v4.5.2.0
[2024-11-15T20:43:46.056109Z] Info : Opened connection socket for ep#14
[2024-11-15T20:43:46.065940Z] Info : Endpoint socket #14: Client connected. 1 connection(s)
```

## Running Zigbeed

This must be done **after** CPCd is running.

### Running Socat

You can run Socat from the command line by following this command:

```bash
socat -x -v pty,link=/dev/ttyZigbeeNCP pty,link=/tmp/ttyZigbeeNCP
```

This command creates a virtual serial port for Zigbee Communication.

## Running Socat as a Service in the Background

Similarly, Socat can be run as a service with `systemd`.

This service requires the `zigbeed-socat.service` file to be created and copied into the `/etc/systemd/system/..` directory. Now you can start zigbeed-socat with:

```bash
sudo systemctl start zigbeed-socat
```

To view the status of this process:

```bash
systemctl status zigbeed-socat
```

Should yield an output like the following:

```bash
● zigbeed-socat.service - Zigbeed socat helper service
     Loaded: loaded (/etc/systemd/system/zigbeed-socat.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-09-23 20:31:18 UTC; 4s ago
   Main PID: 1171 (socat)
      Tasks: 1 (limit: 9238)
        CPU: 14ms
     CGroup: /system.slice/zigbeed-socat.service
             └─1171 /usr/bin/socat -v pty,link=/dev/ttyZigbeeNCP pty,link=/tmp/ttyZigbeeNCP
```

### Running Zigbeed from the Binary

This method runs the Zigbeed binary directly in the terminal. If logging is enabled, debug logs will appear as shown below. The log level can be adjusted in the `zigbeed.conf` file.

```bash
From the zigbeed/build/debug/out folder:

./zigbeed -[options]

or

zigbeed/build/debug/out/zigbeed -[options]
```

The zigbeed logs will appear in the terminal as shown below:

```bash
By using this software, you are agreeing to the Silicon Labs MSLA found at https://www.silabs.com/about-us/legal/master-software-license-agreement.
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:0, tid:0, cmd:PROP_VALUE_IS, key:LAST_STATUS, status:RESET_SOFTWARE
zigbeed[5479]: [I] P-SpinelDrive-: co-processor reset: RESET_SOFTWARE
zigbeed[5479]: [D] P-SpinelDrive-: Sent spinel frame, flg:0x2, iid:1, tid:0, cmd:RESET
zigbeed[5479]: [D] P-SpinelDrive-: Waiting response: key=0
zigbeed[5479]: [C] P-SpinelDrive-: Software reset co-processor successfully
zigbeed[5479]: [D] P-SpinelDrive-: Waiting response: key=1
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:1, cmd:PROP_VALUE_IS, key:PROTOCOL_VERSION, major:4, minor:3
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:0, tid:0, cmd:PROP_VALUE_IS, key:LAST_STATUS, status:RESET_SOFTWARE
zigbeed[5479]: [I] P-SpinelDrive-: co-processor reset: RESET_SOFTWARE
zigbeed[5479]: [D] P-SpinelDrive-: Waiting response: key=2
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:1, cmd:PROP_VALUE_IS, key:NCP_VERSION, version:SL-OPENTHREAD/2.5.1.0_GitHub-1fceb225b;
zigbeed[5479]: [D] P-SpinelDrive-: Waiting response: key=5
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:1, cmd:PROP_VALUE_IS, key:CAPS, caps:COUNTERS UNSOL_UPDATE_FILTER 802_15_4_2450MHZ_OQP
zigbeed[5479]: [D] Platform------: instance init:0xcd178 - iid = 1
zigbeed[5479]: [D] P-RadioSpinel-: Wait response: tid=1 key=8
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:1, cmd:PROP_VALUE_IS, key:HWADDR, eui64:90395efffee405e1
zigbeed[5479]: [D] P-RadioSpinel-: RCP supports crash dump logging. Requesting crash dump.
zigbeed[5479]: [D] P-RadioSpinel-: Wait response: tid=2 key=178
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:2, cmd:PROP_VALUE_IS, key:LAST_STATUS, status:OK
zigbeed[5479]: [D] P-RadioSpinel-: Wait response: tid=3 key=176
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:3, cmd:PROP_VALUE_IS, key:RCP_API_VERSION, version:10
zigbeed[5479]: [D] P-RadioSpinel-: Wait response: tid=4 key=4619
zigbeed[5479]: [D] P-SpinelDrive-: Received spinel frame, flg:0x2, iid:1, tid:4, cmd:PROP_VALUE_IS, key:RADIO_CAPS, caps:255
zigbeed[5479]: Zigbeed started
zigbeed[5479]: RCP version: SL-OPENTHREAD/2.5.1.0_GitHub-1fceb225b; EFR32; Sep 20 2024 15:27:16
zigbeed[5479]: Zigbeed Version: GSDK 8.0.1 - Jul 22 2024 - 21:58:48
```

To record the zigbeed logs to a file, you can use a command like this:

```bash
./zigbeed 2>&1  | sudo tee [path-to-out-file]
```

### Running Zigbeed as a Service in the Background

Again, running Zigbeed as a service in the background will require you to install the `zigbeed.service` file into `/etc/systemd/system` directory.

```bash
systemctl start zigbeed
```

To view the status of this process:

```bash
systemctl status zigbeed
```

Should yield an output like the following:

```bash
● zigbeed.service - Zigbeed service
     Loaded: loaded (/etc/systemd/system/zigbeed.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-09-23 20:31:18 UTC; 4s ago
   Main PID: 1173 (zigbeed)
      Tasks: 1 (limit: 9238)
        CPU: 17ms
     CGroup: /system.slice/zigbeed.service
             └─1173 /usr/local/bin/zigbeed
```

## Running the Zigbee Host Application

This must be done **after** Zigbeed has been started. The Zigbee Host application is run directly in the terminal via the binary in the build folder.

```bash
From the zigbee_z3_gateway/build/debug/out folder:

./zigbee_z3_gateway -n 0 -p /dev/ttyZigbeeNCP

or

zigbee_z3_gateway/build/debug/out/z3gateway -n 0 -p /dev/ttyZigbeeNCP
```

Z3Gateway CLI

```bash
Reset info: 11 (SOFTWARE)
ezsp ver 0x0E stack type 0x02 stack ver. [8.0.1 GA build 270]
Ezsp Config: set address table size to 0x0002:Success: set
Ezsp Config: set TC addr cache to 0x0002:Success: set
Ezsp Config: set MAC indirect TX timeout to 0x1E00:Success: set
Ezsp Config: set max hops to 0x001E:Success: set
Ezsp Config: set tx power mode to 0x8000:Success: set
Ezsp Config: set supported networks to 0x0001:Success: set
Ezsp Config: set stack profile to 0x0002:Success: set
Ezsp Config: set security level to 0x0005:Success: set
Ezsp Value : set end device keep alive support mode to 0x00000003:Success: set
Ezsp Policy: set binding modify to "allow for valid endpoints & clusters only":Success: set
Ezsp Policy: set message content in msgSent to "return":Success: set
Ezsp Value : set maximum incoming transfer size to 0x00000052:Success: set
Ezsp Value : set maximum outgoing transfer size to 0x00000052:Success: set
Ezsp Value : set default timeout for transient device table to 0x00002710:Success: set
Ezsp Config: set binding table size to 0x0002:Success: set
Ezsp Config: set key table size to 0x0004:Success: set
Ezsp Config: set max end device children to 0x0006:Success: set
Ezsp Config: set aps unicast message count to 0x000A:Success: set
Ezsp Config: set broadcast table size to 0x000F:Success: set
Ezsp Config: set neighbor table size to 0x0010:Success: set
Ezsp Config: set end device poll timeout to 0x0008:Success: set
Ezsp Config: set zll group addresses to 0x0000:Success: set
Ezsp Config: set zll rssi threshold to 0xFFD8:Success: set
Ezsp Config: set transient key timeout to 0x012C:Success: set
Ezsp Config: set retry size to 0x0010:Success: set
Ezsp Endpoint 1 added, profile 0x0104, in clusters: 8, out clusters 17
Ezsp Endpoint 242 added, profile 0xA1E0, in clusters: 0, out clusters 1
Starting identifying on endpoint 0x01, identify time is 0 sec
Stopping identifying on endpoint 0x01
No endpoints identifying; stopping identification feedback.
Found 0 files
zigbee_z3_gateway>
```

## Running the OTBR

In the Multiprotocol context, this must be done **after** CPCd is running. The `otbr-agent` can be run as a service with `systemd` or directly in the terminal via the binary.

### Running the otbr-agent Binary

The binary is located in: `~/simplicity_sdk/util/third_party/ot-br-posix/build/otbr/src/agent`. The following command will start the otbr-agent, and logs will appear in the terminal window.

```bash
sudo ./otbr-agent -d 6 -v -I wpan0 -B eth0 'spinel+cpc://cpcd_0?iid=2&iid-list=0'
```

If the Radio URL is **not** configured properly, the OTBR will respond with the following error message:

```bash
Aug 14 14:18:21 raspberrypi otbr-agent[8037]: 49d.19:21:57.018 [C] Platform------: Init() at radio.cpp:107: InvalidArgument
Aug 14 14:18:21 raspberrypi systemd[1]: otbr-agent.service: Main process exited, code=exited, status=2/INVALIDARGUMENT
```

The OTBR logs will appear similar to the following if it is running properly:

```bash
otbr-agent[1892]: [NOTE]-AGENT---: Running 0.3.0-v2024.6.1-0-1-g36e12f01
otbr-agent[1892]: [NOTE]-AGENT---: Thread version: 1.4
otbr-agent[1892]: [NOTE]-AGENT---: Thread interface: wpan0
otbr-agent[1892]: [NOTE]-AGENT---: Radio URL: spinel+cpc://cpcd_0?iid=2&iid-list=0
otbr-agent[1892]: [NOTE]-ILS-----: Infra link selected: eth0
otbr-agent[1892]: [INFO]-NCP-----: OpenThread log level changed to 4
otbr-agent[1892]: 49d.17:11:58.417 [I] P-SpinelDrive-: co-processor reset: RESET_SOFTWARE
otbr-agent[1892]: 49d.17:12:00.447 [C] P-SpinelDrive-: Software reset co-processor successfully
otbr-agent[1892]: 49d.17:12:00.504 [I] P-Netif-------: Sent request#1 to set addr_gen_mode to 1
otbr-agent[1892]: 00:00:00.087 [I] ChildSupervsn-: Timeout: 0 -> 190
otbr-agent[1892]: 00:00:00.088 [I] TrelInterface-: Enabled interface, local port:60358
otbr-agent[1892]: 00:00:00.088 [I] RoutingManager: Initializing - InfraIfIndex:2
otbr-agent[1892]: 00:00:00.089 [I] InfraIf-------: Init infra netif 2
otbr-agent[1892]: 00:00:00.089 [I] Settings------: Read BrUlaPrefix fd8b:b68:2457::/48
otbr-agent[1892]: 00:00:00.089 [N] RoutingManager: BR ULA prefix: fd8b:b68:2457::/48 (loaded)
otbr-agent[1892]: 00:00:00.089 [I] RoutingManager: Generated local OMR prefix: fd8b:b68:2457:1::/64
otbr-agent[1892]: 00:00:00.089 [I] RoutingManager: Generated local NAT64 prefix: fd8b:b68:2457:2:0:0::/96
otbr-agent[1892]: 00:00:00.089 [N] RoutingManager: Local on-link prefix: fdde:ad00:beef:cafe::/64
otbr-agent[1892]: 00:00:00.089 [I] InfraIf-------: State changed: NOT RUNNING -> RUNNING
otbr-agent[1892]: 00:00:00.089 [I] RoutingManager: Enabling
otbr-agent[1892]: 00:00:00.089 [I] Nat64---------: IPv4 CIDR for NAT64: 192.168.255.0/24 (actual address pool: 192.168.255.1 - 192.168.255.254, 254 addresses)
otbr-agent[1892]: 00:00:00.089 [I] Nat64---------: NAT64 translator is now Disabled
otbr-agent[1892]: 00:00:00.090 [I] P-Resolver----: Got nameserver #0: 192.168.0.1
otbr-agent[1892]: [INFO]-UTILS---: Set state callback: OK
otbr-agent[1892]: 00:00:00.090 [I] Nat64---------: NAT64 translator is now NotRunning
otbr-agent[1892]: [INFO]-BA------: Start Thread Border Agent
otbr-agent[1892]: [INFO]-BA------: Publish meshcop service OpenThread BorderRouter #102C._meshcop._udp.local.
otbr-agent[1892]: 00:00:00.091 [I] Settings------: Read BorderAgentId {id:27497e430c9fd5c4b52cb337e06f77a7}
otbr-agent[1892]: [INFO]-MDNS----: Registering service OpenThread BorderRouter #102C._meshcop._udp
otbr-agent[1892]: 00:00:00.097 [I] BbrLocal------: Add Domain Prefix: ::/0, NotFound
otbr-agent[1892]: 00:00:00.098 [I] BbrLocal------: Add BBR Service: seqno (73), delay (5s), timeout (3600s), InvalidState
otbr-agent[1892]: [INFO]-ADPROXY-: Started
otbr-agent[1892]: [INFO]-DPROXY--: Started
otbr-agent[1892]: [INFO]-APP-----: Thread Border Router started on AIL eth0.
otbr-agent[1892]: 00:00:00.108 [I] Notifier------: StateChanged (0x42038210) [MLAddr NetData PanId NetName ExtPanId BbrState Nat64]
otbr-agent[1892]: 00:00:00.108 [I] Bbr-----------: Start listening on port 61631
otbr-agent[1892]: 00:00:00.108 [I] Bbr-----------: Backbone TMF subscribes ff32:40:fdde:ad00:beef:0:0:3: OK
otbr-agent[1892]: 00:00:00.109 [I] BbrManager----: Start Backbone TMF agent: OK
otbr-agent[1892]: 00:00:00.114 [I] Platform------: Execute command `ipset flush otbr-ingress-allow-dst-swap` = 0
otbr-agent[1892]: 00:00:00.118 [I] Platform------: Execute command `ipset flush otbr-ingress-deny-src-swap` = 0
otbr-agent[1892]: 00:00:00.122 [I] Platform------: Execute command `ipset add otbr-ingress-deny-src-swap fdde:ad00:beef:0::/64 -exist` = 0
otbr-agent[1892]: 00:00:00.130 [I] Platform------: Execute command `ipset swap otbr-ingress-deny-src-swap otbr-ingress-deny-src` = 0
otbr-agent[1892]: 00:00:00.134 [I] Platform------: Execute command `ipset swap otbr-ingress-allow-dst-swap otbr-ingress-allow-dst` = 0
otbr-agent[1892]: 00:00:00.134 [I] P-Netif-------: NAT64 CIDR updated to 192.168.255.0/24.
otbr-agent[1892]: 00:00:00.135 [I] P-Netif-------: Deleting route for NAT64
otbr-agent[1892]: 00:00:00.135 [I] P-McastRtMgr--: Disable: OK
otbr-agent[1892]: [INFO]-BA------: Publish meshcop service OpenThread BorderRouter #102C._meshcop._udp.local.
otbr-agent[1892]: 00:00:00.135 [I] RouterTable---: Route table
otbr-agent[1892]: 00:00:00.135 [I] TrelInterface-: Registering DNS-SD service: port:60358, txt:"xa=f2385e307513102c, xp=dead00beef00cafe"
otbr-agent[1892]: 00:00:00.142 [I] P-Netif-------: Host netif is down
otbr-agent[1892]: 00:00:00.143 [I] P-Netif-------: Succeeded to process request#1
otbr-agent[1892]: 00:00:00.143 [W] P-Netif-------: Failed to process request#2: No such process
otbr-agent[1892]: [INFO]-MDNS----: Successfully registered service OpenThread BorderRouter #102C._meshcop._udp
otbr-agent[1892]: [INFO]-BA------: Result of publish meshcop service OpenThread BorderRouter #102C._meshcop._udp.local: OK
otbr-agent[1892]: [INFO]-BA------: Result of publish meshcop service OpenThread BorderRouter #102C._meshcop._udp.local: OK
```

You should now be able to interact with the OT CLI:

```bash
sudo ot-ctl state
enabled
```

### Running the OTBR as a Service

OTBR Service can be started as by calling:

```bash
sudo systemctl start otbr-agent 
```

## Running the Bluetooth Host Applications

For both multiprotocol RCP and NCP architectures, CPCd must be started before running the Bluetooth host applications.

### Bluetooth Host Applications for Multiprotocol RCP

Follow the steps below after CPCd starts up successfully.

1. Run the CPC-HCI bridge application (bt_host_cpc_hci_bridge). It connects to CPCd using the standard instance name cpcd_0, opens a CPC endpoint to the BLE RCP running on the EFR, and creates a numbered virtual serial device on the host, for example `/dev/pts/2`. The actual number may vary. For convenience, `bt_host_cpc_hci_bridge` also creates a symlink to the device from `pts_hci` in the working directory.

   ```bash
   [I] Silicon Labs | CPC-HCI bridge
   [I] CPC successfully initialized with 'cpcd_0'
   [I] CPC Bluetooth endpoint opened
   [I] PTY device file opened at '/dev/pts/6'
   ```

2. Use the following command to attach the Bluetooth stack to the newly created virtual serial device, where \<device\> is the name of the virtual serial device:

   ```bash
   sudo btattach -B <device> -S 115200
   ```

3. Run `sudo bluetoothctl` to start the Bluetooth CLI utility and use commands to control the Bluetooth controller device.

4. A utility that comes with the standard Bluetooth tools, called `btmon`, can be used to view the HCI traffic between the Bluetooth host and the controller.

    * To view the HCI traces for live debugging, run `sudo btmon &`.
    * To save the HCI traces and debug later using tools such as Wireshark, run `sudo btmon --write ~/hcitrace.snoop`.

### Bluetooth Host Application for Multiprotocol NCP

Run the example Bluetooth host application `bt_host_empty` with the following command:

```bash
./bt_host_empty -C cpcd_0
```

The -C option is used to specify the CPC instance name (ex: -C cpcd_0).
