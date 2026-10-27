# Tests Enabled in the DTM Framework

DTM enables a set of RF PHY test cases, which are defined by the Bluetooth Special Interest Group (SIG) in the documents from the sections called "TCRL Release Table" and "Core - Test Requirements" found at the beginning of the [Bluetooth Specifications](https://www.bluetooth.com/specifications/specs/) web page.

The Capability tests (as defined in the standard ISO subgroups) are organized in levels and groups representing protocol services, functional modules, and purposes, the latter being divided in operating conditions for the transmitter and the receiver. All the relevant RF PHY tests are in accordance to the test specifications RF-PHY.TS.5.1.1 or the updated RF-PHY.TS.p15 and are shown in the RF-P sheet of the Excel file called *Core.TCRL.2019-2.xlsx*  found inside the *2019-2 TCRLs_2020-01-15_HFP1.8.zip*.

Below are examples of test cases for the Physical Layer Conformance. They are referred to by their identifiers, where RF-PHY stands for *RF-PHY Test Purpose* and TRM and RCV stand for Transmitter and Receiver test respectively.

- RF-PHY/TRM/BV-01-C [Output power]

- RF-PHY/TRM/BV-03-C [In-band emissions, uncoded data at 1 Ms/s]

- RF-PHY/TRM/BV-05-C [Modulation Characteristics, uncoded data at 1 Ms/s]

- RF-PHY/TRM/BV-06-C [Carrier frequency offset and drift, uncoded data at 1 Ms/s]

- RF-PHY/TRM/BV-08-C [In-band emissions at 2 Ms/s]

- RF-PHY/TRM/BV-10-C [Modulation Characteristics at 2 Ms/s]

- RF-PHY/TRM/BV-13-C [Modulation Characteristics, LE Coded (S=8)]

- RF-PHY/RCV/BV-01-C [Receiver sensitivity, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-03-C [C/I and Receiver Selectivity Performance, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-04-C [Blocking Performance, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-05-C [Intermodulation Performance, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-06-C [Maximum input signal level, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-07-C [PER Report Integrity, uncoded data at 1 Ms/s]

- RF-PHY/RCV/BV-08-C [Receiver sensitivity at 2 Ms/s]

- RF-PHY/RCV/BV-10-C [Blocking performance at 2 Ms/s]

- RF-PHY/RCV/BV-27-C [Receiver sensitivity, LE Coded (S=8)]

All the above tests can be performed with the EFR32xG SoCs and the BGM/MGM modules, because they are supported by the DTM implementation built into the Silicon Labs Bluetooth stack.

> **Note**: To accomplish the full set of tests in the specification, up to two external signal generators are needed to provide complete interference signals, in addition to a spectrum analyzer.
