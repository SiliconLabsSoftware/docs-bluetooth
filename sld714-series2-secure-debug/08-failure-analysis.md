# Failure Analysis

The following table describes the different scenarios when returning a Series 2 device to Silicon Labs for failure analysis.

| **State** | **Secure Boot Disabled** | **Secure Boot Enabled (2)** |
|-----------|--------------------------|-----------------------------|
| Standard debug unlock | Device erase is not necessary for failure analysis. | Device erase is not necessary, but a correctly signed image is required to perform failure analysis. |
| Standard debug lock | Device erase is required to perform failure analysis. | Require device erase and correctly signed image to perform failure analysis. |
| Permanent debug lock | Cannot perform failure analysis. | Cannot perform failure analysis. |
| Secure debug lock (1) | Require debug unlock token to perform failure analysis. | Require debug unlock token and correctly signed image to perform failure analysis. |

**Notes**:

1. Follow the procedures mentioned in [Debug Access Flow](./05-debug-unlock.md#debug-access-flow) to generate a valid debug unlock token for each device returned to Silicon Labs for failure analysis.

2. Secure boot enabled devices, especially with secure boot failure, may limit Silicon Labs' ability to determine the root cause of failure.
