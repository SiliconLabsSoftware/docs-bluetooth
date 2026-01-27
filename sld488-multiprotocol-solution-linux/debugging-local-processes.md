# Debugging with Logs and Silicon Labs Components

## Introduction

This is a comprehensive overview of how to enable and read logs for each component in a Concurrent Multiprotocol Radio Co-Processor (CMP RCP) setup. It also provides general debugging advice for Thread and Zigbee protocols.

## Details

Debugging a CMP RCP setup can be difficult due to the many components that make up the system. For a Zigbee and OpenThread setup, for example, the components that enable the protocols are:

- **Radio Co-Processor (RCP)**: Implements the physical layer and connects to the Co-Processor Communication Daemon (CPCd) as the secondary.
- **CPCd**: Multiplexes protocol streams and communicates with the RCP over a serial link such as UART or SPI.
- **OTBR Agent**: Host program that implements the OpenThread (OT) Stack.
- **Zigbeed**: Host program that implements the Zigbee Stack.
- **Zigbee Host Application (Z3Gateway)**: Communicates with Zigbeed using EmberZNet Serial Protocol (EZSP) over a virtual serial port.

## Debugging RCP

The RCP is responsible for receiving packets over the air and filtering data for the host-side processes to receive.

Here are several ways to debug with an RCP:

- Use the [packet trace interface (PTI)](https://community.silabs.com/s/article/why-should-i-enable-the-pti-peripheral-x?language=en_US) to view data received by the RCP along with RAIL messages.
- Configure [SWO](https://community.silabs.com/s/article/how-to-send-string-through-the-swo-interface-by-calling-printf-function-x?language=en_US) debug to print out logs.
- Use Ozone for debugging if the issue is reproduced with a Wireless Starter Kit (WSTK).
- Check the CPCd logs for RCP connection status (systemctl status cpcd).
- RCP logs with OT stack (see [Debugging OTBR](debugging-local-processes.md#debugging-otbr)).

When using an OTBR + RCP setup, you can enable [efr32RadioCounters](https://github.com/SiliconLabsSoftware/sisdk-release/blob/sisdk-2025.12/openthread/platform-abstraction/include/radio_counters.h) by following the steps below:

1. Add the "EFR32 Platform extension" component to your RCP project.

   - `ot_platform_ext` component if generating project with slc-cli

2. Set SL_CATALOG_OPENTHREAD_EFR32_CLI_PRESENT = 1.
3. Build OTBR with -DSL_OT_EFR32_CLI=ON to enable the host-side API.
4. Use cli command `ot-ctl efr32 counters radio` or call `otPlatRadioExtensionGetRadioCounters()` on the host to get the counter data.

## Debugging CPCd

[CPCd](https://github.com/SiliconLabs/cpc-daemon) is a key component in a multiprotocol architecture, bridging the physical bits received by the RCP and forwarding the information to the host stack process. The `cpcd.conf` file contains fields to enable tracing:

```
# Prints tracing information to stdout
# Optional, defaults to 'false'
# Allowed values are 'true' or 'false'
stdout_trace: true

# Prints tracing information to a file located under traces_folder
# Optional, defaults to 'false'
# Allowed values are 'true' or 'false'
trace_to_file: true

# Traces folder
# Optional, defaults to '/dev/shm/cpcd-traces'
# Folder mounted on a tmpfs is preferred
traces_folder: /dev/shm/cpcd-traces

# Enable frame trace
# Optional, defaults to 'false'
# Allowed values are 'true' or 'false'
enable_frame_trace: true
```

>**Note**: These are not enabled by default so they must enabled to see the [spinel](https://github.com/openthread/spinel-spec/blob/master/core/intro-frames.md) frame traces. This is beneficial if CPC security is disabled in both the RCP and CPCd. In enabling these traces, the raw spinel frames being transferred along with the [CPC headers](https://github.com/SiliconLabs/cpc-daemon/blob/133b29678b3d0bc7578e098d2f46b4d5bcd2ebb4/doc/uart.md?plain=1#L23) can be debugged.

The CPCd process itself will output information to syslog about the state of connection of the secondary (RCP), along with information about how many connections and endpoint has. If a stack process, such as Zigbeed, is disconnected something like the following will be printed:

```
Info : Endpoint socket #12: Client connected. 2 connections
Info : Endpoint socket #12: Client disconnected. 1 connections
```

If the RCP is stuck or CPCd is unable to connect, you can view the live CPCd syslogs with:

```
tail -F /var/log/syslog | grep cpcd
```

First, you must enable the Zigbee logs in the zigbeed.conf file:

```
# Debug level
# Optional
# The debug level for the Spinel driver.
# Message are sent to syslog if the syslog service is running.
# All messages with level less than or equal to the supplied value are logged.
# For example, setting debug-level=5 logs all messages.
# 0=NONE, 1=CRIT, 2=WARN, 3=NOTE, 4=INFO, 5=DEBG
# Uncomment to enable.

# debug-level=5
```

Note that this is not enabled by default so you must remove the '#' in the `zigbeed.conf` file to obtain the Zigbeed logs.

The Zigbeed logs will display vital information such as network data, spinel transactions, EmberZNet API prints (such as `emberSendUnicast()` information), user custom syslog prints, RCP disconnections, and Zigbeed crashes/resets.

A few Silicon Labs components can be added to a Zigbeed project to help debug:

- [Counters](https://docs.silabs.com/zigbee/latest/zigbee-af-api/counters): This component provides support for reading and manipulating counters that record different events in the EmberZnet stack.
- [Stack Diagnostics](https://docs.silabs.com/d/zigbee-af-api/6.9/plugin-stack-diagnostics): This component provides support for reading and manipulating counters that record different events in the stack.
- [Packet Handoff](https://community.silabs.com/s/article/packet-handoff-plugin-how-to-guide?language=en_US#:~:text=Details-,The%20packet%20handoff%20functionality%20allows%20the%20application%20to%20get%20access,journey%20through%20the%20Zigbee%20stack.): This component hooks directly into the stack and provides a mechanism to intercept packets and hand them off to other components as a flat buffer.

In the Zigbeed code, you can make use of the APIs from these components to print out helpful debug information. The information can then be found in the syslogs. For example, you can use the packet handoff in combination with debug prints to display a packet:

```bash
// packetHandoff
sl_zigbee_packet_action_t sl_zigbee_af_incoming_packet_filter_cb(sl_zigbee_zigbee_packet_type_t packetType, uint8_t* packetData, uint8_t* size_p, void* data) {
  switch (packetType)
    {
    //case <packetType>:
      //code
      //break;
    case SL_ZIGBEE_ZIGBEE_PACKET_TYPE_APS_DATA: // APS packet
      syslog(LOG_INFO, "APS_DATA ----> %u", packetType);
      break;
    default:
      break;
    }

  syslog(LOG_INFO, "Packet data bytes ---->");
  for (int i = 0; i < *size_p; ++i)
    {
      syslog(LOG_INFO, "%x", packetData[i]);
    }

  return SL_ZIGBEE_ACCEPT_PACKET;
}
```

## Debugging OTBR

If there is any issue with the Thread network and the RCP is working fine, then you can view the OT process syslogs by running:

```
tail -F /var/log/syslog | grep otbr-agent
```

Unlike Zigbeed, the Thread stack does not have a .conf file to enable logging. Logging can be enabled by setting the log level of your OTBR agent to 5:

```
Sudo ot-ctl log level 5
```

For more information, refer to [OpenThread CLI Commands](https://openthread.io/reference/cli/commands).

Before doing this, the OTBR on the host must be built with the following configuration flags while calling the setup script:

```
OTBR_OPTIONS='-DOT_LOG_LEVEL=DEBG -DOT_FULL_LOGS=ON'
```

The OT agent logs, when set to level 5, are helpful for debugging network information (Rx and Tx), spinel data, and OTBR agent state.

The OT stack also allows for users to view RCP logs. To enable these:

1. OTBR must be built with the debug configs above.
2. In platform-abstraction/efr32/board_config.h, set RADIO_CONFIG_DEBUG_COUNTERS_SUPPORT to 1 to enable debug counters in radio.c.
3. Rebuild RCP with the following configuration: OPENTHREAD_CONFIG_LOG_OUTPUT:OPENTHREAD_CONFIG_LOG_OUTPUT_PLATFORM_DEFINED, OPENTHREAD_CONFIG_LOG_LEVEL:5, OPENTHREAD_FULL_LOGS_ENABLE:1.

Once these are followed, you can view any RCP logs in RTT or printed in the OTBR syslog (OPENTHREAD_CONFIG_LOG_OUTPUT_APP).

You should be able to see:

- Any errors surfaced on the RCP.
- Rx and Tx data, along with failures.
- Users can use otLogInfoPlat("") to print any data, such as efr32RadioCounters.

>Important note for RAIL tx counters / generally any RCP debug output: there is a limit to the amount of information that can be printed if OPENTHREAD_CONFIG_LOG_OUTPUT is set to OPENTHREAD_CONFIG_LOG_OUTPUT_APP. Printing too much information will result in the RCP being unresponsive, it will surface a NOMEM error in the syslog.

## Zigbee Host Application (Z3Gateway)

The Z3Gateway has no logs but there are prints in the console which may be helpful. For example, you can see the configuration data to ensure that zigbeed was configured properly (pay close attention to messages containing 'FAIL' while Z3Gateway configures Zigbeed).

You should also be able to see APS layer messages that are received and transmitted. If there are any issues with transmitting, a [tx 0x66](https://community.silabs.com/s/question/0D51M00007xeTcLSAU/what-cause-result-in-tx-66-error-print?language=en_US) error will be printed in the host console. You can also view the raw EZSP traces between Zigbeed and the host program, as well as any EZSP errors between zigbeed and Z3Gateway. APIs like `sl_zigbee_app_debug_print()` are useful for customer debug prints.

One of the more useful ways to debug with Z3Gateway is to use the CLI commands, most of which map to Zigbeed functions (so you can also implement these in Zigbeed). These commands can be found by typing 'help' in the Z3Gateway console. Inputting 'plugin' will print all of the plugins that you can use. 'counters', 'mfglib', and 'stack-diagnostics' can be really useful for debugging.

## Miscellaneous Network Debugging

Silicon Labs offers helpful debugging tools through Simplicity Studio. Some examples:

- Simplicity Commander: v1.17.4 - [User's Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/)
- Network Analyzer: Network Analyzer - v5.7.1 - Simplicity Studio 5 Users Guide Silicon Labs
- Energy Profiler: Energy Profiler - v5.7.1 - Simplicity Studio 5 Users Guide Silicon Labs
- Many sample apps such as Zigbee StandardizedRfTesting and RAIL SoC RAILtest
- Studio tools: Tools Overview - v5.7.1 - Simplicity Studio 5 Users Guide Silicon Labs

Refer to the release notes page to check for known bugs and workarounds.

## Common Issues

_CPC won't connect to the secondary_

Look over the CPC syslogs or state. This will output obvious mistakes such as mismatching settings or versions. If the CPC logs show the RCP is unresponsive, double-check the following:

- UART/SPI baud & settings for the secondary
- UART baud for the WSTK (if used), console -> admin -> 'serial vcom'
- A bootloader has been flashed to the RCP

Refer to [Common CPCD Issues & Debugging](https://community.silabs.com/s/article/Common-CPCD-Issues-Debugging?language=en_US) for more information.

_My OTBR agent won't start_

- Look over the syslogs for the OTBR agent
- Ensure the URL in /etc/default/otbr-agent is correct
- If using CPCd, make sure CPCd is connected before starting the service

_Tx timeout reported, 'RCP reset' messages after normal operation_

If these messages are printed the RCP has crashed or is stuck. Follow the "OTBR" section to capture what assert the RCP is hitting.

### Running the OTBR CLI Utility

```bash
 sudo ot-ctl
 ```

For a complete list of OTBR CLI commands, refer to the [OpenThread CLI Command Reference](https://openthread.io/reference/cli/commands).
