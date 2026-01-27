# Main Bootloader in Main Flash

For xG13 and xG14 devices, the entire main stage bootloader might not fit into the bootloader flash if the user installs some extra components. In such scenarios, the main stage bootloader can be placed in the main flash by installing the **Bootloader in Main Flash** core component in the bootloader project.

![image18](./resources/sld795-image18.png)

For more information on this component, see [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/).
