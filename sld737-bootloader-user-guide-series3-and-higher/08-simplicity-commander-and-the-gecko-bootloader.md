# Simplicity Commander and the Gecko Bootloader

Simplicity Commander is a single, all-purpose tool to be used in a production environment. It is invoked using a simple CLI (Command Line Interface) that is also scriptable. You can use Simplicity Commander to perform these essential tasks:

- Generating key files for signing and encryption

- Signing application images for Secure Boot

- Creating GBL images (encrypted or unencrypted, signed, or unsigned)

- Parsing GBL images

Simplicity Commander is used throughout the examples in the following sections. For more information on executing commands to complete these tasks, see the [Simplicity Commander User Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-start/).

>**Note**: Simplicity Commander also offers a GUI (Graphical User Interface) that can be used in the lab for typical tasks such as flashing device images. The functions described in this User Guide are performed from the CLI.

## Creating GBL V4 Files Using Simplicity Commander

To create an unsigned GBL V4 file from an application **myapp.s37**, execute `commander gbl4 create myapp.gbl -–config configfile.yaml`.

Example of a config file is as follows:

```
manifest:
  product_id: “00000000000000000000000000000000”
updates:
  - data: myapp.s37
    block_size: 0
```

To create an unsigned GBL V4 file from a bootloader upgrade **mybootloader.s37**, execute `commander gbl4 create mybootloader.gbl –-config confligfile.yaml`.

Example of a config file is as follows:

```
manifest:
  product_id: “00000000000000000000000000000000”
updates:
  - data: mybootloader.s37
    block_size: 0
```

This file can be used with the standalone bootloader configurations of the Gecko Bootloader.

To create an unsigned GBL file from a Secure Engine, upgrade **mySecureElement.seuv2**, and execute `commander gbl4 create mySecureElement.gbl –config configfile.yaml`.

Example of a config file is as follows:

```
manifest:
  product_id: “00000000000000000000000000000000”
se_upgrade:
  se_file: mysecureElement.seuv2
```

The Secure Engine images, .seuv2, are provided by Silicon Labs and can be found through Simplicity Studio. See the *Gecko Bootloader Operation - Secure Engine Upgrade* section on the *Gecko Bootloader Operation Secure Engine Upgrade* page.

The command can also be used to create a single upgrade image, suitable for use with application bootloader configurations of the Gecko Bootloader: `commander gbl4 create myupgrade.gbl –config configfile.yaml`.

Example of a config file is as follows:

```
manifest:
  product_id: “00000000000000000000000000000000”
updates:
  - data: myapp.s37
  - data:mybootloader.s37

se_upgrade:
  se_file: mysecureElement.seuv2
```
