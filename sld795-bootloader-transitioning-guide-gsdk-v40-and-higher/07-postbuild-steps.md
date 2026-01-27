# Postbuild Steps

For Series 1 devices, the main stage bootloader needs to be combined with the first stage bootloader, which results in a combined bootloader binary. To accomplish this, a postbuild script is made available with the GSDK, which combines the appropriate first stage bootloader with the main stage bootloader to produce a combined bootloader binary. A postbuild step must be explicitly added through the Project’s settings so that the postbuild script can be run after the bootloader build is complete. For more details, see [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/).

>**Note**: Adding the postbuild steps is mandatory when building bootloader binaries for Series 1 devices. If the postbuild step is not configured correctly, Simplicity Studio cannot build a combined bootloader binary image.
