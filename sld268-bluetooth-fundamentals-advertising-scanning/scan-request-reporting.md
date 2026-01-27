# Using Scan Request Reporting

## Introduction

Together with the new BT5 advertising APIs introduced in SDK 2.3.x, a new `sl_bt_evt_advertiser_scan_request` event was introduced to notify the application that a scan request has been received. Scan requests (not to be confused with scan responses) can be received, when the device is in scannable advertising mode. Scan request reporting can be a useful feature to detect and react to the presence of scanner devices. For example, if a scanner device sends a scan request, the advertisement interval of the advertising device can be decreased to speed up the connection process.

## Usage

*sl_bt_advertiser_set_report_scan_request* API command enables scan request reporting. When enabled, a `sl_bt_evt_advertiser_scan_request` event will be generated every time a scan request is received.

The `sl_bt_evt_advertiser_scan_request` event informs the application about the Bluetooth address and address type as well as bonding status of the device that sent the scan request.
