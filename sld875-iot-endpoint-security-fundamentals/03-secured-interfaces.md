# Secured Interfaces

*All product interfaces shall be appropriately secured by the manufacturer.*

The interfaces to be secured will vary by product configuration. For example, in an NCP topology the NCP interface must be secured. Debug interfaces should always be locked. Wireless interfaces should be secured by using strong pairing and commissioning methods and by enabling encrypted and authenticated transmissions.

While securing the interfaces is in the end your responsibility, Silicon Labs provides the tools to enable that security.

Series 1, Series 2 and Series 3 devices are designed to support securing debug access. For Series 1 devices, that functionality is provided through writing a Debug Lock word to the device. Unlocking the device erases the main application and the key material stored in the Lockbits page. For Series 2 and 3 devices, securing debug access is done through the device’s Secure Engine. Both allow the developer to lock the debug port itself. See [Silicon Labs Gecko Bootloader User's Guide for GSDK 4.0 and Higher (series 1 and 2 devices)](https://docs.silabs.com/mcu-bootloader/latest/bootloader-user-guide-gsdk-4/) or *UG266: Silicon Labs Gecko Bootloader User’s Guide for GSDK 3.2 and Lower* for an overview of securing debug access, and [Series 2 and 3 Secure Debug](https://docs.silabs.com/iot-security/latest/series2-secure-debug/) for details on the Series 2 and 3 implementation. [Testing and Debugging Applications for the Silicon Labs EFR32MG Platforms](https://docs.silabs.com/zigbee/latest/test-debug-apps-efr32mg/) provides an overview of the various application testing stages and the debug access (hardware and software) required in each.

For more information on Wireless interface security in the different protocols, see the following:

- [Zigbee Security](https://docs.silabs.com/zigbee/latest/zigbee-security/)

- [Bluetooth LE Fundamentals](https://docs.silabs.com/bluetooth/latest/bluetooth-le-fundamentals/)

- [UG235.03: Architecture of the Silicon Labs Connect Stack v2.x](https://www.silabs.com/documents/public/user-guides/ug235-03-architecture-of-connect.pdf)

- [Architecture of the Silicon Labs Connect Stack v3.x](https://docs.silabs.com/connect-stack/latest/network-co-processor-applications-connect-v3x/)
