# Upgrade Existing Application to TrustZone

The main concerns when upgrading existing deployment to the TrustZone solution are:

- The [Secure/Non-secure pair for the bootloader](./05-r-implementation.md#) (24 kB) does not fit inside the current allotted bootloader space (16 kB).

- The [Secure/Non-secure pair for the application](./07-r-ex.md#) does not fit inside the current allotted application space.

- The [PSA ITS](./05-r-implementation.md#) moves from a non-encrypted to an encrypted format, so the existing stored cryptographic keys in NVM3 cannot be reused after upgrading the current application to TrustZone.

The [Secure Library](./05-r-implementation.md#) is based on PSA Crypto, so the existing application cannot integrate with the TrustZone if one of the following conditions is valid.

- Use [SE Manager APIs](https://docs.silabs.com/gecko-platform/latest/service/api/group-sl-se-manager) for cryptographic and attestation operations.

- Use classic Mbed TLS APIs for cryptographic operations (except for X.509 certificate) and Transport Layer Security (TLS) protocol.

## System Requirements

The following table lists the tools and software required for TrustZone development on Series 2 devices.

<table class="classic"><colgroup><col width="30%"/><col width="20%"/><col width="50%"/></colgroup><thead><tr><th>Tool/Software</th><th>Required Version</th><th>Description</th></tr></thead><tbody><tr><td><p>GCC</p></td><td><p>v10.3.1</p></td><td><p>Fix a bug (ID 99271) on cmse_nonsecure_call attribute.</p></td></tr><tr><td><p>IAR EWARM</p></td><td><p>v9.20.4</p></td><td><p>Fix a bug (EWARM-9484) on __cmse_nonsecure_call attribute.</p></td></tr><tr><td><p>Segger J-Link</p></td><td><p>≥ v7.6.2c</p></td><td><p>v7.6.2c is the first version to add basic TrustZone support on Series 2 devices.</p></td></tr><tr><td><p>Simplicity Studio</p></td><td><p>≥ v5.6.3.0</p></td><td><p>v5.6.3.0 is the first version to support TrustZone software development on Series 2 devices.</p></td></tr><tr><td><p>Simplicity Commander</p></td><td><p>≥ v1.13.3</p></td><td><p>v1.13.3 includes a TrustZone-aware flash loader and supports features required for TrustZone development.</p></td></tr><tr><td><p>GSDK</p></td><td><p>≥ v4.2.2</p></td><td><p>GSDK v4.2.2 is the first version to support TrustZone software development on Series 2 devices.</p></td></tr><tr><td><p>SE Firmware</p></td><td><p>≥ v1.2.14</p></td><td><p>v1.2.14 is the first version to fully support TrustZone on xG21 (HSE) and xG22 (VSE) devices.</p></td></tr><tr><td><p>SE Firmware</p></td><td><p>≥ v2.2.1</p></td><td><p>v2.2.1 is the first version to fully support TrustZone on other Series 2 HSE and VSE devices.</p></td></tr></tbody></table>

**Notes**:

- Required GCC and IAR EWARM versions are GSDK-dependent.

- [Bug list of GCC v10.3](https://gcc.gnu.org/bugzilla/buglist.cgi?bug_status=RESOLVED&resolution=FIXED&target_milestone=10.3)

- [IAR EWARM release note](https://updates.iar.com/?product=EWARM)

- [Segger J-Link release note](https://www.segger.com/downloads/jlink/ReleaseNotes_JLink.html)

- [Simplicity Studio user guide](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-overview/)

- [Latest version of Simplicity Commander](https://www.silabs.com/developers/mcu-programming-options)

- [GSDK release note](https://github.com/SiliconLabs/gecko_sdk/releases)

- Silicon Labs strongly recommends installing the latest SE firmware on Series 2 devices to support the required TrustZone features. The latest SE firmware image and release notes after installing the GSDK (Windows folder):

  C:\Users\<PC USER NAME>\SimplicityStudio\SDKs\gecko_sdk\util\se_release\public

## Peripheral Addresses in Device Header Files

The device header files (e.g., efr32mg21b020f1024im32.h) need to be configurable for different situations. The `SL_TRUSTZONE_SECURE` and `SL_TRUSTZONE_NONSECURE` definitions specify whether the compilation is for Secure or Non-secure applications. The `SL_TRUSTZONE_SECURE` and `SL_TRUSTZONE_NONSECURE` should be exclusive. If none of the definitions are true, the state should be similar to the Non-secure configuration, but the [startup code](#startup-code) (`SystemInit()` in system_*.c) will be responsible for reconfiguring the system.

<table class="classic"><colgroup><col width="33%"/><col width="33%"/><col width="34%"/></colgroup><thead><tr><th>Define (Software Component)</th><th>Default Peripheral Pointer</th><th>Startup Code</th></tr></thead><tbody><tr><td><p>SL_TRUSTZONE_SECURE (TrustZone Secure)</p></td><td><p>Point to Secure peripherals (*_BASE = *_S_BASE)</p></td><td><p>No effect on SystemInit()</p></td></tr><tr><td><p>SL_TRUSTZONE_NONSECURE (TrustZone Non-Secure)</p></td><td><p>Point to Non-secure peripherals (*_BASE = *_NS_BASE)</p></td><td><p>No effect on SystemInit()</p></td></tr><tr><td><p>None of the above (-)</p></td><td><p>Point to Non-secure peripherals (*_BASE = *_NS_BASE)</p></td><td><p>SystemInit() moves peripherals to Non-secure</p></td></tr></tbody></table>

When building a Secure application (`SL_TRUSTZONE_SECURE` is true), all peripherals shall have their non-suffixed default address pointing to the Secure location of the peripheral (e.g., EMU). But the definitions in sl_trustzone_secure_config.h can force the addresses of specific peripherals pointing to the Non-secure location.

```C

#ifndef SL_TRUSTZONE_SECURE_CONFIG_H
#define SL_TRUSTZONE_SECURE_CONFIG_H
// Specify security configuration of peripherals. Peripherals that are not
// included here will automatically have their _BASE addresses point to their
// secure address. This might not be true, since most peripherals are configured
// to be non-secure -- but it's also not a problem if the peripheral is not
// accessed from the S app.
// Used in multiple places.
#define SL_TRUSTZONE_PERIPHERAL_CMU_S (0)
// Used by SE Manager service.
#define SL_TRUSTZONE_PERIPHERAL_AHBRADIO_S (0)
// Used by MSC service.
#define SL_TRUSTZONE_PERIPHERAL_LDMA_S (1)
// Used by MSC service.
#define SL_TRUSTZONE_PERIPHERAL_LDMAXBAR_S (1)
#endif // SL_TRUSTZONE_SECURE_CONFIG_H

```

```C

#if defined(SL_CATALOG_TRUSTZONE_SECURE_CONFIG_PRESENT)
#include "sl_trustzone_secure_config.h"
#endif
#if ((defined(SL_TRUSTZONE_SECURE) && !defined(SL_TRUSTZONE_PERIPHERAL_EMU_S))
    || (defined(SL_TRUSTZONE_PERIPHERAL_EMU_S) && (SL_TRUSTZONE_PERIPHERAL_EMU_S != 0)))
#define EMU_BASE               (EMU_S_BASE)                  /* EMU base address */
#else

```

In other cases (`SL_TRUSTZONE_NONSECURE` is true or both `SL_TRUSTZONE_SECURE` and `SL_TRUSTZONE_NONSECURE` are false), all peripherals shall have their non-suffixed default address pointing to the Non-secure location of the peripheral (e.g., EMU).

```C

#define EMU_BASE               (EMU_NS_BASE)                 /* EMU base address */

```

>**Note**: Do not install the __TrustZone Secure__ or __TrustZone Non-Secure__ software component to the [TrustZone-unaware](#startup-code) application.

## Startup Code

The startup code moves peripherals from Secure to Non-secure to support the [default peripheral locations](#peripheral-addresses-in-device-header-files). In a TrustZone-aware application (either `SL_TRUSTZONE_SECURE` or `SL_TRUSTZONE_NONSECURE` is true), this is the application's responsibility (skip lines 168 to 194 in `SystemInit()`) and is done in the [Secure firmware](./02-r-basic.md#) of the system.

For the TrustZone-unaware application (both `SL_TRUSTZONE_SECURE` and `SL_TRUSTZONE_NONSECURE` are false), the `SystemInit()` in system_*.c (e.g., system_efr32mg21.c) moves peripherals to the Non-secure location.

- The `SystemInit()` sets the accesses of all peripherals to Non-secure except for the [SMU](./03-r-bls.md#) and HSE SEMAILBOX (lines 172 to 178).

- The `SystemInit()` sets the SAU in [All Non-secure](./03-r-bls.md#) configuration (lines 180 to 187).

  - It ensures Non-secure access to Non-secure peripherals.

  - The device component files (e.g., `efr32mg21b020f1024im32.slcc`) enable the [CMSE](https://developer.arm.com/documentation/ecm0359818/latest) compiler option (`-mcmse` for GCC and `--cmse` for IAR) to pass the condition in line 181 to program the SAU.

  - To catch the missing CMSE compiler option, it will generate a preprocessor error (line 186) if the CMSE flag is not set when manually upgrading a project from GSDK v4.0.x to ≥v4.1.x for the TrustZone-unaware application.

- The `SystemInit()` does not program the [ESAU](./03-r-bls.md#) (default Secure flash is 32 MB), so the whole program is run in the __Secure__ state.

- The `SystemInit()` also enables the [`BMPUSEC`](./03-r-bls.md#) and [`PPUSEC`](./03-r-bls.md#) interrupts in the SMU (lines 189 to 193). It ensures the TrustZone-unaware application catches any violations of Bus Master and peripheral security access permissions.

![system_init](./resources/sld717-system-init.png)

The `SMU_BASE` and HSE `SEMAILBOX_HOST_BASE` in device header files must point to the Secure location regardless of the `SL_TRUSTZONE_SECURE` and `SL_TRUSTZONE_NONSECURE` settings to avoid security violations on peripherals in the TrustZone-unaware application (SMU and HSE SEMAILBOX are set to Secure peripherals).

```C

#if ((defined(SL_TRUSTZONE_SECURE) && !defined(SL_TRUSTZONE_PERIPHERAL_SMU_S))
    || (defined(SL_TRUSTZONE_PERIPHERAL_SMU_S) && (SL_TRUSTZONE_PERIPHERAL_SMU_S != 0)))
#define SMU_BASE               (SMU_S_BASE)                  /* SMU base address */
#else
#define SMU_BASE               (SMU_S_BASE)                  /* SMU base address */

```

```C

#if ((defined(SL_TRUSTZONE_SECURE) && !defined(SL_TRUSTZONE_PERIPHERAL_SEMAILBOX_HOST_S))
    || (defined(SL_TRUSTZONE_PERIPHERAL_SEMAILBOX_HOST_S) && (SL_TRUSTZONE_PERIPHERAL_SEMAILBOX_HOST_S != 0)))
#define SEMAILBOX_HOST_BASE    (SEMAILBOX_S_HOST_BASE)       /* SEMAILBOX_HOST base address */
#else
#define SEMAILBOX_HOST_BASE    (SEMAILBOX_S_HOST_BASE)       /* SEMAILBOX_HOST base address */

```

**Notes**:

- The CMSE compiler option of GCC is in the Other flagswindow under C/C++ Build → Settings → Tool Settings → GNU ARM C Compiler→ Miscellaneous.

  ![gcc_cmse](./resources/sld717-gcc-cmse.png)

- The CMSE compiler option of IAR is in the Command line options: (one per line) window under Options... → C/C++ Compiler → Extra Options.

  ![iar_cmse](./resources/sld717-iar-cmse.png)

## Linker File

The [`template_contribution`](https://siliconlabs.github.io/slc-specification/latest/format/component/template_contribution/) defined in the [slcp](https://siliconlabs.github.io/slc-specification/latest/format/project/) files of [Secure and Non-secure projects](./07-r-ex.md) will override the default memory settings defined in the device component files (e.g., `efr32mg21b020f1024im32.slcc`) to generate the linker files for [Secure](./07-r-ex.md#) and [Non-secure](./07-r-ex.md#) applications.

<table class="classic"><colgroup><col width="33%"/><col width="33%"/><col width="34%"/></colgroup><thead><tr><th>Memory Region</th><th>Default Setting in Device Component File</th><th>Override Setting in template_contribution</th></tr></thead><tbody><tr><td><p>Flash start address</p></td><td><p>device_flash_addr</p></td><td><p>memory_flash_start</p></td></tr><tr><td><p>Flash size</p></td><td><p>device_flash_size</p></td><td><p>memory_flash_size</p></td></tr><tr><td><p>RAM start address</p></td><td><p>device_ram_addr</p></td><td><p>memory_ram_start</p></td></tr><tr><td><p>RAM size</p></td><td><p>device_ram_size</p></td><td><p>memory_ram_size</p></td></tr></tbody></table>

![linker_file](./resources/sld717-linker-file.png)

The [ESAU](./03-r-bls.md#) sets the flash and RAM start address, so these addresses should be alignment at __4 kB__ (0x1000). The Secure project linker file needs to have a section for [NSC](./02-r-basic.md#) (Secure Gateway) at the end of the Secure flash section. The [SAU](./03-r-bls.md#) sets the start address of the NSC section, so this section only needs to be __32 bytes__ aligned.

- GCC NSC: The `.gnu.sgstubs` region in the Secure application map file (.map)

- IAR NSC: The `Veneer$$CMSE` region in the Secure application map file (.map)

The Secure and Non-secure flash and RAM sizes are incremented or decremented in __4 kB__. The memory configurations in Secure and Non-secure applications are correlated, so the flash and RAM settings are in pairs.

![mem_config](./resources/sld717-mem-config.png)

>**Note**: Users should not directly edit the `template_contribution` in the `slcp` file, but rather use the [Memory Editor](./07-r-ex.md#) in Simplicity Studio to update the memory configuration.

## Debugger

Simplicity Studio supports two [debuggers](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-testing-and-debugging/using-the-debugger):

- GNU Debugger (GDB) client and SEGGER's GDB server

- Simplicity Studio Debugger

The [TrustZone-unaware](#startup-code) and [TrustZone-aware](./05-r-implementation.md#) applications enable the [`PPUSEC`](./03-r-bls.md#) interrupts in the SMU. The debugger will trigger the `SMU_SECURE_IRQHandler` if the __[Registers]__ or __[Peripherals]__ view feature violates peripheral security access permission.

### Simplicity Studio Debugger

The __[Registers]__ view of Simplicity Studio Debugger can only access the Secure location of a peripheral. The following figure demonstrates the `Default_Handler` (`SMU_SECURE_IRQHandler` not defined) is triggered (`PPUSEC` in `SMU->IF` = 1) when viewing the registers of GPIO peripheral (`PPUFSPERIPHID` = 13) that is set to Non-secure access in the SMU.

The debugger can access the registers of the SMU since this peripheral is set to Secure access in the SMU.

This limitation does not apply to __GSDK \< v4.1.0__ since no peripherals are configured for Non-secure access.

![ppusec_fault](./resources/sld717-ppusec-fault.png)

The Simplicity Studio Debugger is not the preferred choice for TrustZone debugging since it has limitations on viewing Non-secure access peripherals.

### GNU Debugger (GDB)

The __[Peripherals]__ view of GNU Debugger can access either the Secure or Non-secure location of the peripheral to avoid conflicts on security access permission. The following figure shows the registers of GPIO on Secure (`GPIO` at `0x4003C000`) and Non-secure (`GPIO_NS` at `0x5003C000`) addresses. The GPIO peripheral is set to Non-secure access in the SMU, so the registers in the Secure address are displayed as zero.

![register-view](./resources/sld717-register-view.png)

The GNU Debugger is the preferred choice for TrustZone debugging and is the default debugger for Simplicity Studio ≥ v5.5.0.0.
