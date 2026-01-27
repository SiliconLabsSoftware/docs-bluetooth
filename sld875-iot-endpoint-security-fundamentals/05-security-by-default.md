# Security by Default

*Product security shall be appropriately enabled by default by the manufacturer.*

The state in which a product is shipped is up to the manufacturer. This standard mandates that any security features provided with the product be enabled before shipping. Customers should not have to turn security on; rather they should actively have to disable it. For example, Silicon Labs Z-Wave end-nodes and gateway SDKs ship with S2 cryptography and SmartStart network formation enabled by default.

Silicon Labs believes that product security should be considered during product design, and not as an afterthought. Within development environments, all Silicon Labs application security features may be enabled or disabled as appropriate during application development. Security must also be considered during device design and testing. [Bringing Up Custom Devices for the EFR32MG and EFR32FG Families](https://docs.silabs.com/connect-stack/latest/custom-nodes-efr32/) describes the security tokens (keys, certificates, and so on) that can be programmed into a custom device to support various types of security, including that provided by the Gecko Bootloader. See [Signed Software Updates](./06-signed-software-updates.md).
