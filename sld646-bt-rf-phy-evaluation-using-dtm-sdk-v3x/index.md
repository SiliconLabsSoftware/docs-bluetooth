# Radio Frequency Physical Layer Evaluation in Bluetooth SDK v3.x and Higher

**Note: This section replaces *AN1267: Radio Frequency Physical Layer Evaluation in Bluetooth SDK v3.x and Higher*. Further updates to this application note will be provided here.**

This section provides an overview of how to perform Bluetooth-based radio frequency (RF) physical layer (PHY) evaluation with Bluetooth-enabled EFR32xG system-on-chips (SoCs) and BGM/MGM modules using Silicon Labs software tools and dedicated firmware. A Bluetooth device’s RF parameters are validated using a protocol called Direct Test Mode (DTM). DTM is described in the Bluetooth Core Specification versions 5.x, Volume 6, Part F. Testing can be performed in three ways:

- Through DTM test commands issued by a host system over the SoC’s or module’s host interface.
- Through DTM test commands issued by a custom application running in the SoC or module itself. Such commands could be autonomously launched by the custom application, for example at boot or after a remote companion device writes a dedicated GATT characteristic over a Bluetooth LE connection.
- In a special test environment supported by dedicated firmware that allows a testing device to control the test target.

With these options, customers can fully evaluate transmit and receive performance, and test the RF functionality of their development kit hardware or custom hardware. Laboratory tests require RF test equipment, such as a spectrum analyzer and an RF signal generator, and/or a Bluetooth tester.
