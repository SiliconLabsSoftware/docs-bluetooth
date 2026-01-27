# TrustZone Platform Examples

The following TrustZone platform examples located in the C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\gecko_sdk\app\common\example folder (Windows) demonstrate the TrustZone implementation on Series 2 devices. All TrustZone platform examples do not include [Gecko Bootloader](./05-r-implementation.md#).

## TrustZone PSA Attestation

![tz_attestation](./resources/sld717-tz-attestation.png)

![tz_attestation_folder](./resources/sld717-tz-attestation-folder.png)

<table class="classic"><colgroup><col width="50%"/><col width="50%"/></colgroup><thead><tr><th>Example Folder</th><th>Description</th></tr></thead><tbody><tr><td><p>tz_psa_attestation</p></td><td><p>The workspace description file (tz_psa_attestation_ws.slcw) creates the TrustZone PSA Attestation example. The project description file (tz_psa_attestation_s.slcp) configures a Secure application that provides the Secure Library functionality required by the Non-secure application.</p></td></tr><tr><td><p>tz_psa_attestation_ns</p></td><td><p>The project description file (tz_psa_attestation_ns.slcp) configures a Non-secure application for the TrustZone PSA Attestation example.</p></td></tr></tbody></table>

**Notes**:

- This example cannot run if the `SECURE_BOOT_ENABLE` (root of trust of the attestation) option in SE OTP is disabled.

- The combined image of Secure and Non-secure applications is signed by the `example_signing_key.pem` (private key) in C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\gecko_sdk\platform\commonfolder (Windows). The `example_signing_pubkey.pem` (public key) in the same folder is installed to the SE OTP to verify the image signature during [Secure Boot](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/).

## TrustZone PSA Crypto ECDH

![tz-ecdh](./resources/sld717-tz-ecdh.png)

![tz-cdh-folder](./resources/sld717-tz-ecdh-folder.png)

| Example Folder |Description |
|-|-|
| tz_psa_crypto_ecdh |The workspace description file (tz_psa_crypto_ecdh_ws.slcw) upgrades the existing Platform - PSA Crypto ECDH example to TrustZone-aware. The project description file (tz_psa_crypto_ecdh_s.slcp) configures a Secure application that provides the Secure Library functionality required by the Non-secure application. |
| tz_psa_crypto_ecdh_ns |  The project description file (tz_psa_crypto_ecdh_ns.slcp) configures the existing Platform - PSA Crypto ECDH example as a Non-secure application. The source code can be reused without changes.   |

The following sections use Simplicity Studio v5.6.3.0 and GSDK v4.2.2. The procedures and pictures may be different if using higher versions of Simplicity Studio 5 and GSDK.

## Project Description File

The project description file ([`.slcp`](https://siliconlabs.github.io/slc-specification/1.0/format/project/)) contains references to the GSDK used and a list of components to use from these. The TrustZone-aware application requires separate `slcp` files for the Secure and Non-secure applications.

Users should not directly edit the `slcp` files, but rather use the [Memory Editor](#memory-editor) and Post Build Editor in Simplicity Studio to update the [memory configuration](#memory-configuration) and post-build actions.

### Secure Application

The following figure describes which TrustZone software components are installed for the TrustZone Secure library of the [TrustZone PSA Crypto ECDH](#trustzone-platform-examples) example.

![tz_secure_component](./resources/sld717-tz-secure-component.png)

**Notes**:

- The services provided by the Secure library are standardized.

- The source files for the Secure library will be automatically added to the application when generating the Secure project from the `slcp` file. For the current TrustZone implementation, modifications of the source files of the Secure library are not recommended.

### Non-secure Application

The following figure describes which TrustZone software components are installed for the Non-secure application of the [TrustZone PSA Crypto ECDH](#trustzone-platform-examples) example.

![tz_nonsecure_component](./resources/sld717-tz-nonsecure-component.png)

**Notes**:

- The following software components are automatically installed when [PSA Crypto and ITS](./05-r-implementation.md#) services are used on the Non-secure application.

  - MSC Service for TrustZone Secure Key Library

  - NVM3 Service for TrustZone Secure Key Library

  - PSA Crypto Service for TrustZone Secure Key Library

  - PSA ITS Service for TrustZone Secure Key Library

  - SYSCFG Service for TrustZone Secure Key Library

- The following software components can be installed to the Non-secure application when those services are required.

  - [PSA Attestation Service for TrustZone Secure Key Library](./05-r-implementation.md#psa-attestation)

  - [SE Manager Service for TrustZone Secure Key Library](./05-r-implementation.md#se-manager)

## Workspace

A workspace is a structure that can contain multiple projects. 'Workspace' is a generic term for this construct. In the context of Simplicity Studio, where workspace has a different, Eclipse-based, meaning, workspaces are referred to as [Solutions](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-developing-with-project-configurator/project-solutions).

 The workspace description file ([`.slcw`](https://siliconlabs.github.io/slc-specification/latest/format/workspace/)) contains references to projects ([`.slcp`](https://siliconlabs.github.io/slc-specification/latest/format/project/)) that make up the workspace. Users should not directly edit the `slcw` file, but rather use the Post Build Editor in Simplicity Studio to update the post-build actions.

## Memory Configuration

The [memory configurations](./06-r-migration.md#linker-file) in the TrustZone platform examples are based on the Series 2 radio board with minimum flash (512 kB) and RAM (32 kB), so these configurations can run on all Series 2 radio boards. Users can customize the settings when more flash and RAM are available on the selected device.

- Memory flash size (total) = `memory_flash_size` (S) + `memory_flash_size` (NS) = 512 kB

- Memory RAM size (total) = `memory_ram_size` (S) + `memory_ram_size` (NS) = 32 kB

### Secure Application

The project description file of the Secure application (`*_s.slcp`) uses the default memory setting below to generate the [Secure linker file](./06-r-migration.md#) (linkerfile.ld for GCC and linkerfile.icf for IAR in the project autogen folder).

The actual memory usage during software development is unknown, so it needs to reserve enough flash (`memory_flash_size`: 176 kB) and RAM (`memory_ram_size`: 12 kB) for the Secure part of all TrustZone platform examples. The bigger RAM size (including stack and heap) is mainly for the software fallback on cryptographic operations in PSA Crypto.

<table class="classic"><colgroup><col width="33%"/><col width="33%"/><col width="34%"/></colgroup><thead><tr><th>Default Memory Setting (Secure)</th><th>xG21 and xG22 Devices</th><th>Other Series 2 Devices</th></tr></thead><tbody><tr><td><p>memory_flash_start</p></td><td><p>0x00000000</p></td><td><p>0x08000000</p></td></tr><tr><td><p>memory_flash_size</p></td><td><p>0x0002C000 (176 kB)</p></td><td><p>0x0002C000 (176 kB)</p></td></tr><tr><td><p>memory_ram_start</p></td><td><p>0x20000000</p></td><td><p>0x20000000</p></td></tr><tr><td><p>memory_ram_size</p></td><td><p>0x00003000 (12 kB)</p></td><td><p>0x00003000 (12 kB)</p></td></tr></tbody></table>

```C

 MEMORY
 {
   FLASH   (rx)  : ORIGIN = 0x0, LENGTH = 0x2c000
   RAM     (rwx) : ORIGIN = 0x20000000, LENGTH = 0x3000
 }

```

### Non-secure Application

The project description files of the Non-secure application (*_ns.slcp) use the default memory setting below to generate the [Non-secure linker file](./06-r-migration.md#) (linkerfile.ld for GCC and linkerfile.icf for IAR in the project autogen folder).

The actual memory usage during software development is unknown, so the remaining flash (`memory_flash_size`: 336 kB) and RAM (`memory_ram_size`: 20 kB) should be big enough for the Non-secure part of all TrustZone platform examples.

<table class="classic"><colgroup><col width="33%"/><col width="33%"/><col width="34%"/></colgroup><thead><tr><th>Default Memory Setting (Non-secure)</th><th>xG21 and xG22 Devices</th><th>Other Series 2 Devices</th></tr></thead><tbody><tr><td><p>memory_flash_start</p></td><td><p>0x0002C000 (176 kB)</p></td><td><p>0x0802C000 (176 kB)</p></td></tr><tr><td><p>memory_flash_size</p></td><td><p>0x00054000 (336 kB)</p></td><td><p>0x00054000 (336 kB)</p></td></tr><tr><td><p>memory_ram_start</p></td><td><p>0x20003000 (12 kB)</p></td><td><p>0x20003000 (12 kB)</p></td></tr><tr><td><p>memory_ram_size</p></td><td><p>0x00005000 (20 kB)</p></td><td><p>0x00005000 (20 kB)</p></td></tr></tbody></table>

```C

 MEMORY
 {
   FLASH   (rx)  : ORIGIN = 0x2c000, LENGTH = 0x54000
   RAM     (rwx) : ORIGIN = 0x20003000, LENGTH = 0x5000
 }

```

>**Note**: The usable flash for Non-secure code should be equal to `memory_flash_size` - `NVM size` (default is 40 kB) if NVM3 storage is required.

### Memory Editor

The default memory setting of [Secure](#secure-application) and [Non-secure](#non-secure-application) applications are good enough for software development and debugging. The final memory layouts of Secure and Non-secure projects are deduced by inspecting the flash and RAM usage in the Secure application memory map file (.map).

The [Memory Editor](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-getting-started/start-a-project#memory-editor) in Simplicity Studio 5 is a graphical tool for editing the memory layout (flash and RAM) of the applications in the workspace. The Memory Editor will update the linker file in the project `autogen` folder with the custom settings. [Rebuild](#build) the projects to use the new memory configurations in the linker files.

The Memory Editor is located at the __Quick Links__ and __CONFIGURATION TOOLS__ of Secure or Non-secure `slcp` file.

![quick_links](./resources/sld717-quick-links.png)

![conf_tools](./resources/sld717-conf-tools.png)

The following items will be determined by the flash usage in the Secure application memory map file:

- memory_flash_size (S)

- memory_flash_start (NS)

- memory_flash_szie (NS)

![memory_edit_flash](./resources/sld717-memory-edit-flash.png)

>**Note**: The Memory Editor in Simplicity Studio v5.6.3.0 can only adjust the flash size in __8 kB__ (page size) alignment, which may not fit the [4kB alignment](./06-r-migration.md) between the Secure and Non-secure flash boundary.

The following items will be determined by the RAM usage in the Secure application memory map file:

- memory_ram_size (S)

- memory_ram_start (NS)

- memory_ram_szie (NS)

![memory_edit_ram](./resources/sld717-memory-edit-ram.png)

## Build

The Secure project must be built first to create the Secure object library (trustzone_secure_library.o) with function entries for the Non-secure project. Both projects need to be rebuilt if any changes in the Secure project. Users can use Simplicity IDE in Simplicity Studio 5 or IAR EWARM v9.20.4 to build the TrustZone platform examples.

### Simplicity IDE

The following procedures are based on the __TrustZone PSA Crypto ECDH__ example on BRD4182A Radio Board (EFR32MG22C224F512IM40).

1. Use the `tz_psa_crypto` keyword to search in __EXAMPLE PROJECTS & DEMOS__ tab. Select the __`tz_psa_crypto_ecdh_ws`__ example.

  ![tz_gcc_search_ws](./resources/sld717-tz-gcc-search-ws.png)
2. Click __[CREATE]__ to generate the [solution](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-developing-with-project-configurator/project-solutions).

  ![./resources/sld717-tz_ecdh](./resources/sld717-tz-ecdh.png)
3. The __Project Configuration__ dialog shows the Secure and Non-secure projects in the target solution. Click __[FINISH]__ to start the creation process.

  ![tz_gcc_proj_conf](./resources/sld717-tz-gcc-proj-conf.png)
4. The Simplicity IDE perspective opens after finishing the solution creation. Click __Build__ on the Simplicity IDE perspective toolbar to build the projects of a selected solution in order (Secure then Non-secure).

  ![tz_gcc_build](./resources/sld717-tz-gcc-build.png)
5. The post-build actions (`.slpb` files) of the Secure project, Non-secure project, and workspace will be processed in sequence if the solution is successfully built. The combined image (tz_psa_crypto_ecdh_ws-combined.s37) in the Secure project artifact folder can be used for programming the device or debugging.

  ![tz_gcc_combine](./resources/sld717-tz-gcc-combine.png)
6. Use [Memory Editor](#memory-editor) to finalize the memory layouts of Secure and Non-secure applications and rebuild the solution to update the memory configurations.

>**Note**: The Simplicity IDE can only apply the post-build action to a particular project if multiple Secure or Non-secure projects exist in the solution.

### IAR EWARM

The following procedures are based on the __TrustZone PSA Crypto ECDH__ example on BRD4181A Radio Board (EFR32MG21A010F1024IM32).

1. Follow steps 1 to 3 in [TrustZone PSA Crypto ECDH](#simplicity-ide) to generate the solution for the `tz_psa_crypto_ws`. Select the `tz_psa_crypto_ecdh_s.slcp` file.

2. The __Overview__ tab shows the __Target and Tool Settings__ card on the left side. Scroll down if necessary and click __[ChangeTarget/SDK/Generators]__.

  ![tz_iar_s_change](./resources/sld717-tz-iar-s-change.png)
3. Drop down the __CHANGE PROJECT GENERATORS__ list and select __IAR Embedded Workbench Project__.

  ![tz_iar_s_select](./resources/sld717-tz-iar-s-select.png)
4. Click __[Save]__ to generate an IAR __Secure__ project (tz_psa_crypto_ecdh_s.ewp).

  ![tz_iar_s_save](./resources/sld717-tz-iar-s-save.png)
5. Select the `tz_psa_crypto_ecdh_ns.slcp` file. Repeat steps 2 to 4 to generate an IAR __Non-secure*__ project (tz_psa_crypto_ecdh_ns.ewp).
6. Use a text editor to create an IAR tz_psa_crypto_ecdh_ws.ewwfile (shown below) to house the projects (tz_psa_crypto_ecdh_s.ewpand tz_psa_crypto_ecdh_ns.ewp) generated in steps 4 and 5. The location of the tz_psa_crypto_ecdh_ws.eww is the directory for $WS_DIR$.

```C

<?xml version ="1.0" encoding="iso-8859-1"?>
<workspace>
  <project>
    <path>$WS_DIR$\tz_psa_crypto_ecdh_s\tz_psa_crypto_ecdh_s.ewp</path>
  </project>
  <project>
    <path>$WS_DIR$\tz_psa_crypto_ecdh_ns\tz_psa_crypto_ecdh_ns.ewp</path>
  </project>
  <batchBuild/>
</workspace>

```

  ![./resources/sld717-tz_iar_folder](./resources/sld717-tz-iar-folder.png)
7. Double-click the tz_psa_crypto_ecdh_ws.ewwfile to open the workspace that includes Secure and Non-secure projects.
  ![./resources/sld717-tz_iar_ws](./resources/sld717-tz-iar-ws.png)
8. Click the tz_psa_crypto_ecdh_s tab to open the Secure project. Click ![tz_iar_make_icon](./resources/sld717-tz-iar-make-icon.png) (Make) to build. It exports the Secure object library (trustzone_secure_library.o) for function entries that will be used by the Non-secure project.

  ![tz_iar_s](./resources/sld717-tz-iar-s.png)
9. Click the tz_psa_crypto_ecdh_ns tab to open the Non-secure project.

  ![./resources/sld717-tz_iar_ns](./resources/sld717-tz-iar-ns.png)
10. The [`SL_TRUSTZONE_NONSECURE`](./06-r-migration.md#peripheral-addresses-in-device-header-files) defined in the Non-secure project disables the [CMSE compiler option](./06-r-migration.md#startup-code) (`--cmse`) regardless of whether the __Project → Options... → General Options → 32-bit → TrustZone → Mode:__ setting is Secure or Non-secure. So changing this configuration from Secure to Non-secure is optional. Click __[OK]__ to exit.

  ![tz_iar_select_ns](./resources/sld717-tz-iar-select-ns.png)
11. Click ![tz_iar_make_icon](./resources/sld717-tz-iar-make-icon.png) (Make) to build the Non-secure project. The post-build actions of the workspace (tz_psa_crypto_ecdh_ws.slpb) will be triggered in IAR to combine the Secure and Non-secure images (tz_psa_crypto_ecdh_ws-combined.s37) to the artifact folder of `tz_psa_crypto_ecdh_s` for programming the device.

  ![tz_iar_combine](./resources/sld717-tz-iar-combine.png)
12. Use [Memory Editor](#memory-editor) to finalize the memory layouts of Secure and Non-secure applications and rebuild the Secure and Non-secure projects to update the memory configurations.

>**Note**: The IAR EWARM can only apply the workspace post-build action to a particular project if multiple Secure or Non-secure projects exist in the workspace.

## Debugging

Users can use Simplicity IDE in Simplicity Studio 5 or IAR EWARM v9.20.4 to debug the TrustZone platform examples. Building the projects with Optimization Level None (-O0) is recommended for debugging.

### Simplicity IDE

The TrustZone debugging process on Simplicity IDE is similar to the existing sample projects in Simplicity Studio.

1. [GNU Debugger (GDB)](./06-r-migration.md#debugger) is recommended to debug TrustZone applications.

2. Flash the combined image (tz_psa_crypto_ecdh_ws-combined.s37) generated in [Simplicity IDE](#simplicity-ide) to the device.

3. Select the Secure or Non-secure project and use the __Debug__ icon to launch a debug session.

  ![gcc_debug_launch](./resources/sld717-gcc-debug-launch.png)
4. Follow the instructions in the __[Using the Debugger](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-testing-and-debugging/using-the-debugger)__ section in Simplicity Studio 5 User's Guide to debug the Secure or Non-secure application.
5. The debugger cannot step into the function in a Non-secure application when debugging the Secure application and vice versa. Use the __Program Counter__ (PC in Secure or Non-secure address) in the __Registers__ window to determine the program status.

  ![gcc_debug_state](./resources/sld717-gcc-debug-state.png)

### IAR EWARM

Use the tz_psa_crypto_ecdh_ws.eww workspace created in [IAR EWARM](#iar-ewarm) for the debugger settings. Except for a minor difference in step 3, the following steps are the same as those to set up the Secure (tz_psa_crypto_ecdh_s) and Non-secure (tz_psa_crypto_ecdh_ns) projects for debugging.

1. Select __Options...__ in the ![tools_menu](./resources/sld717-tools-menu.png) context menu of the Secure or Non-secure project and open the __IDE Options → Stack__ dialog. Uncheck the __Stack pointer(s) not valid until program reaches*__ checkbox. Click __[OK]__ to exit.

  ![uncheck_stack](./resources/sld717-uncheck-stack.png)
2. Select __Options...__ in the ![project_menu](./resources/sld717-project-menu.png) context menu of the Secure or Non-secure project and open the window for __Debugger__ options. Click the __Setup__ tab to open a dialog, and uncheck the __Run to → main__ checkbox. Click the __Images__ tab to set up another option.

  ![uncheck_main](./resources/sld717-uncheck-main.png)
3. Check the __ownload extra image__ option. Enter the location of the .out file to __Path:__ with __Offset:__ set to 0. All project relative paths are resolved from the directory location of the tz_psa_crypto_ecdh_ws.eww workspace file.

  Location of Non-secure .out file for Secure project: tz_psa_crypto_ecdh_ns\ewarm-iar\exe\tz_psa_crypto_ecdh_ns.out

  ![set_image_s](./resources/sld717-set-image-s.png)

  Location of Secure .out file for Non-secure project: tz_psa_crypto_ecdh_s\ewarm-iar\exe\tz_psa_crypto_ecdh_s.out

  ![set_image_ns](./resources/sld717-set-image-ns.png)
4. Click the __Extra Options__ tab to set up another option.
5. Check the __Use command line options__. Enter --drv_vector_table_base=0x00000000 to __Command line options: (one per line)__ window. Click __[OK]__ to exit.

  ![set_extra](./resources/sld717-set-extra.png)
6. Finish the debug settings in Secure and Non-secure projects, and click ![debug_icon](./resources/sld717-debug-icon.png) (Download and Debug) in the Secure or Non-secure project to download the Secure and Non-secure images for debugging (assume both projects had successfully [built](#iar-ewarm) before). Click ![go_icon](./resources/sld717-go-icon.png) (Go) to start running the code in a Secure or Non-secure project.
7. The debugger will automatically switch between Secure and Non-secure projects when stepping into a function or hitting a breakpoint in a Secure or Non-secure project. Use the __Program Counter__ (PC in Secure or Non-secure address) or __SECURE__ (0 or 1) in the __Registers__ window to determine the program status.

  ![debug_state](./resources/sld717-debug-state.png)
8. Click ![stop_icon](./resources/sld717-stop-icon.png) (Stop Debugging) to end the debug session.

## Benchmark

The TrustZone implementation will affect the memory footprint and performance of cryptographic operations. The following comparisons are based on the __TrustZone PSA Crypto ECDH__ example on BRD4182A Radio Board (EFR32MG22C224F512IM40) with SE firmware v1.2.14.

### Memory Footprint

The memory footprint of a TrustZone project depends on which services (software components in the figure below) provided by the [Secure Library](./05-r-implementation.md) are used in the Non-secure application (`tz_psa_crypto_ecdh_ns` project).

![tz_nonsecure_component](./resources/sld717-tz-nonsecure-component.png)

The following tables compare the memory footprint of the [TrustZone-unaware](./06-r-migration.md#startup-code) (`Platform - PSA Crypto ECDH`) and [TrustZone-aware](./07-r-ex.md#memory-configuration) projects (`tz_psa_crypto_ecdh_ws`) based on the following conditions.

- The `tz_psa_crypto_ecdh_ns` reuses the source code from the `Platform - PSA Crypto ECDH` example without any changes.

- The total size in `tz_psa_crypto_ecdh_ns` does not consider the [4 kB alignment](./06-r-migration.md#) on the Secure and Non-secure flash and RAM. The 4 kB alignment requirement will increase the actual usage of flash and RAM.

- All source code is compiled with Optimize for size (-Os) in Simplicity IDE (GNU ARM v10.3.1) of Simplicity Studio 5.

__Table__: Flash Size Comparison

<table class="classic"><colgroup><col width="20%"/><col width="20%"/><col width="20%"/><col width="20%"/><col width="20%"/></colgroup><thead><tr><th>Platform Example</th><th>Secure</th><th>NSC</th><th>Non-secure</th><th>Total</th></tr></thead><tbody><tr><td><p>Platform - PSA Crypto ECDH</p></td><td><p>64688 B</p></td><td><p>-</p></td><td><p>-</p></td><td><p>64688 B</p></td></tr><tr><td><p>tz_psa_crypto_ecdh_ws</p></td><td><p>79172 B</p></td><td><p>288 B</p></td><td><p>29264 B</p></td><td><p>108724 B</p></td></tr></tbody></table>

>**Note**: The NSC is part of the Secure code, and the total size does not include the flash for NVM3 storage.

__Table__: RAM Size Comparison

<table class="classic"><colgroup><col width="20%"/><col width="20%"/><col width="20%"/><col width="20%"/><col width="20%"/></colgroup><thead><tr><th>Platform Example</th><th>Secure</th><th>NSC</th><th>Non-secure</th><th>Total</th></tr></thead><tbody><tr><td><p>Platform - PSA Crypto ECDH</p></td><td><p>3784 B</p></td><td><p>-</p></td><td><p>-</p></td><td><p>3764 B</p></td></tr><tr><td><p>tz_psa_crypto_ecdh_ws</p></td><td><p>2156 B</p></td><td><p>-</p></td><td><p>1200 B</td><td><p>3356 B</p></td></tr></tbody></table>

>**Note**: The total size does not include the RAM for the stack and heap. The Secure and Non-secure applications have their independent stack and heap.

### PSA Crypto Performance

The following sections compare the PSA Crypto performance of the [TrustZone-unaware](./06-r-migration.md#) (`Platform - PSA Crypto ECDH`) and [TrustZone-aware](#memory-configuration) projects (`tz_psa_crypto_ecdh_ws`) based on the following conditions.

- The `tz_psa_crypto_ecdh_ns` reuses the source code from the `Platform - PSA Crypto ECDH` example without any changes.

- All source code is compiled with Optimize most (-O3) in Simplicity IDE (GNU ARM v10.3.1) of Simplicity Studio 5.

- Use ECC curve `SECP256R1` on volatile and persistent keys.

- The EFR32MG22C224 runs at 38 MHz HFRCODPLL.

#### Volatile key ECDH operation on `Platform - PSA Crypto ECDH`

```C

  . ECDH Client
  + Creating a SECP256R1 (256-bit) VOLATILE PLAIN client key... PSA_SUCCESS (cycles: 2928 time: 77 us)
  + Creating a SECP256R1 (256-bit) VOLATILE PLAIN server key... PSA_SUCCESS (cycles: 2960 time: 77 us)
  + Exporting a public key of a SECP256R1 (256-bit) VOLATILE PLAIN server key... PSA_SUCCESS (cycles: 332134 time: 8740 us)
  + Computing client shared secret with a SECP256R1 (256-bit) server public key... PSA_SUCCESS (cycles: 336860 time: 8864 us)

```

#### Volatile key ECDH operation on `tz_psa_crypto_ecdh_ws`

```C

  . ECDH Client
  + Creating a SECP256R1 (256-bit) VOLATILE PLAIN client key... PSA_SUCCESS (cycles: 5047 time: 132 us)
  + Creating a SECP256R1 (256-bit) VOLATILE PLAIN server key... PSA_SUCCESS (cycles: 5067 time: 133 us)
  + Exporting a public key of a SECP256R1 (256-bit) VOLATILE PLAIN server key... PSA_SUCCESS (cycles: 333956 time: 8788 us)
  + Computing client shared secret with a SECP256R1 (256-bit) server public key... PSA_SUCCESS (cycles: 338470 time: 8907 us)

```

#### Persistent key ECDH operation on `Platform - PSA Crypto ECDH`

```C

  . ECDH Client
  + Creating a SECP256R1 (256-bit) PERSISTENT PLAIN client key... PSA_SUCCESS (cycles: 27489 time: 723 us)
  + Creating a SECP256R1 (256-bit) PERSISTENT PLAIN server key... PSA_SUCCESS (cycles: 27587 time: 725 us)
  + Exporting a public key of a SECP256R1 (256-bit) PERSISTENT PLAIN server key... PSA_SUCCESS (cycles: 332949 time: 8761 us)
  + Computing client shared secret with a SECP256R1 (256-bit) server public key... PSA_SUCCESS (cycles: 337803 time: 8889 us)

```

#### Persistent key ECDH operation on `tz_psa_crypto_ecdh_ws`

```C

  . ECDH Client
  + Creating a SECP256R1 (256-bit) PERSISTENT PLAIN client key... PSA_SUCCESS (cycles: 46998 time: 1236 us)
  + Creating a SECP256R1 (256-bit) PERSISTENT PLAIN server key... PSA_SUCCESS (cycles: 45962 time: 1209 us)
  + Exporting a public key of a SECP256R1 (256-bit) PERSISTENT PLAIN server key... PSA_SUCCESS (cycles: 334127 time: 8792 us)
  + Computing client shared secret with a SECP256R1 (256-bit) server public key... PSA_SUCCESS (cycles: 338321 time: 8903 us)

```

The overheads on the TrustZone-aware project (`tz_psa_crypto_ecdh_ws`) are due to the following operations of [Secure Library](./05-r-implementation.md#) implementation.

- Packages the list of input arguments in the appropriate format before calling into the NSC function.

- Switches from a Non-secure to a Secure state.

- Validates all input arguments before calling into the function in SPE.

- Encrypts PSA ITS if using a persistent key.

- Returns to a Non-secure state.
