# Typical Configuration

Several of the available [tamper sources](./05-tamper-sources.md) report internal HSE errors. A number of these sources are configured to reset the device (level 4) by default. Custom handling of internal and external tamper sources (default level 0) can be configured to trigger an interrupt (level 1) on the Cortex-M33 or trigger an interrupt and increase a counter in the tamper filter (level 2) as in the following figure for EFR32xG21B devices.

![Custom Handling of Tamper Sources (EFR32xG21B Devices)](./resources/sld715-custom-handling-tamper-sources.png)

> **Note**: The actions for level 1 on the right side are implemented by the tamper interrupt handler.

## Typical Configuration Highlights

- The response of the TRNG monitor depends on the failure rate due to lack of entropy.

- The voltage and digital glitch detectors can see spurious activations. They should typically not be used to drive a high-level tamper response directly. Instead, they should feed their signals into a tamper interrupt, which activates a high-level action (e.g., Reset in this example) through PRS tamper if a certain number of detections (noise filter) occur in a short time window.

- The operating conditions decide the time out of the specification filter for the temperature sensor. For some systems, any time out of specification should trigger a reset.

- Mailbox authorization is handled similarly for voltage and digital glitch detectors.

- A PRS tamper implements a high-level response for a tamper interrupt, which issues a tamper reset (level 4) to prevent or slow further attacks.

- In extreme cases, if the system identifies an attack with high confidence, a PRS tamper can be configured as [Erase OTP](./04-tamper-responses.md#erase-otp) (level 7) to brick the part and prevent further attacks. This is recommended only when the destruction of parts is acceptable and where high confidence of an attack can be achieved.

- Another PRS tamper detects enclosure opening from GPIO. This source feeds into the tamper filter counter (level 2), which will trigger an interrupt cumulative effect and activate a `Filter counter` number 1 response (Reset in this example) if the filter counter reaches the trigger threshold within the filter reset period. This filter counter response approach is less flexible than the interrupt response approach since the trigger threshold and filter reset period are one-time programmable.
