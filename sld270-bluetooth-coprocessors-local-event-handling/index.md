# Local Event Handling on Bluetooth NCP

## Introduction

A network co-processor (NCP) firmware can handle stack events locally instead of sending every event to the host processor, which allows event pre-processing, useful for some use cases, to prevent overflowing the host with events.

## Background

Ordinarily, an NCP device sends and receives Bluetooth packets on behalf of a host processor but does not have any application layer intelligence. It receives commands from and sends events to the host and the host must tell the NCP what to do.

However, in some cases, it may be useful for the NCP to pre-process some of the events itself rather than to send every event to the host. For example, when the NCP acts as a scanner in a high-traffic environment, it reports every single advertisement to the host, which can easily be overwhelming for the host to process.

## Implementation

This section explains how to modify the standard NCP firmware and assumes that the user is already familiar with the NCP by reading NCP-related documentation, for example [Using the Bluetooth Stack in Network Co-Processor Mode](/bluetooth/{build-docspace-version}/bluetooth-network-coprocessor-mode).

By default, the NCP firmware passes all events to the host processor. However, this can be controlled by an implementation of `sl_ncp_local_evt_process()`. This function can be implemented in any source file of the firmware, e.g., app.c. This function implementation will override the default implementation that performs no handling of events and returns *true* to indicate that the event passed to it has not been handled and should be sent to the host. If the function returns *false*, the event will **not be sent** to the host device.

To handle a particular event locally, this function can be customized. In this case, let’s assume that the desired behavior is to filter out any advertisements or scan responses that do not come from a device with OUI 00:0B:57 (which is an OUI of Silicon Labs) and **not** to send them to the host. This can be accomplished by implementing a switch case statement in the `sl_ncp_local_evt_process()` function that captures the *scanner_scan_report* event and checks the OUI.

```c
/**************************************************************************//**
 * Local event processor.
 *
 * Use this function to process Bluetooth stack events locally on NCP.
 *
 * @param[in] evt Event coming from the Bluetooth stack.
 *
 * @return true, if the event should be forwarded to the NCP-host.
 *         false, if the event has been handled locally.
 *
 * @note This overrides the default weak implementation.
 *****************************************************************************/
bool sl_ncp_local_evt_process(sl_bt_msg_t *evt)
{
  bool evt_to_host = true;
  bd_addr address;
  switch (SL_BT_MSG_ID(evt->header)) {

    case sl_bt_evt_scanner_scan_report_id:
      address = evt->data.evt_scanner_scan_report.address;
      if(address.addr[5] == 0x00 && address.addr[4] == 0x0B && address.addr[3] == 0x57) {
        // Scanned device has OUI 00:0B:57 so the event shall be sent to the host
        evt_to_host = true;
      }
      else {
        // Scanned device does *not* have OUI 00:0B:57 so the event shall *not* be sent to the host
        evt_to_host = false;
      }
      break;

    default:
      break;
  }
  return evt_to_host;
}
```

This section of code checks whether the first three bytes of the Bluetooth address, which come in reverse order, match the desired OUI. If that's the case, *evt_to_host* is set **true** and the event **will be sent** to the host. Otherwise, *evt_to_host* is set to **false** indicating that the event will **not be sent** to the host. This is a small change but it could dramatically reduce the amount of traffic between NCP and host as well as reducing the burden on the host itself.

In this way the `sl_ncp_local_evt_process()` function can be implemented to perform any local event handling.
