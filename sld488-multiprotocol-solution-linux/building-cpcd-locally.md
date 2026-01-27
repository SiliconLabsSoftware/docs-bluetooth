# Co-Processor Communication Daemon

The Co-Processor Communication Daemon documentation and software can be found in the github repository: https://github.com/SiliconLabs/cpc-daemon.

The Co-Processor Daemon (CPCd) enables users to have multiple stack protocols interact with a secondary processor (Either an RCP or NCP) over a shared physical link by using multiple endpoints.

CPCd runs with three main components:

1. Daemon Binary (CPCd)
2. libcpc.so which is a library that enables C applications to interact with the Daemon
3. CPCD.Conf configuration file to configure CPCd

The figure below shows how CPCd runs in the Host application.

![CPCd System Diagram](resources/figure-3-1-cpc-system-diagram.png)

## Building CPCd Locally

>**Note: It is important to use the git tag that corresponds to the SDK version of the RCP. For Example 4.1.0.0 GSDK should map to CPCD git commit 4.1.0.0.**

## Install and Make CPCD on the System

```bash
git clone https://github.com/SiliconLabs/cpc_daemon.git
cd cpc_daemon
mkdir build
cd build
cmake ../
make
```

By default, the make install places libcpc.so in `/usr/local/lib/arm-linux-gnueabihf` and sl_cpc.h in `/usr/local/include`.

### Install CPCD and the CPC Library

```bash
sudo make install
sudo ldconfig
sudo cp ../cpcd.conf /usr/local/etc/.
```

### CPCD Service File

If you want CPCD to be included in a system service, create a CPCD Service file named: cpcd.service file. An example of the service file is shown below:

```bash
[Unit]
Description=Cpcd service
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
ExecStart=/usr/bin/stdbuf -o0 /usr/local/bin/cpcd
ExecStop=/bin/kill -WINCH ${MAINPID}
PIDFile=/run/cpcd.pid
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cpcd

[Install]
WantedBy=multi-user.target
```

After moving the cpcd.service file /etc/systemd/system, the cpc daemon can not be started from anywhere with:

```bash
sudo systemctl start cpcd
```

## CPCD Configuration

CPCD allows users to have flexibility in how to connect the secondary (radio) to the host processor. This includes configuring the SPI or UART communication, baudrates, GPIO pins, etc.

### Modify CPCD.Conf File

To modify your CPCd configuration, you can modify the file located in `/usr/local/etc/cpcd.conf`.

Some important Parameters to note:

- **bus_type**: This configures CPCd to expect UART or SPI communication with the RCP.
- **disable_encryption**: This parameter is set to false by default. It is very important that the CPCd encryption setting matches the CPC encryption setting on the co-processor (SL_CPC_SERCURITY_ENABLED component). By default, CPCd and the RCP **enable** encryption.
- **stdout_trace**: This parameter can be configured to print more verbose logging.

More information on CPCD Configuration can be found by following the instructions at https://github.com/SiliconLabs/cpc-daemon/blob/main/readme.md.

### CPCD in a Heavy Traffic Environment

In heavy traffic environments, it is recommended to increase the host's socket buffers to be able to process all of the incoming network packets. An error message like so typically points to the socket buffers filling up on the host:

```bash
Write() at cpc_interface.cpp:242: Broken pipe
```

To mediate this issue, you can increase the socket buffer sizes in /etc/sysctl.conf.

## CPCD Configuration for SPI Communication

This section shows how to configure the host and co-processor to run concurrent multiprotocol (CMP) application with a co-processor over SPI.

Some examples on this page were done with Silicon Labs Gecko SDK, the latest software is now being released as part of Silicon Labs Simplicity SDK which can be found: https://github.com/SiliconLabsSoftware/sisdk-release.

To check your co-processor configuration, refer to [Co-Processor Configuration for SPI](co-processor-configuration.md#configure-spi-rcp).

## Running CPCD with SPI Configuration

In the CPCD.CONF file you will need to change **bus_type: SPI**. If you changed any host GPIO pin outs those will need to be reflected on the CPCD.CONF as well. Furthermore, by default CPCD has the bootloader_recovery_pins_enabled: false. If you would like to make use of using the bootloader wake pin and nReset pin then you will need to enable this parameter to true.

Assuming that CPCD has already been installed on the host you can run this command to set up connection between the Host and the RCP. Note you can see that the bus is now changed to **SPI**. For questions on how to configure the host to run SPI and how to connect the co-processor to the host please reference section 3: Local Host Configuration.

```bash
sudo /usr/local/bin/cpcd -c ~/cpc-daemon/cpcd.conf

WARNING in function 'main' in file /home/pi/cpc-daemon/main.c at line #186 : Running CPCd as 'root' is not recommended. Proceed at your own risk.
[21:55:49:125777] Info : [CPCd v4.3.1.0] [Library API v3] [RCP Protocol v4]
[21:55:49:126074] Info : Git commit: 133b29678b3d0bc7578e098d2f46b4d5bcd2ebb4 / branch:
[21:55:49:126115] Info : Sources hash: 1253de9aeadd9a3091781c41f4487219097dbd92209a7913d0818747b9a3da3c
[21:55:49:126157] WARNING : In function 'main' in file /home/pi/cpc-daemon/main.c at line #186 : Running CPCd as 'root' is not recommended. Proceed at your own risk.
[21:55:49:126288] Info : Reading cli arguments
[21:55:49:126340] Info : /usr/local/bin/cpcd -c /home/pi/cpc-daemon/cpcd.conf
[21:55:49:126968] Info : Reading configuration
[21:55:49:127008] Info :   file_path = /home/pi/cpc-daemon/cpcd.conf
[21:55:49:127037] Info :   instance_name = cpcd_0
[21:55:49:127064] Info :   socket_folder = /dev/shm
[21:55:49:127092] Info :   operation_mode = MODE_NORMAL
[21:55:49:127120] Info :   use_encryption = false
[21:55:49:127146] Info :   binding_key_file = /root/.cpcd/binding.key
[21:55:49:127173] Info :   stdout_tracing = false
[21:55:49:127199] Info :   file_tracing = false
[21:55:49:127225] Info :   lttng_tracing = false
[21:55:49:127251] Info :   enable_frame_trace = false
[21:55:49:127277] Info :   traces_folder = /dev/shm/cpcd-traces
[21:55:49:127303] Info :   bus = SPI
[21:55:49:127329] Info :   spi_file = /dev/spidev0.0
[21:55:49:127355] Info :   spi_bitrate = 1000000
[21:55:49:127381] Info :   spi_irq_chip = gpiochip0
[21:55:49:127408] Info :   spi_irq_pin = 22
[21:55:49:127434] Info :   fu_recovery_pins_enabled = true
[21:55:49:127460] Info :   fu_reset_chip = gpiochip0
[21:55:49:127487] Info :   fu_spi_reset_pin = 23
[21:55:49:127513] Info :   fu_wake_chip = gpiochip0
[21:55:49:127539] Info :   fu_spi_wake_pin = 24
[21:55:49:127565] Info :   fu_connect_to_bootloader = false
[21:55:49:127591] Info :   fu_enter_bootloader = false
[21:55:49:127617] Info :   restart_cpcd = false
[21:55:49:127643] Info :   application_version_validation = false
[21:55:49:127669] Info :   print_secondary_versions_and_exit = false
[21:55:49:127696] Info :   use_noop_keep_alive = false
[21:55:49:127722] Info :   reset_sequence = true
[21:55:49:127748] Info :   stats_interval = 0
[21:55:49:127774] Info :   rlimit_nofile = 2000
[21:55:49:127800] Info : ENCRYPTION IS DISABLED
[21:55:49:127826] Info : Starting daemon in normal mode
[21:55:49:130871] Info : Connecting to Secondary...
[21:55:49:308569] Info : RX capability is 256 bytes
[21:55:49:308614] Info : Connected to Secondary
[21:55:49:309227] Info : Secondary Protocol v4
[21:55:49:310603] Info : Secondary CPC v4.3.1
[21:55:49:311238] Info : Secondary bus bitrate is 0
[21:55:49:311893] Info : Secondary max bus bitrate is 4000000
[21:55:49:311905] Info : SPI bitrate from the config file is lesser than the maximum value returned by the secondary. For performance reason, consider raising it.
[21:55:49:311916] Info : The negotiated SPI bitrate will be 1000000
[21:55:49:312651] Info : Secondary APP vUNDEFINED
[21:55:49:312774] Info : Daemon startup was successful. Waiting for client connections
```

As you can see, the following CPCD configuration parameters will be printed to the console. If the CPCD connection is successful, you will see **Info: Daemon Startup was successful. Waiting for client connections**.

More information and documentation on CPCD can be found at https://github.com/SiliconLabs/cpc-daemon.
