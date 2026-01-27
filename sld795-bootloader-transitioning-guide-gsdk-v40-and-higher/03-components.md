# Components

This section maps the various Gecko Bootloader components to the previously available plugins (in the AppBuilder workflow). Although the basic subgroups might remain the same, specific component locations are subject to change across various GSDK versions. Therefore, it is good practice to search for individual components by their names rather than the specific subgroups that they belong to.

| AppBuilder Plugin | Simplicity Studio v5 Component | Additional Comments |
|-------------------|--------------------------------|---------------------|
| Communication |  |  |
| BGAPI UART DFU | BGAPI UART DFU |  |
| EZSP-SPI | EZSP-SPI |  |
| UART XMODEM | UART XMODEM |  |
| XMODEM Parser | XMODEM Parser |  |
| Core |  |  |
| Application Upgrade Version Check | Application Upgrade Version Check |  |
| Bootloader Core | Bootloader Core |  |
| GBL Compression (LZ4) | GBL Compression (LZ4) |  |
| GBL Compression (LZMA) | GBL Compression (LZMA) |  |
| Image Parser | Image Parser |  |
| Image Parser with legacy EBL support | Image Parser with legacy EBL support |  |
| Image Parser without encryption support | Image Parser without encryption support |  |
| “ | Bootloader Include Parser | New component that includes common Image Parser header files |
| “ | Bootloader in Main Flash | New component that enables the user to place bootloader in main flash for xG13 and xG14 devices |
| Drivers |  |  |
| Delay | Bootloader Delay Driver |  |
| SPI Master | Bootloader SPI Controller USART Driver |  |
| SPI Slave | Bootloader SPI Peripheral USART Driver |  |
| UART | Bootloader UART Driver |  |
| EUART | Bootloader EUART Driver |  |
| “ | Bootloader SPI Controller EUSART Driver | New component supporting SPI EUSART driver. This component can be used with devices supporting EUSART interface. |
| “ | Bootloader SPI Peripheral EUSART Driver | New component supporting SPI EUSART driver. This component can be used with devices supporting EUSART interface. |
| Storage |  |  |
| Common Storage | Common Storage |  |
| Common Storage (single storage slot only) | Common Storage (single storage slot only) |  |
| Internal Storage | Internal Storage |  |
| SPI Flash Storage | SPI Flash Storage |  |
| “ | Bootloader Storage Slot Setup | New component that facilitates configuring storage slots for Internal and SPI Flash based bootloaders |
| Utils |  |  |
| Crypto | Crypto |  |
| Cyclic Redundancy Check | Cyclic Redundancy Check |  |
| Debug | Debug |  |
| EMLIB | EMLIB Peripheral HAL | This component is no longer part of the Bootloader module. EMLIB component(s) can be found under Platform \> Peripheral |
| EZSP GPIO Activation | EZSP GPIO Activation |  |
| GPIO Activation | GPIO Activation |  |
| SE Manager | SE Manager | This component is now available under Platform \> Security. This component is only applicable for Series 2 devices. |
| Token Management | Token Management |  |
| mbed TLS | Mbed TLS … | These components are available under Platform \> Security |
| Bootloader Interface |  |  |
| | App Properties | A new component to configure application properties (version), which is used during creation of a GBL file using Simplicity Commander. |
| Bootloader-interface | Bootloader Application Interface |  |
