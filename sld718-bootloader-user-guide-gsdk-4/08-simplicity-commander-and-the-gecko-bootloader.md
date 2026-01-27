# Simplicity Commander and the Gecko Bootloader

Simplicity Commander is a single, all-purpose tool to be used in a production environment. It is invoked using a simple CLI (Command Line Interface that is also scriptable. You can use Simplicity Commander to perform these essential tasks:

- Generating key files for signing and encryption

- Signing application images for Secure Boot

- Creating GBL images (encrypted or unencrypted, signed, or unsigned)

- Parsing GBL images

Simplicity Commander is used throughout the examples in the following sections. For more information on executing the commands to complete these tasks, see [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

>**Note**: Simplicity Commander also offers a GUI (Graphical User Interface) that can be used in the lab for typical tasks such as flashing device images. The functions described in this User Guide are performed from the CLI.

## Creating GBL Files Using Simplicity Commander

To create an unsigned GBL file from an application **myapp.s37**, execute `commander gbl create myapp.gbl --app myapp.s37`.

To create an unsigned GBL file from a main bootloader upgrade **mybootloader.s37**, execute `commander gbl create mybootloader.gbl --bootloader mybootloader.s37`. This file can be used with the standalone bootloader configurations of the Gecko Bootloader.

To create an unsigned GBL file from a Secure Engine, upgrade **mySecureElement.seu**, and execute `commander gbl create mySecureElement.gbl --seupgrade mySecureElement.seu`. The Secure Engine images, .seu, are provided by Silicon Labs and  can be found through Simplicity Studio. See section [Gecko Bootloader Operation - Secure Engine Upgrade](./05-gecko-bootloader-operation-secure-engine-upgrade.md#gecko-bootloader-operation-secure-engine-upgrade).

These commands can also be combined to create a single upgrade image, suitable for use with application bootloader configurations of the Gecko Bootloader: `commander gbl create myupgrade.gbl --app myapp.s37 --bootloader mybootloader.s37 --seupgrade mySecureElement.seu`.
