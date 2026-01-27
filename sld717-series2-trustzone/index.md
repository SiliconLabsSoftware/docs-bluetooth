# Series 2 TrustZone

> **Note: This section replaces *AN1374: TrustZone*. Further updates to this user guide will be provided here**.

ARMv8-M TrustZone is a technology that provides a foundation for improved system security in embedded applications. It allows the ARMv8-M to be aware of the security states of the system. Series 2 devices use the Cortex-M33 core to implement the ARMv8-M TrustZone security extension, which provides the ability to restrict access to peripherals and memory regions based on the processor security attribute. TrustZone works with the MPU, which controls privileged/unprivileged execution of code to provide a complete security solution.

ARMv8-M TrustZone is an extensive topic. The references below are publicly available on the [ARM Developer Documentation](https://developer.arm.com/docs) website.

- [ARMv8-M Architecture Reference Manual](https://developer.arm.com/documentation/ddi0553/latest)
- [ARMv8-M Architecture Technical Overview](https://community.arm.com/cfs-file/__key/telligent-evolution-components-attachments/01-2142-00-00-00-00-66-90/Whitepaper-_2D00_-ARMv8_2D00_M-Architecture-Technical-Overview.pdf)
- [ARM Cortex-M33 Processor Technical Reference Manual](https://developer.arm.com/documentation/100230/latest)
- [System Design with ARMv8-M](https://developer.arm.com/documentation/100767/0100/System-Design-for-ARMv8-M)
- [TrustZone technology for ARMv8-M Architecture](https://developer.arm.com/documentation/100690/latest/)
- [ARM Cortex-M33 Devices Generic User Guide](https://developer.arm.com/documentation/100235/latest)
- [Secure software guidelines for ARMv8-M](https://developer.arm.com/documentation/100720/0300)
- [Software Development in ARMv8-M Architecture](https://community.arm.com/cfs-file/__key/telligent-evolution-components-attachments/01-2142-00-00-00-01-27-19/ARM-Cortex-_2D00_-session-11-_2D00_-Yiu-_2D00_-Software-Development-in-ARMv8_2D00_M-Architecture.pdf)
Reading guides:  
- Beginner
- Minimal experience with TrustZone, starting with [TrustZone Basics](./02-r-basic.md)
- Intermediate - Have a basic understanding of the TrustZone technology, starting with [Bus Level Security](./03-r-bls.md)
- Advanced - Developed experience on TrustZone, starting with [TrustZone Implementation](./05-r-implementation.md)
-Demo - Starting with [TrustZone Platform Examples](./05-r-implementation.md)

## Key Points

- TrustZone Basics
- Bus Level Security (BLS)
- Secure and Privileged Programming Model
- TrustZone Implementation
- Upgrade Existing Application to TrustZone
- TrustZone Platform Examples
