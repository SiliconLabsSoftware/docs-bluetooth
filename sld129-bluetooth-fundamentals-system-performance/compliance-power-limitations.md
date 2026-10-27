# TX Power Limitations for Regulatory Compliance (ETSI, FCC)

## Introduction

Local regulatory bodies put limitations on how much power a radio equipment is allowed to radiate. This document discusses the rules applied by two main regulatory bodies (ETSI and FCC) and presents how Silicon Labs' Bluetooth stack limits TX power to comply with these regulations.

> Note that TX power is also limited by the radio chip. Different power limits apply for different parts. Check the data sheet of your device for more information. The power limits discussed here are the absolute maximums.

## Without Adaptive Frequency-Hopping (AFH)

### ETSI EN 300 328

It is generally true that ETSI EN 300 328 allows 20 dBm RF output power when equipment is using wide band modulations other than FHSS (Frequency Hopping Spread Spectrum). In these cases, PSD (Power Spectral Density) also must be tested, which allows 10 dBm / 1 MHz.
These restrictions apply to those BLE devices where adaptive frequency hopping is not enabled. For 125 kbps, 500 kbps and 1 Mbps PHY (~1 MHz bandwidth), which means that the maximum radiated power allowed is 10 dBm. For 2 Mbps PHY, it is a few tenths dB more.

### FCC 15.247

Based on FCC part 15.247 for wideband digital modulation, the output power can go up to 30 dBm and the power spectral density must be below 8 dBm / 3 KHz.
For 500 kbps (coded), 1 Mbps and 2 Mbps PHY, 30 dBm limitation is applied.
For 125 Kbps coded PHY, the device doesn’t pass the 8 dBm/ 3 KHz PSD limit with full power. As a result, the maximum output power allowed is 14 dBm.

### Summary

Because Bluetooth stack follows strict regulations, the maximum output power is 10 dBm in those cases when AFH is not enabled (or AFH is enabled but no more than 15 channels are available). This stack limitation (10 dBm EIRP) is valid for all PHYs and devices.

| Power Limits Without AFH |     EIRP [dBm]     |         EIRP [dBm]         |
| :----------------------: | :----------------: | :------------------------: |
|                          | 125 kbps coded PHY | 500 kbps, 1 Mbps and 2 Mbps |
|           FCC            |         14         |              30            |
|           ETSI           |         10         |              10            |
|    Supported by stack    |         10         |              10            |

## With Adaptive Frequency-Hopping (AFH)

### ETSI EN 300 328

When adaptive frequency hopping is allowed (and at least 15 channels is available), the only limitation is maximum 20 dBm EIRP. There are no restrictions for PSD.

### FCC 15.247

If AFH is used, the maximum output power, which is allowed by FCC, can be 30 dBm and there are no PSD limitations. FCC contains, however, a restricted band from 2483.5 MHz to 2500 MHz. As a result, in several cases power limitation is needed on the edge channels.

### Summary

Considering regulations of FCC and ETSI, when AFH is applied and at least 15 channels are available, the maximum conducted output power, which is allowed by BLE stack, is 20 dBm on all channels except of on channel 37 and 38 (physical channels not logical channels). The output power is limited to 18 dBm on channel 37 and 15.3 dBm on channel 38 in the case of all PHYs. There isn’t any limitation on channel 39 because the upper channel is only used for advertisements, so with the low duty cycle correction advertisements can be sent at full power.

| Power Limits With AFH | EIRP [dBm] | EIRP [dBm] |     EIRP [dBm]     |
| :-------------------: | :--------: | :--------: | :----------------: |
|                       | channel 37 | channel 38 | all other channels |
|          FCC          |     18     |    15.3    |         30         |
|          ETSI         |     20     |     20     |         20         |
|   Supported by stack  |     18     |    15.3    |         20         |

## Additional Restrictions

On Series 1 devices (EFR32xG1x), the TX power is always limited to 14 dBm when Coded PHY is used.
