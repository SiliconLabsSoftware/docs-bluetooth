# Using NVM3 with Zigbee EmberZNet v7.x and Higher

This chapter applies to the Silicon Labs Zigbee EmberZNet SDK v7.x and higher used with the Simplicity Project Configurator in Simplicity Studio 5. Zigbee EmberZNet 6.x and lower use AppBuilder. AppBuilder use is described in [Using NVM3 in AppBuilder-Based Applications](./08-using-nvm3-in-appbuilder-based-applications.md).

All Zigbee sample applications in the GSDK by default are configured to use NVM3 to store data in non-volatile memory. When doing so, these applications:

- Use the common default NVM3 instance

- Include the NVM3 Core and the NVM3 Default Instance components in the project

- Use the native NVM3 API to access the NVM3 object

If you have non-volatile data stored in the older SimEE format and need to preserve the data when migrating your Zigbee application to EmberZNet v7.x or higher, then the Token Manager component is available to help manage the data storage. For more information, see the Token Manager component's description.

The NVM3 key space used by the Zigbee stack is 0x10000 to 0x1FFFF.
