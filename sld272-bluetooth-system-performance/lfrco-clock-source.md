# Using the LFRCO as Low-Frequency Clock Source

## Introduction

The Bluetooth specification sets the Sleep Clock Accuracy (SCA) to be +/- 500 ppm for the sleep clock used by a BLE device to keep BLE connection.  The EFR32xG22 devices include an internal `LFRC` oscillator that provides accuracy to within +/-500 ppm, which is suitable for BLE sleep applications and **eliminates the need for a 32.768 kHz crystal**. The LFRCO can be used by the Bluetooth stack as the low-frequency clock source (instead of the LFXO) to wake up the device on each connection interval when sleep is enabled in the stack.

Using the `LFRCO` is best suited for some BLE use cases, such as for applications with very constrained cost targets or board layout space, devices that maintain short connection intervals or have infrequent BLE connections, and devices that advertise most of the time (such as iBeacon or Google Eddystone devices).

The LFRCO can be set either with `precision` or `non-precision` mode. When precision mode is enabled, the LFRCO performs self recalibration periodically against the `38.4 MHz HFXO` crystal. Hardware detects temperature changes and initiates a re-calibration of the LFRCO as needed when operating in EM0, EM1, or EM2. If a re-calibration is necessary and the HFXO is not active, the precision mode hardware will automatically enable HFXO for a short time to perform the calibration. EM4 operation is not allowed when precision mode is enabled.

Note that, starting with SDK3.1.2, if LFRCO is used by the Bluetooth stack as the low-frequency clock source, the following applies:

- Using `non-precision` mode, EM2 is not supported if Bluetooth connection or periodic advertising feature is included by the app; Otherwise EM2 is allowed.
- Using `precision` mode, EM2 is supported in all Bluetooth cases.

## Enabling Precision Mode

You can enable precision mode in Simplicity Studio project configurator by going to **Services > Runtime > Device Initialization > Device Init:LFRCO** and change the setting from `Default precision` to `High precision` or by manually editing the `sl_device_init_lfrco_config.h` file.

Change

```C
#define SL_DEVICE_INIT_LFRCO_PRECISION          cmuPrecisionDefault
```
to

```C
#define SL_DEVICE_INIT_LFRCO_PRECISION          cmuPrecisionHigh
```

## Selecting the LFRCO

Using the LFRCO requires a few changes in **sl_device_init_clocks.c**. In `sl_device_init_clocks`, select LFRCO as a clock source for the low-frequency clocks.

```C
sl_status_t sl_device_init_clocks(void)
{
  CMU_ClockSelectSet(cmuClock_SYSCLK, cmuSelect_HFXO);
#if defined(CMU_EM01GRPACLKCTRL_MASK)
  CMU_ClockSelectSet(cmuClock_EM01GRPACLK, cmuSelect_HFXO);
#endif
#if defined(CMU_EM01GRPBCLKCTRL_MASK)
  CMU_ClockSelectSet(cmuClock_EM01GRPBCLK, cmuSelect_HFXO);
#endif
  CMU_ClockSelectSet(cmuClock_EM23GRPACLK, cmuSelect_LFRCO);
  CMU_ClockSelectSet(cmuClock_EM4GRPACLK, cmuSelect_LFRCO);
#if defined(RTCC_PRESENT)
  CMU_ClockSelectSet(cmuClock_RTCC, cmuSelect_LFRCO);
#endif
  CMU_ClockSelectSet(cmuClock_WDOG0, cmuSelect_LFRCO);
#if WDOG_COUNT > 1
  CMU_ClockSelectSet(cmuClock_WDOG1, cmuSelect_LFRCO);
#endif

  return SL_STATUS_OK;
}
```

## Tradeoffs

The most obvious benefit of using the LFRCO instead of the LFXO is the cost savings because you're not using the low-frequency crystal. The drawback is that the LFRCO increases the sleep current consumption and extends the RX receive window on the peripheral device side at the beginning of each connection interval.

## Sleep Current

Below are sleep current measurements on the same device with LFXO and precision mode LFRCO (PLFRCO) where the sleep current difference is about ~280 nA.

![LFXO Sleep Current](resources/lfxo-sleep-current.png?darkModeUrl=resources/lfxo-sleep-current.png)

![PLFRCO Sleep Current](resources/plfrco-sleep-current.png?darkModeUrl=resources/plfrco-sleep-current.png)

## RX Period Extension

At the beginning of each connection interval the central device is the first to send out a packet. As a result, the peripheral device must be listening (in RX) to avoid losing the initial packet from the central device. The combined clock accuracy from client and peripheral device (sum of both accuracy values in ppm) is used to calculate when the peripheral device should wake-up to listen for the incoming packet.

When the accuracy is lower, the peripheral device must wake-up earlier. As a result, the RX receive window is longer when using a PLFRCO compared to the LFXO. The images below show an empty packet taken from the peripheral device side with 1 second connection interval and 0 dBm output power. When using PLFRCO, the RX receive window is extended by ~250 µs. The longer the connection intervals, the more pronounced the RX window extension compared to the LFXO usage.

![LFXO Empty Packet](resources/empty-packet-lfxo.png?darkModeUrl=resources/empty-packet-lfxo.png)

![PLFRCO Empty Packet](resources/empty-packet-plfrco.png?darkModeUrl=resources/empty-packet-plfrco.png)

Note that when the peripheral device misses connection intervals (for example, if the device was temporarily out of range or because of interference), the RX receive window is widened by the combined accuracy until the peripheral device is able to catch the initial packet from the client. Consider a combined accuracy of +/-600 ppm. If the connection interval is 1 s and the peripheral device misses a connection interval, the next interval's RX receive window will be widened by 600 µs = 600 parts of 1 million micro-seconds.
