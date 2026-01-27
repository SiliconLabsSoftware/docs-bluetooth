# Failure Analysis

The following table describes the different scenarios when returning a Series 2 or Series 3 device to Silicon Labs for failure analysis.

| **State** | **Secure Boot Disabled** | **Secure Boot Enabled (2)** |
|-----------|--------------------------|-----------------------------|
| Standard debug unlock | Device erase is not necessary for failure analysis. | Device erase is not necessary, but a correctly signed image is required to perform failure analysis. |
| Standard debug lock | Device erase is required to perform failure analysis. | Require device erase and correctly signed image to perform failure analysis. |
| Permanent debug lock | Cannot perform failure analysis. | Cannot perform failure analysis. |
| Secure debug lock (1) | Require debug unlock token to perform failure analysis. | Require debug unlock token and correctly signed image to perform failure analysis. |

**Notes**:

1. Follow the procedures in [Series 2 and Series 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/) section *Secure Debug Unlock and Roll Challenge - Simplicity Commander* to generate a valid debug unlock token for each device returned to Silicon Labs for failure analysis.

2. Secure boot enabled devices, especially with secure boot failure, may limit Silicon Labs' ability to determine the root cause of failure.
