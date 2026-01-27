# Overview

The HSE-SVH and Series 3 Secure Vault Anti-Tamper module is used to hamper or prevent both reverse engineering and re-engineering of proprietary software systems or applications.

Tamper attacks come from one or more vectors. Common attacks include voltage glitching, magnetic interference, and forced temperature adjustment. The HSE-SVH and Series 3 Secure Vault Anti-Tamper module provides fast hardware detection of external tamper signals such as case opening, glitching, and logical attacks allowing analysis and escalation up to and including bricking the device.

The anti-tamper module connects a number of hardware and software-driven tamper signals to a set of configurable hardware and software responses. This can be used to program the device to automatically respond to external events that could signal that someone is trying to tamper with the device, and very rapidly remove secrets stored in the HSE.

The available tamper signals range from signals based on failed authentication and secure boot to specialized glitch detectors. When any of these signals fire, the tamper block can be configured to trigger several different responses, ranging from triggering an interrupt to erasing the One-Time-Programmable (OTP) memory, removing all HSE secrets and resulting in a permanently destroyed device.

Silicon Labs provides [Custom Part Manufacturing Service (CPMS)](https://www.silabs.com/developers/custom-part-manufacturing-service) to protect the users' privacy by configuring the most effective tamper detection features at the Silicon Labs factory. For more information about CPMS, see [Custom Part Manufacturing Service User's Guide](https://docs.silabs.com/iot-security/latest/iot-security-cpms/).

Some SVM devices (e.g. xG25A and xG27) and SVH devices (e.g. xG25B) and Series 3 Secure Vault devices (e.g. xG301) feature an External Tamper Detect module which is used to detect signals such as case opening. The ETAMPDET signal on SVH and Series 3  Secure Vault devices is routed to the SE as an Anti-Tamper module tamper source, in addition to being a stand-alone module. For more information about ETAMPDET operation, refer to the device reference manual. Examples demonstrating how to use ETAMPDET can be found on the [Silicon Labs peripheral_example github repository](https://github.com/SiliconLabs/peripheral_examples/tree/master/series2/etampdet). The peripheral reflex system (PRS) can also be routed as a tamper source. For more information on PRS, refer to your device’s reference manual.
