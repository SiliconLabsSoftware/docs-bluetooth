# Bluetooth LE Electronic Shelf Label

**This section replaces *AN1419: Bluetooth LE Electronic Shelf Label*. Further updates to this application note will be provided here.**

The Bluetooth Special Interest Group released the Electronic Shelf Label (ESL) profile and service specification at the beginning of 2023. With the essential additions in the Bluetooth Core Specification version 5.4 (e.g., Periodic Advertising with Responses and Encrypted Advertising Data), the ESL specification defines a standardized use of Bluetooth Low Energy (BLE) in electronic shelf labels.

This application note introduces the Bluetooth - SoC ESL Tag example project, accompanied by an Access Point host application through examples, which describes the essential features of the BLE ESL. In addition, this document also describes how the ESL network can be configured, through multiple examples. And finally, the document also describes the first steps for how to modify the sample application, extending it to use an additional, or different, display, and using non-volatile memory for storing images.

[Bluetooth ESL](./04-bluetooth-esl) provides a detailed introduction to BLE Electronic Shelf Label (ESL) Tags.

[Preparing the ESL Network](./05-preparing-the-esl-network) provides the steps to prepare the ESL access point and how to build the **Bluetooth – SoC ESL Tag** example project and program it to the target device.

[The ESL Network](./06-the-esl-network) describes how to run the ESL network, both in automatic and manual mode. The section, [ESL AP in Manual Mode](./06-the-esl-network#esl-ap-in-manual-mode), focuses on the different states an ESL Tag can be in, and how to do the transitions between these with the access point implementation. All essential commands are introduced, most of them with an example.

[Modifying the Bluetooth ESL example](./07-modifying-the-bluetooth-esl-examples) provides information on how to extend the example code with an additional display and the relevant configurations and API calls.

[Demo](./08-demo) shows how he access point can be controlled with a Simplicity Connect mobile application.

[Supported Radio Boards](./10-supported-radio-boards) is an appendix of all supported radio boards.
