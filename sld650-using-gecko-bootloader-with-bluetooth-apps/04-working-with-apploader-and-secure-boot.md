# Working with AppLoader and Secure Boot

When employing secure boot, the AppLoader and application must be signed individually for the application to be allowed to run. This is because the AppLoader authenticates the application before allowing it to run, just as the bootloader authenticates the AppLoader before allowing it to run.

![diagram](resources/sld650-image18.png)

This section describes how to work with this capability in a production environment.

## Creating a Single Signed Image with a Batch File

As described previously, a signed GBL file can be produced by executing `create_bl_files.bat/create_bl_files.sh/create_bl_files.py` with a private signing key file, `app-sign-key.pem` in the same folder or by using the [Post-Build Editor](./06-post-build-editor.md). The resulting GBL file, `full.gbl`, can be flashed directly to the target device for a successful boot. This method is convenient for testing secure boot during the development process, but is not secure for production since it requires the private signing key to be available in plain PEM format, rather than isolating it in a Hardware Security Module (HSM) and does not support the use of bootloader certificates.

## Signing Firmware Images for Production

For Series 1 Devices:

To sign a firmware image using an HSM, the image must first be separated into the AppLoader and application parts as follows.

1. Extract the AppLoader portion with the following command: `objcopy -O srec -j .text_apploader* apploader.s37`.

2. Sign the AppLoader for secure boot. For specific instructions on signing images with an HSM, see *Signing an Application for Secure Boot using a Hard Security Module* in [Simplicity Commander Reference Guide](https://docs.silabs.com/simplicity-commander/latest/simplicity-commander-commands/convert-and-modify-file-commands#signing-an-application-for-secure-boot-signing-using-a-signature-created-by-a-hardware-security-module). It is also possible to sign the AppLoader with a certificate although direct signing is sufficient for most use cases. For instructions on signing with a certificate with an HSM, see *Signing an Application for Secure Boot using an Intermediary Certificate* in the Simplicity Commander Reference Guide.

3. Extract the application with the following command: `objcopy -O srec -R .text_apploader* -R .text_signature* application.s37`.

4. Sign the application for secure boot. The instructions for signing the AppLoader in step 2 above also apply to the application.

5. Combine the signed AppLoader and application into a single image as follows: `commander convert <signed apploader>.s37 <signed application>.s37 –outfile signed_fw_image.s37`.

6. Optionally, see *Creating a Partial Signed and Encrypted GBL Upgrade File for Use with a Hardware Security Module* and *Creating a Signed GBL File Using a Hardware Security Module* in the *Simplicity Commander Reference Guide*.

For Series 2 devices (EFR32xG2x), the AppLoader can be included in the bootloader project as a software component. This makes it possible to sign the application and bootloader binaries without any need to perform steps 1 – 6 above.

For more information on secure boot, see [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/).
