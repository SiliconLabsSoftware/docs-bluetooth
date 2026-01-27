# Gecko Bootloader Operation - Bootloader Upgrade

Bootloader upgrade functionality is provided by the Secure Engine on Series 3 devices. The Secure Engine itself is also upgradable. For more details, see the *Gecko Bootloader Operation - Secure Engine Upgrade* page.

Requirements for upgrading the main bootloader vary depending on the bootloader configuration:

- Application bootloader with storage: Upgrading the main bootloader requires a single GBL file containing both bootloader and application upgrade images.

- Standalone bootloader with communication interface: Upgrading the bootloader requires a GBL file with only the bootloader upgrade image.

Security of the bootloader upgrade process is provided by signing the GBL file. See *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* on the *Gecko Bootloader Security Features* page.

## Bootloader Upgrade on Bootloaders with Communication Interface (Standalone Bootloaders)

The process is illustrated in the following figure:

![Standalone Bootloader: Bootloader Upgrade](resources/sld737-image5.png)

1. The device reboots into the bootloader.

2. A GBL file containing only a bootloader upgrade image is transmitted from the host to the device.

3. The contents of the GBL Bootloader tag are written to the data region in flash

4. The device reboots into the Secure Engine.

5. The Secure Engine replaces the main bootloader with the new version found in the bootloader upgrade GBL.

6. The device boots into the new main bootloader. Bootloader upgrade is complete.

A bootloader upgrade is started in the same way as an application upgrade.

### Downloading and Applying a Bootloader GBL Upgrade File

When the bootloader has entered the receive loop, a GBL upgrade file containing a bootloader upgrade is transmitted to the bootloader. When a packet is received, it is passed to the image parser. The image parser parses the data and returns bootloader upgrade data in a callback. The bootloader core implements this callback and flashes the data to the flash at the bootloader upgrade location.

The bootloader prevents a newly uploaded bootloader upgrade image from being interpreted as valid by holding back parts of the bootloader upgrade vector table until the GBL file hash and GBL signature (if required) have been verified.

When a complete bootloader upgrade image is received, the bootloader signals the Secure Engine that it should enter firmware upgrade mode. Secure Engine communication is used to signal that bootloader upgrade is ready to be performed.

The authenticity of the main bootloader optionally can be verified before applying the bootloader upgrade. See *Setting a Version Number* on the *Configuring the Gecko Bootloader* page for more information about versioning bootloader images.

### Upgrading Bootloaders without Secure Boot to Bootloaders with Secure Boot

A bootloader without the secure boot feature can be upgraded to a bootloader with the secure boot feature, using the following procedure:

1. Prepare a Gecko Bootloader image with secure boot enabled. The version of the bootloader needs to be higher than the bootloader on the device.

   - Turn on secure boot in Simplicity Studio by going to the **Bootloader Core** component and selecting the **Enable secure boot** option.

   - (Optional) In the **Bootloader Core** component, select the **Require signed firmware upgrade files** option. This means that the Gecko Bootloader will only accept signed GBL files.

2. Generate a public/private Signing Key pair. See the *Generating Keys* section on the *Gecko Bootloader Security Features* page for more information on creating a Signing Key pair.

3. Write the public key generated from the previous step to the device. The public key is stored as a manufacturing token in the device by default. Key locations are defined in the bootloader project file btl_security_tokens.c.

4. Create a GBL file using the Gecko Bootloader image. The GBL file needs to be signed/unsigned depending on the current configuration of the Gecko Bootloader running on the device. For more details on creating a GBL file, see the *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* section on the *Gecko Bootloader Security Features* page.

5. Upload the GBL file. For more details on the upgrade process, see the *Bootloader Upgrade on Bootloaders with Communication Interface (Standalone Bootloaders)* section.

### Enabling Secure Boot RTSL on Series 3 Devices

Secure Boot RTSL (Root of Trust and Secure Loader) can be enabled using the following procedure:

1. Prepare a Gecko Bootloader image with secure boot enabled. The version of the Gecko Bootloader needs to be higher than the Gecko Bootloader on the device.

   - Turn on secure boot in Simplicity Studio by going to the **Bootloader Core** component and selecting the **Enable secure boot** option.

   - (Optional) In the **Bootloader Core** component, select the **Require signed firmware upgrade files** option. This means that the Gecko Bootloader will only accept signed GBL files.

2. Generate a public/private Signing Key pair. See the *Generating Keys* section on the *Gecko Bootloader Security Features* page for more information on creating a Signing Key pair.

3. Prepare an application that installs the public key generated from step 2 to the Secure Engine One-time Programmable memory. Installing a key in the VSE requires a reset routine. Make sure that the application does not end up in the reset loop. Create an unsigned GBL file from this application and upload it. For more information on installing public keys, see the *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* section on the *Gecko Bootloader Security Features* page.

4. Sign the Gecko Bootloader image generated from step 1 using the private key generated in step 2. See the *Signing an Application Image for Secure Boot* section on the *Gecko Bootloader Security Features* page for more information on signing binaries.

5. Make a custom application that turns on secure boot on the Secure Engine and sign this application binary with the private key generated from step 2.

6. Create a GBL file using the Gecko Bootloader image from step 4.

7. Create a GBL file using the application from step 5. The GBL file need to be signed if the **Bootloader Core** component option **Require signed firmware upgrade files** was selected in step 1.

8. Upload the GBL file containing the Gecko Bootloader image.

9. Upload the GBL file containing the application.

## Bootloader Upgrade on Application Bootloaders with Storage

The process is illustrated in the following figure.

![Application Bootloader: Bootloader Upgrade](resources/sld737-image6.png)

1. A single GBL file containing a bootloader upgrade image is downloaded onto the storage medium of the device.

2. The device reboots into the main bootloader.

3. The main bootloader verifies the integrity of the upgrade image and then resets the device with reset reason BOOTLOADER_RESET_REASON_UPGRADE to apply the upgrade.

4. The device reboots into the Secure Engine.

5. The Secure Engine replaces the main bootloader with the new version.

6. The device boots into the new main bootloader.

A bootloader upgrade is started in the same way as an Application Upgrade. A GBL file containing a bootloader is written to storage by the application, and the bootloader is entered.

The bootloader iterates over the list of storage slots marked for bootload and attempts to verify the GBL file stored within. Verification returns information about whether the GBL file contains a bootloader, or both a bootloader and an application. The image parser parses the file. If the GBL file contains a bootloader, the bootloader upgrade data is returned in a callback. The bootloader core implements this callback and flashes the data to internal flash at the bootloader upgrade location.

The bootloader prevents a newly uploaded bootloader upgrade image from being interpreted as valid by holding back parts of the bootloader upgrade vector table until the GBL file hash  and GBL signature (if required) have been verified.

Secure Engine communication interface is used to signal the Secure Engine that a bootloader upgrade is ready to be performed.

On Series 3 devices, the authenticity of the main bootloader optionally can be verified before applying the bootloader upgrade. See the *Setting a Version Number* section on the *Configuring the Gecko Bootloader* page for more information about versioning bootloader images.

The new main bootloader is entered, and the images in the list of storage slots marked for bootload are verified. When the image parser parses the slot containing the GBL file with the bootloader + application upgrade, the version number of the bootloader upgrade is equal to the running main bootloader version, so another bootloader upgrade will not be performed. Instead, the application upgrade data are returned in a callback. Bootloading of the new application proceeds as described in the *Application Bootloader Operation* section on the *Gecko Bootloader Operation Application Upgrade* page.

### Storage Space Size Configuration

The storage space size must be configured to have enough space to store the upgrade images.

### Upgrading Bootloaders without Secure Boot to Bootloaders with Secure Boot

A bootloader without the secure boot feature can be upgraded to a bootloader with the secure boot feature. The procedure is as follows:

1. Prepare a Gecko Bootloader image with secure boot enabled. The version of the bootloader needs to be higher than the bootloader on the device.

   - Turn on secure boot from the **Bootloader Core** component in Simplicity Studio by selecting the **Enable secure boot** option.

2. Generate a public/private Signing Key pair. See the *Generating Keys* section on the *Gecko Bootloader Security Features* page for more information on creating a Signing Key pair.

3. Write the public key generated from the previous step to the device. The public key is stored as a manufacturing token in the device by default. This key can be written by application code running on the device. The Gecko Bootloader prepared from step 1 can be modified to look for the decryption and signature keys in a different location. Key locations are defined in the bootloader project file `btl_security_tokens.c`.

4. Prepare a signed application image using the private key generated in step 2. See the *Signing an Application Image for Secure Boot* section on the *Gecko Bootloader Security Features* page for more information on signing an application.

5. Create a GBL file using the Gecko Bootloader image and the signed application image. The GBL file needs to be signed/unsigned depending on the configuration of the Gecko Bootloader running on the device. For more details on creating a GBL file, see the *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* section on the *Gecko Bootloader Security Features* page.

6. Upload the GBL file. For more details on the upgrade process, see the *Bootloader Upgrade on Application Bootloaders with Storage* section.

### Enabling Secure Boot RTSL on Series 3 Devices

Secure Boot RTSL can be enabled by using the following procedure:

1. Prepare a Gecko Bootloader image with secure boot enabled. The version of the Gecko Bootloader needs to be higher than the Gecko Bootloader on the device.

   - Turn on secure boot from the **Bootloader Core** component in Simplicity Studio by selecting the **Enable secure boot** option.

   - (Optional) In the **Bootloader Core** component, select the **Require signed firmware upgrade files** option. This means that the Gecko Bootloader will only accept signed GBL files.

2. Generate a public/private Signing Key pair. See the *Generating Keys* section on the *Gecko Bootloader Security Features* page for more information on creating a Signing Key pair.

3. Prepare an application that installs the public key generated from step 2 to the Secure Engine One-time Programmable memory. Installing a key in VSE requires a reset routine. Make sure that the application does not end up in the reset loop. Create an unsigned GBL file from this application and upload it. For more details on creating a GBL file, see the *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* section on the *Gecko Bootloader Security Features* page.

4. Sign the Gecko Bootloader image generated from step 1 using the private key generated in step 2. See the *Signing an Application Image for Secure Boot* section on the *Gecko Bootloader Security Features* page for more information on signing binaries.

5. Make a custom application that turns on secure boot on the Secure Engine and sign this application binary with the private key generated from step 2. For more details on how to turn on secure boot on the Secure Engine.

6. Create a GBL file using the Gecko Bootloader image from step 4 and the application from step 5. The GBL file must be signed if the **Bootloader Core** component option **Require signed firmware upgrade files** was selected in step 1. For more details on creating a GBL file, see the *Creating a Signed and Encrypted GBL Upgrade Image File from an Application* section on the *Gecko Bootloader Security Features* page.

7. Upload the GBL file containing the Gecko Bootloader image and the application.
