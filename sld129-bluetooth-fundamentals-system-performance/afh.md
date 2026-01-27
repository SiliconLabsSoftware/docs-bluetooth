
# Adaptive Frequency Hopping

## Introduction

Bluetooth functions in the 2.4 GHz frequency range. Because this is an ISM band, it's highly likely to have interference with other devices working in the same frequency band. The Bluetooth standard makes it possible for the communicating devices to agree on which channels to use from the 37 available data channels during communication. If the central device detects high interference on a channel, it can initiate a channel map update. Adaptive frequency hopping (AFH) means that the communicating devices are continuously monitoring their environment for interference and are continuously changing the channel map to address the interference.

> Note: AFH is a requirement for using TX power over +10 dBm. See [Bluetooth TX power settings](./tx-power).

## Non-Adaptive Channel Blocking

Channel map update procedure is part of the Bluetooth 4.0 specification. This procedure makes it possible for the peer devices to agree on which channels they will use from the available 37 data channels during communication. Channel map update procedure can be initiated by the central device only.

This procedure is supported in all SDK versions because it's part of the Bluetooth 4.0 standard. For example, if a smartphone connects to an EFR device, the smartphone, which is a central, can disable some channels and the EFR will accept it. This feature is very often used to disable the Bluetooth channels that overlap with the Wi-Fi channels used by the smartphone.

However, when the EFR is the central, the EFR will enable all channels by default, without any restriction, and they can be disabled only manually. See section [Manual Channel Blocking](#manual-channel-blocking). Adaptive Frequency Hopping was introduced to overcome this issue and to block low-quality channels dynamically.

## Interference Detection

Starting with Bluetooth SDK v3.x, Adaptive Frequency Hopping feature can be enabled by installing the `AFH (Adaptive Frequency Hopping)` software component to your project. When this component is installed, the `sl_bt_init_afh()` function is automatically called inside `sl_bt_init()` by the code generator. This process initializes and and enables adaptive frequency hopping without needing to call `sl_bt_init_afh()` in the application directly.

If AFH is enabled and at least one advertiser is running with extended advertisements or at least one connection is active, the Bluetooth stack runs a periodic background task that sweeps all channels once every `afh_scan_interval` and measures received power on each channel. If the measured power is beyond limit (-71 dBm) on a channel, the channel is **blocked** for at least 8 `afh_scan_interval`s. (If interference is still found on the channel, the blocking is prolonged, i.e., unblocking of a channel needs 8 consecutive measurements with no interference detected.)

After each sweep, a new **channel map** is created based on the interference measurements. If the channel map has changed, the **central device** sends the new channel map to all slaves, using the channel map update process. Slaves cannot initiate channel map update.

`ah_scan_interval` is set to 1 second by default and the sweep of the 40 channels takes around 10 ms, which means an additive energy consumption of ca. 240 uW.

> Note: The AFH sweep has the highest priority among all Bluetooth operations. Therefore, if there are multiple operations within 10 ms, sweeping cannot be fitted between them and they may be blocked.

![Additive energy consumption](resources/afh-scan-energy.png?darkModeUrl=resources/afh-scan-energy.png)

## Changing Scan Interval

`afh_scan_interval` is set to one second by default, that is, one channel sweep is made every second. The `afh_scan_interval` can be changed by calling the following function:

```C
#define AFH_SCAN_INTERVAL_CONFIG_KEY 7

uint8_t afh_scan_interval = 10; // set afh scan interval to 100ms

sl_bt_system_linklayer_configure(AFH_SCAN_INTERVAL_CONFIG_KEY,sizeof(uint8_t),&afh_scan_interval);
```

The unit of `afh_scan_interval` is 10 ms.

Channel sweep is always done after a radio operation, e.g., after an advertisement is sent. Setting the scan interval e.g., to 100 ms does not mean that the sweep will be done every 100 ms but that after every radio operation the Bluetooth stack checks if at least 100 ms has passed since the last sweep. Let's say that `afh_scan_interval` is set to 100 ms and advertising is done with 80 ms advertisement interval. In this case, the scan is done after every second advertisement, i.e., once every 160 ms.

## Manual Channel Blocking

Channels can also be blocked manually using the following command: `sl_bt_gap_set_data_channel_classification()`. For details, see the stack API documentation. If a channel is blocked manually, that channel will be never used until it is unblocked manually. In other words, the channels blocked by interference detection and channels blocked manually will be merged into a common channel map.

## Transmit Power

You are allowed to use TX power above +10 dBm when AFH is enabled and at least 15 channels are available. See [Bluetooth TX power settings](/bluetooth/{build-docspace-version}/bluetooth-system-performance/tx-power). However, note that high transmit power is only allowed once for each channel after a measurement on that channel occurs. In other words, if you use the same channel multiple times for transmitting within `afh_scan_interval`, the second and consecutive transmission will use +10 dBm. If you have a short connection interval and long afh_scan_interval, this can easily happen. See the following figure:

![Energy profiler](resources/afh-energy-profiler.png?darkModeUrl=resources/afh-energy-profiler.png)

## Peripheral Adaptivity

To be ETSI compliant, peripheral devices need to be adaptive as well. The central device might be non-adaptive, so the peripheral cannot blindly follow the transmissions of the central. ETSI allows using *control* transfer on blocked channels with +10 dBm or lower, but not *data* transfer. For ETSI compliance reasons, if the peripheral detects that a blocked channel is used, it will only send a single empty/control packet on that channel to prevent supervision timeout.

## Frequency Hopping Algorithm

Bluetooth Low Energy has two channel selection algorithms. Algorithm #1 is the original and mandatory. Algorithm #2 was introduced with Bluetooth 5.0. The hopping algorithm used is chosen by the central at connection time. Algorithm #2 has more randomness so it is always used unless connecting to a legacy device.

- Algorithm #1: Channels are selected sequentially using fixed hop interval passed while establishing connection.
- Algorithm #2: The channel to be used is calculated from the current event counter and access address. The selected channel appears to be semi-random.

If a channel to be used is blocked on the channel map, the channel is remapped to an available channel using a remapping algorithm.

## Other Limitations

When AFH is applied, the length of the connection events (not to be confused with the connection interval) is limited to 40 ms. In other words, in every connection interval you can send packets only for 40 ms. This is usually not a problem because it takes around 2.5 ms to transmit a packet with 251B payload. However, to achieve maximum throughput with unacknowledged data transmission (see [Throughput with Bluetooth Low Energy](./throughput)), you have to take into account this limitation. For example, if you have 100 ms connection interval, you can send packets only 40% of the time. To achieve maximum throughput, decrease your connection interval to 40 ms or lower.
