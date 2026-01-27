# Using NVM3 with Silicon Labs Connect

This chapter applies to Connect in the Proprietary Flex SDK v3.x used with the Project Configurator in Simplicity Studio 5. Connect in Proprietary Flex SDK v2.x uses AppBuilder in Simplicity Studio 4. AppBuilder use is described in [Using NVM3 in AppBuilder-Based Applications](./08-using-nvm3-in-appbuilder-based-applications.md).

For Silicon Labs Connect v3.x applications, a **Token Manager** component is available under Platform \> Driver in the Simplicity Studio Project Configurator Software Component view. While the token manager provides the [Token API](./10-nvm3-api-options.md), an additional component must be selected for the token storage backend, either **Token Manager using Sim EEPROM 1**, **Token Manager using Sim EEPROM 2**, or **Token Manager using NVM3.** If NVM3 is chosen as the storage backend, **NVM3** and the **NVM3 Default Instance** components are included in the project.
