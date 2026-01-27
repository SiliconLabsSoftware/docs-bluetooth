# Using NVM3 with Silicon Labs OpenThread Applications

This section applies to the Silicon Labs OpenThread SDK v1.x used with the Simplicity Project Configurator in Simplicity Studio 5. All OpenThread sample applications in the GSDK by default are configured to use NVM3 to store data in non-volatile memory. When doing so, these applications:

- Use the common default NVM3 instance.

- Include the NVM3 Core and the NVM3 Default Instance components in the project.

- Use the native NVM3 API to access the NVM3 object

The NVM3 key space used by the OpenThread stack is 0x20000 to 0x2FFFF.
