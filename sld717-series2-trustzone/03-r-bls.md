# Bus Level Security (BLS)

## System Design

The following figure shows two system designs:

- The sample system contains an ARMv8-M processor and the required components to support TrustZone.

- Bus Level Security (BLS) on Series 2 devices implements the concepts introduced in the ARM TrustZone sample system. BLS enforces Secure and privileged programming models and uses security components (colored blocks) to configure the security attribute and privileged level of peripherals and Bus Masters.

![ARMv8-M TrustZone Implementation](./resources/sld717-armv8-m-trustzone-implementation.png)

### ARMv8-M Processor

The ARMv8M processor is TrustZone capable of Secure and Non-secure states. It has a dedicated internal [SAU](#security-attribution-unit) that is fully programmable up to 8 different memory regions. Out of reset, the processor is in a Secure state and every transaction is a Secure transaction.

ARMv8-M Processor in Series 2 devices is the Cortex-M33.

### System Security Controller

The system security controller is the central location for all security settings in the system. Each type of controller, IDAU, and wrapper receives its security configuration and bus response configuration from this block.

System Security Controller in Series 2 devices is the [Security Management Unit (SMU)](#security-management-unit-smu).

### Implementation Defined Attribution Unit (IDAU)

The [IDAU](./02-r-basic.md#) generates the security attribute for a given address. All IDAUs in the system have the same memory partitioning. The IDAU is intended only for ARMv8-M cores and utilizes the entire IDAU interface for the core. The lite IDAU uses only the Secure and Non-secure interface from the IDAU and is intended for Non-ARMv8-M Bus Masters.

IDAU in Series 2 devices is the [External Secure Attribution Unit (ESAU)](#external-secure-attribution-unit-esau).

### Security Wrapper

The Security Wrapper gives a legacy Bus Master the ability to drive security attribution. The security wrapper outputs the transaction address to the lite IDAU which returns the security attribute of the address. If the wrapper is configured as Non-secure, any transactions to a Secure address are blocked.

Security Wrapper in Series 2 devices is the [Bus Master Protect Unit (BMPU)](#bus-master-protection-unit-bmpu).

### Memory Protection Controller (MPC)

MPC has a security configuration for a per block of memory or memory above and below the watermark. If the security attribute of the block or memory region does not match the security attribute of the address, the transaction is blocked. This controller is used in a system that alias RAM or flash memory locations. This controller is not needed when the memory region size is programmable in an IDAU.

Series 2 devices have a programmable flash and RAM region in the [ESAU](#external-secure-attribution-unit-esau) (equivalent to IDAU) and are not implementing this block.

### Peripheral Protection Controller (PPC)

PPC has a security configuration for every peripheral. If the security attribute of the selected peripheral does not match the security attribute of the address, the transaction is blocked. This controller is used in systems that alias the peripheral memory locations.

PPC in Series 2 devices is the [Peripheral Protection Unit (PPU)](#peripheral-protection-unit-ppu).

Hardware security is now extended to the peripheral bus system of the processor. Each component on the bus can verify and propagate the security level for each bus operation.  The following sections describe the individual security component for BLS on Series 2 devices.

## Security Management Unit (SMU)

The SMU is the only user-facing block in the BLS architecture and houses all the configuration and status for the [ESAUs, BMPUs, and PPUs](#bus-level-security-bls).

- Thirteen memory regions ([ESAU](#external-secure-attribution-unit-esau))

- Per Bus Master privileged and security attribute ([BMPU](#bus-master-protection-unit-bmpu))

- Interrupt flag for Bus Master security fault (fault table in BMPU section)

- Per peripheral privileged and security attribute ([PPU](#peripheral-protection-unit-ppu))

- Interrupt flags for privileged, security, and instruction peripheral access faults (fault tables in PPU section)

- Separate Secure and Privileged IRQ

The SMU configurations can be [locked](./04-r-programming.md#) down and protected from runaway code. The `SMU_LOCK` register resets to UNLOCK. Any write other than the unlock code (`0xACCE55`) locks all SMU registers from further updates. The `SMU_STATUS` register contains a `SMULOCK` bitfield with the current lock state of the SMU.

The `SMU_M33CTRL` register can [lock](./04-r-programming.md#) down internal security and privileged configurations below.

- Cortex-M33 SAU

- Non-secure MPU

- Secure MPU

- Non-secure Vector Table Offset Register (VTOR)

- Secure AIRCR register

Interrupt flags in the `SMU_IF` register can generate a [Secure or Privileged interrupt](./04-r-programming.md#) in the table below when its corresponding interrupt enable bit in the `SMU_IEN` register is set and `IRQn` is enabled.

<table class="classic"><colgroup><col width="33%"/><col width="33%"/><col width="34%"/></colgroup><thead><tr><th>Enable Bit in SMU_IEN Register</th><th>IRQn</th><th>Interrupt Handler</th></tr></thead><tbody><tr><td><p>BMPUSEC, PPUSEC</p></td><td><p>SMU_SECURE_IRQn</p></td><td><p>SMU_SECURE_IRQHandler()</p></td></tr><tr><td><p>PPUPRIV, PPUINST</p></td><td><p>SMU_PRIVILEGED_IRQn</p></td><td><p>SMU_PRIVILEGED_IRQHandler()</p></td></tr></tbody></table>

Each interrupt flag in the `SMU_IF` register can be cleared by writing 1 to the corresponding bit of the `SMU_IF_CLR` register.

## External Secure Attribution Unit (ESAU)

The ESAU is responsible for determining the memory region and [security attribute](./02-r-basic.md#) of a given address. Referring to [ARMv8-M TrustZone Implementation on page 16](./03-r-bls.md#system-design), the Cortex-M33 interfaces with an ESAU  and the BMPUs of other Bus Masters interface with lite ESAUs to determine the security attribute of all transactions. The following figure describes the security attributes of different memory regions defined by the ESAU on Series 2 devices.

![System Memory Map of Series 2 Device with TrustZone](./resources/sld717-system-memory-map-of-series-2-device-with-trustzone.png)

**Notes**:

- For Series 2 devices with base address `0x08000000` in region 0, the memory address from `0x0` to `0x07FFFFFF` is an invalid region.

- The invalid regions are deemed as Secure.

- The NSC and Exempted attributes are only available to the ESAU, and all lite ESAUs in the system view these attributes as [Secure](./05-r-implementation.md#).

The ESAU divides the memory map into 13 memory regions and has a maximum of 6 Non-secure regions.

- Four Movable Region Boundaries (MRBs) determine the size of 6 regions.

- Two regions have configurable security attributes.

- Each memory region consists of a base address that specifies the start of the region and a limit address that specifies the end of the region plus one (__+ 1__).

- The address is valid if it falls between the base (≥ base) and limit (\< limit) of a region.

- If the memory region is not defined, it is deemed invalid and Secure.

The MRBs distinguish the Secure, Non-secure Callable, and Non-secure regions in flash and RAM. The two configurable regions determine if the Info flash and Cortex-M33 [EPPB](./02-r-basic.md#) regions are Secure or Non-secure. The MRBs have a specific programming sequence. Any [misprogramming](./04-r-programming.md#) results in a `SMUPRGERR` in the `SMU_STATUS` register.

### ARMv8-M CODE Regions

- [Regions 0, 1, and 2](./04-r-programming.md#) are in the Main space of flash. Region 3 is the info space of flash.

- The mrb01 (`ESAUMRB01` in `SMU_ESAURMBR01` register) determines the end of region 0 and the start of region 1.

- The mrb12 (`ESAUMRB12` in `SMU_ESAURMBR12` register) determines the end of region 1 and the start of region 2.

- The size of region 3 is device-dependent.

- Three regions' security attributes are static, and one region is configurable. Region 0 is always Secure, region 1 is always Non-secure Callable, and region 2 is always Non-secure. [Region 3](./04-r-programming.md#) is configurable as either Secure or Non-secure (`ESAUR3NS` in `SMU_ESAURTYPES0` register, default is secure after reset).

- Sizes of regions 0, 1, and 2 are adjusted in __4 kB increments__ with the lower 12 bits of `ESAUMRB##` in `SMU_ESAURMBR##` ignored.

  - The Secure region can be set to size 0 when mbr01 = base address of region 0.

  - The Non-secure Callable regions can be set to size 0 when mbr01 = mbr12.

- The default value of mbr01 is equal to base address + `0x02000000`, so the size of region 0 is 32 MB. Out of reset, all flash is Secure since all Series 2 devices have less than 32 MB of flash.

<table class="classic"><colgroup><col width="15%"/><col width="20%"/><col width="25%"/><col width="25%"/><col width="15%"/></colgroup><thead><tr><th>Region</th><th>Memory</th><th>Base Address</th><th>Limit Address</th><th>Security Attribute</th></tr></thead><tbody><tr><td><p>0</p></td><td><p>Main flash</p></td><td><p>0x00000000 or 0x08000000</p></td><td><p>(0x00000000 or 0x08000000) | mbr01</p></td><td><p>Secure</p></td></tr><tr><td><p>1</p></td><td><p>Main flash</p></td><td><p>(0x00000000 or 0x08000000) | mbr01</p></td><td><p>(0x00000000 or 0x08000000) | mbr12</p></td><td><p>Non-secure Callable</p></td></tr><tr><td><p>2</p></td><td><p>Main flash</p></td><td><p>(0x00000000 or 0x08000000) | mbr12</p></td><td><p>0x0FE00000</p></td><td><p>Non-secure</p></td></tr><tr><td><p>3</p></td><td><p>Info flash</p></td><td><p>0x0FE0000</p></td><td><p>0x10000000</p></td><td><p>Secure or Non-secure</p></td></tr></tbody></table>

### ARMv8-M RAM Regions

- [Regions 4, 5, and 6](./04-r-programming.md#) cover the entire available RAM in the device.

- The mrb45 (`ESAUMRB45` in `SMU_ESAURMBR45` register) determines the end of region 4 and the start of region 5.

- The mrb56 (`ESAUMRB56` in `SMU_ESAURMBR56` register) determines the end of region 5 and the start of region 6.

- All three regions' security attributes are static. Region 4 is always Secure, region 5 is always Non-secure Callable, and region 6 is always Non-secure.

- Sizes of all three regions are adjusted in __4 kB increments__ with the lower 12 bits of `ESAUMRB##` in `SMU_ESAURMBR##` ignored.

  - The Secure region can be set to size 0 when mbr45 = base address of region 4.

  - The Non-secure Callable region can be set to size 0 when mbr45 = mbr56.

- The default value of mbr45 is equal to `0x02000000`, so the size of region 4 is 32 MB. Out of reset, all RAM is Secure since all Series 2 devices have less than 32 MB of RAM.

<table class="classic"><colgroup><col width="15%"/><col width="20%"/><col width="25%"/><col width="25%"/><col width="15%"/></colgroup><thead><tr><th>Region</th><th>Memory</th><th>Base Address</th><th>Limit Address</th><th>Security Attribute</th></tr></thead><tbody><tr><td><p>4</p></td><td><p>SRAM</p></td><td><p>0x20000000</p></td><td><p>0x20000000 | mbr45</p></td><td><p>Secure</p></td></tr><tr><td><p>5</p></td><td><p>SRAM</p></td><td><p>0x20000000 | mbr45</p></td><td><p>0x20000000 | mbr56</p></td><td><p>Non-secure Callable</p></td></tr><tr><td><p>6</p></td><td><p>SRAM</p></td><td><p>0x20000000 | mbr56</p></td><td><p>0x30000000</p></td><td><p>Non-secure</p></td></tr></tbody></table>

### ARMv8-M Peripheral Regions

- These regions are aliases to the [chip peripherals and SE mailbox](./04-r-programming.md) (a device with HSE).

- Both regions have a fixed size.

- Both regions' security attributes are static. Region 7 is always Secure, and region 8 is always Non-secure.

<table class="classic"><colgroup><col width="15%"/><col width="20%"/><col width="25%"/><col width="25%"/><col width="15%"/></colgroup><thead><tr><th>Region</th><th>Memory</th><th>Base Address</th><th>Limit Address</th><th>Security Attribute</th></tr></thead><tbody><tr><td><p>7</p></td><td><p>Chip Peripherals</p></td><td><p>0x40000000</p></td><td><p>0x50000000</p></td><td><p>Secure</p></td></tr><tr><td><p>8</p></td><td><p>Chip Peripherals</p></td><td><p>0x50000000</p></td><td><p>0x60000000</p></td><td><p>Non-secure</p></td></tr></tbody></table>

### ARMv8-M Device Regions

- These regions are aliases to all radio peripherals and radio RAM.

- Both regions have a fixed size.

- Both regions' security attributes are static. Region 9 is always Secure, and region 10 is always Non-secure.

- From the perspective of the device bus system, the [radio is one peripheral that is either Secure or Non-secure](./04-r-programming.md#). So any Bus Master accessing the radio needs to know the security attribute of the radio. From the perspective of the radio, all of its radio bus peripherals are accessible regardless of the security attribute. However, the radio needs to know the security attribute of chip bus peripherals to access them through the correct alias.

<table class="classic"><colgroup><col width="15%"/><col width="20%"/><col width="25%"/><col width="25%"/><col width="15%"/></colgroup><thead><tr><th>Region</th><th>Memory</th><th>Base Address</th><th>Limit Address</th><th>Security Attribute</th></tr></thead><tbody><tr><td><p>9</p></td><td><p>Radio Peripherals</p></td><td><p>0xA0000000</p></td><td><p>0xB0000000</p></td><td><p>Secure</p></td></tr><tr><td><p>10</p></td><td><p>Radio Peripherals</p></td><td><p>0xB0000000</p></td><td><p>0xC0000000</p></td><td><p>Non-secure</p></td></tr></tbody></table>

### ARMv8-M System Private Peripheral Bus (PPB) Regions

- Both regions have a fixed size.

- [Region 11](./04-r-programming.md) is the Cortex-M33 EPPB memory region and is configurable as either Secure or Non-secure (`ESAUR11NS` in `SMU_ESAURTYPES1` register, default is secure after reset). It is important to note that the Cortex-M33 core is the only Bus Master that sees these memory regions. All other Bus Masters in the system do not have access to the System PPB, and it is an invalid region.

- Region 12 has a static security attribute of Exempted. It means that the Cortex-M33 core allows the transaction in all cases. It permits debuggers to read the system ROM Table regardless of the state of the Cortex-M33 core.

<table class="classic"><colgroup><col width="15%"/><col width="20%"/><col width="25%"/><col width="25%"/><col width="15%"/></colgroup><thead><tr><th>Region</th><th>Memory</th><th>Base Address</th><th>Limit Address</th><th>Security Attribute</th></tr></thead><tbody><tr><td><p>11</p></td><td><p>EPPB</p></td><td><p>0xE0044000</p></td><td><p>0xE00FE000</p></td><td><p>Secure or Non-secure</p></td></tr><tr><td><p>12</p></td><td><p>System ROM Table</p></td><td><p>0xE00FE000</p></td><td><p>0xE00FF000</p></td><td><p>Exempted</p></td></tr></tbody></table>

**Notes**:

- The regions in flash (0/1/2) and RAM (4/5/6) can only create in the order of Secure, Non-secure Callable, and Non-secure.

- The [ESAU and lite ESAUs](#bus-level-security-bls) handle the transactions of Bus Masters and must have consistent security attribute mapping. Therefore, configurations in the SMU registers apply to ESAU and lite ESAUs.

- Unlike other Bus Masters using BMPU and lite ESAU, merging the address lookup results from the [internal SAU and ESAU](./02-r-basic.md#) determines the [security attribute](#security-attribution-unit) of the Cortex-M33 transaction.

<table class="classic"><colgroup><col width="50%"/><col width="50%"/></colgroup><thead><tr><th>Bus Master</th><th>Security Attribution</th></tr></thead><tbody><tr><td><p>Cortex-M33</p></td><td><p>SAU and ESAU</p></td></tr><tr><td><p>Other</p></td><td><p>Lite ESAU</p></td></tr></tbody></table>

## Security Attribution Unit

In Series 2 devices, the combination of the integrated SAU in the Cortex-M33 processor and an ESAU determine the security attribute of a Cortex-M33 transaction.

The SAU consists of several [programmable registers](https://developer.arm.com/documentation/100690/0201/SAU-register-summary). These registers are placed in the [System Control Space (SCS)](./02-r-basic.md) and are only accessible from the Secure privileged state.

- SAU Control Register (`SAU_CTRL`) — The SAU is disabled after RESET

- SAU Type Register (`SAU_TYPE`) — Indicates the number of [available regions](./01-series-2-security-features.md) (read-only)

- SAU Region Number Register (`SMU_RNR`) — Assigns a region number

- SAU Region Base Address Register (`SAU->RBAR`) — Configures selected region base address

- SAU Region Limit Address Register (`SAU->RLAR`) — Configures selected region limit address and security attribute (NSC or NS), enable or disable the region

The following figure shows three different SAU configurations for determining the security attribute of a Cortex-M33 transaction.

![Configuration of SAU_CTRL Register](./resources/sld717-configuration-of-sau-ctrl-register.png)

**Notes**:

- All address ranges after RESET in SAU are Secure by default.

- The SAU can configure a **32 bytes aligned** region as Non-secure or Non-secure Callable. Any address not defined in the SAU defaults to Secure.

- An [ESAU](#external-secure-attribution-unit-esau) can configure or hard-code a region as Secure, Non-secure Callable, Non-secure, or Exempted. An [Exempted](./02-r-basic.md#) region enables Non-secure debuggers to access debugging components and establish a debug connection to the processor before the SAU is configured.

- The processor determines the final attribute of the address based on the higher security attribute (Exempted > S > NSC > NS) from either the SAU or the ESAU.

  ![state_hierarchy](./resources/sld717-state-hierarchy.png)

### All Secure Configuration

Highlights:

- SAU is disabled.

- `ALLNS` bit in the SAU Control register is clear.

- The whole memory is in a Secure state (highest security attribute apart from Exempted).

- All Cortex-M33 transactions in this configuration are Secure or Exempted and give the Cortex-M33 access to all memory locations through either the Secure or Non-secure alias after RESET.

  ![all_s](./resources/sld717-all-s.png)

- It is up to the boot procedure in a Secure state to keep the current configuration or use other configurations once the boot process is complete.

### All Non-secure Configuration

Highlights:

- SAU is disabled.

- `ALLNS` bit in the SAU Control register is set.

- The whole memory is in a Non-secure state (lowest security attribute).

- Therefore the [ESAU configuration](#external-secure-attribution-unit-esau) determines the security attribute of all Cortex-M33 transactions.

  ![all_ns](./resources/sld717-all-ns.png)

- Except for the `SAU_CTRL` [register](./04-r-programming.md), this configuration does not require programming on other SAU registers.

### Configurable Configuration

Highlights:

- SAU is enabled.

- `ALLNS` bit in the SAU Control register can be 0 or 1 (do not care).

- The `NSC` bit on the `SAU_RLAR` register determines the security attribute of an address as Non-secure or Non-secure Callable if an address matches an SAU region.

- The security attribute of an address is Secure by default if the address does not match any SAU region.

- This configuration [programs](./04-r-programming.md)`SAU_RNR`, `SAU_RBAR`, and `SAU_RLAR` registers to correlate the [Non-secure regions](./03-r-bls.md#configurable-configuration) in ESAU.

- The SAU or ESAU [overrides](./02-r-basic.md) the attribute to a [higher security level](#security-attribution-unit) if any security attribute mismatch occurs in a memory region.

  ![address_lookup](./resources/sld717-address-lookup.png)

- The following figure is an example of a configurable configuration with the size of ESAU regions 0 and 5 are set to zero.

  ![all_config](./resources/sld717-all-config.png)

>**Note**: The Cortex-M33 has an internal SAU that defaults all undefined addresses to Secure if enabled. If the Secure regions do not align between the [Cortex-M33 (SAU + ESAU) and other Bus Masters (lite ESAU)](#bus-level-security-bls), the Cortex-M33 treats a memory region as Secure while other Bus Masters treat it as Non-secure. It can lead to the leaking of secure data if the Cortex-M33 stores secure data in what other Bus Masters think is a Non-secure area ([Main Flash Layout on page 34](./05-r-implementation.md#)).

## Bus Master Protection Unit (BMPU)

The BMPU is a security wrapper used for assigning a Bus Master specific security and privileged states. Referring to [Figure 3.1 ARMv8-M TrustZone Implementation on page 16](#system-design), the BMPU generally lies between the Bus Master and the Advanced High-performance Bus (AHB) Matrix. BMPU interfaces with a [lite ESAU](#bus-level-security-bls) to determine the security attribute of all Bus Master transactions.

The registers below in SMU configure the [security](./04-r-programming.md) and [privileged](./04-r-programming.md) state of a Bus Master. The Bus Masters in group 0 are device-dependent. Out of reset, each Bus Master is Secure and privileged.

<table class="classic"><colgroup><col width="30%"/><col width="auto"/></colgroup><thead><tr><th>Register</th><th>Description</th></tr></thead><tbody><tr><td><p>SMU_BMPUPATD0</p></td><td><p>Bitfields (privileged if set) for privileged attribute configuration on Bus Master group 0</p></td></tr><tr><td><p>SMU_BMPUSATD0</p></td><td><p>Bitfields (Secure if set) for security attribute configuration on Bus Master group 0</p></td></tr></tbody></table>

>**Note**: The Bus Master privileged attribute only applies to peripheral accesses. Flash and RAM accesses ignore the privileged attribute of the Bus Master.

The BMPU generates a security fault when the security attribute of the bus transaction is Secure, and the security attribute (`SMU_BMPUSATD0`) for the BMPU is configured as Non-secure.

Below is the security fault table that shows how the security attribute on the bus is driven based on the lite ESAU attribute and the BMPU security configuration. The interrupt is triggered if `BMPUSEC` in `SMU_IEN` is set and the `SMU_SECURE_IRQn` is enabled.

<table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th>Lite ESAU Attribute</th><th>Secure Bus Master</th><th>Non-secure Bus Master</th></tr></thead><tbody><tr><td><p>Non-secure</p></td><td><p>Non-secure</p></td><td><p>Non-secure</p></td></tr><tr><td><p>Secure</p></td><td><p>Secure</p></td><td><p>FAULT</p></td></tr></tbody></table>

Upon a BMPU fault, the registers in SMU below notify that a BMPU security fault occurred and on which Bus Master. The registers also identify the offending fault address. If a fault is detected, the response is Read As Zero (RAZ) or Write Ignored (WI) and the corresponding interrupt flag is set in the `SMU_IF` register. The values in `SMU_BMPUFS` and `SMU_BMPUFSADDR` do not change until the BMPU fault (`BMPUSEC`) in the `SMU_IF` register is cleared by software.

<table class="classic"><colgroup><col width="25%"/><col width="35%"/><col width="40%"/></colgroup><thead><tr><th>Register</th><th>Bitfield</th><th>Fault</th></tr></thead><tbody><tr><td><p>SMU_IF</p></td><td><p>BMPUSEC</p></td><td><p>Security Fault if set</p></td></tr><tr><td><p>SMU_BMPUFS</p></td><td><p>BMPUFSMASTERID</p></td><td><p>ID of the Bus Master that triggered the fault</p></td></tr><tr><td><p>SMU_BMPUFSADDR</p></td><td><p>BMPUFSADDR</p></td><td><p>Access address that triggered the fault</p></td></tr></tbody></table>

>**Note**: No privileged fault is generated because all the other Bus Masters in the system do not drive the privileged attribute.

## Peripheral Protection Unit (PPU)

The PPU is a security wrapper used for assigning a Bus Slave peripheral specific security and privileged states. Referring to [Figure
3.1 ARMv8-M TrustZone Implementation on page 16](#system-design), the PPU comes in the form of a PPU in Advanced High-performance Bus (AHB) and a PPU in Advanced Peripheral Bus (APB).

- The PPU AHB generally lies between the Bus Matrix and an AHB Bus Slave peripheral.

- The PPU APB lies between the output of an AHB to APB bridge and all of the APB Slaves on that APB bus.

The registers below in SMU configure the [security](./04-r-programming.md#) and [privileged](./04-r-programming.md#) state of a peripheral. The peripherals in groups 0 and 1 are device-dependent. Out of reset, each peripheral is Secure and privileged. While each peripheral in address `0x40000000` (region 7) or `0x50000000` (region 8) can be configured independently, the radio subsystem in `0xA0000000` (region 9) or `0xB0000000` (region 10) is configured as a [unit](./04-r-programming.md#).

<table><tbody><tr><th>Register</th><th>Description</th></tr><tr><td>SMU_PPUPATD0</td><td>Bitfields (privileged if set) for privileged access configuration on peripheral group 0</td></tr><tr><td>SMU_PPUPATD1</td><td>Bitfields (privileged if set) for privileged access configuration on peripheral group 0</td></tr><tr><td>SMU_PPUSATD0</td><td>Bitfields (Secure if set) for security access configuration on peripheral group 0</td></tr><tr><td>SMU_PPUSATD1</td><td>Bitfields (Secure if set) for security access configuration on peripheral group 1</td></tr></tbody></table>

The PPU can generate three types of faults:

1. Privileged faults occur on unprivileged transactions to privileged peripherals. Below is the privileged fault table that shows when a privileged fault occurs based on the PPU peripheral privileged configuration and the bus transaction privileged attribute. The interrupt is triggered if `PPUPRIV` in `SMU_IEN` is set and the `SMU_PRIVILEGED_IRQn` is enabled.

   <table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th>Bus Attribute</th><th>Privileged Peripheral</th><th>Unprivileged Peripheral</th></tr></thead><tbody><tr><td><p>Privileged</p></td><td><p>SUCCESS</p></td><td><p>SUCCESS</p></td></tr><tr><td><p>Unprivileged</p></td><td><p>FAULT</p></td><td><p>SUCCESS</p></td></tr></tbody></table>

2. Security faults occur on Secure transactions to Non-secure peripherals and Non-secure transactions to Secure peripherals. Below is the security fault table that shows when a security fault occurs based on the PPU Peripheral security configuration and the bus transaction security attribute. The interrupt is triggered if `PPUSEC` in `SMU_IEN` is set and the `SMU_SECURE_IRQn` is enabled.

    <table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th>Bus Attribute</th><th>Secure Peripheral</th><th>Non-secure Peripheral</th></tr></thead><tbody><tr><td><p>Secure</p></td><td><p>SUCCESS</p></td><td><p>FAULT</p></td></tr><tr><td><p>Non-secure</p></td><td><p>FAULT</p></td><td><p>SUCCESS</p></td></tr></tbody></table>

3. Instruction faults occur on any transaction marked as an instruction fetch. Below is the instruction fault table that shows when a PPU instruction fault occurs based on the bus transaction instruction attribute. The interrupt is triggered if `PPUINST` in `SMU_IEN` is set and the `SMU_PRIVILEGED_IRQn` is enabled.

    <table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th>Bus Attribute</th><th>Secure Peripheral</th><th>Non-secure Peripheral</th></tr></thead><tbody><tr><td><p>Secure</p></td><td><p>SUCCESS</p></td><td><p>FAULT</p></td></tr><tr><td><p>Non-secure</p></td><td><p>FAULT</p></td><td><p>SUCCESS</p></td></tr></tbody></table>

Upon a [PPU fault](./04-r-programming.md#), the registers below in [SMU](#security-management-unit-smu) notifies which PPU fault occurred and on which peripheral. If a fault is detected, the response is Read As Zero (RAZ) or Write Ignored (WI) and set the corresponding interrupt flag in the `SMU_IF` register. The values in `SMU_IF` and `SMU_PPUFS` do not change until all PPU faults in the `SMU_IF` register are cleared by software.

<table class="classic"><colgroup><col width="25%"/><col width="35%"/><col width="40%"/></colgroup><thead><tr><th>Register</th><th>Bitfield</th><th>Fault</th></tr></thead><tbody><tr><td><p>SMU_IF</p></td><td><p>PPUPRIV</p></td><td><p>Privilege Fault if set</p></td></tr><tr><td><p>SMU_IF</p></td><td><p>PPUSEC</p></td><td><p>Security Fault if set</p></td></tr><tr><td><p>SMU_IF</p></td><td><p>PPUINST</p></td><td><p>Instruction Fault if set</p></td></tr><tr><td><p>SMU_PPUFS</p></td><td><p>PPUFSPERIPHID</p></td><td><p>ID of the peripheral that caused the fault</p></td></tr></tbody></table>

## Compatibility

Secure software usually controls the [SYSCFG and SMU](./05-r-implementation.md) peripherals to prevent Non-secure software from changing critical configurations in the Secure domain. It requires switching between Secure and Non-secure states when Non-secure software wants to update the registers in these peripherals. Therefore dedicated registers for Non-secure access are added to SYSCFG and SMU peripherals on newer Series 2 devices.

### System Configuration (SYSCFG)

Except for EFR32xG21 devices, the following tables apply to all Series 2 devices.

**Table**: Dedicated Bitfield to Configure Access for Non-secure SYSCFG Registers

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>Bitfield (Register)</th><th>Description</th></tr></thead><tbody><tr><td><p>SYSCFGCFGNS (SMU_PPUPATD0)</p></td><td><p>Bitfields (privileged if set) for privileged access configuration on NS SYSCFG registers</p></td></tr><tr><td><p>SYSCFGCFGNS (SMU_PPUSATD0)</p></td><td><p>Bitfields (Secure if set) for security access configuration on NS SYSCFG registers</p></td></tr></tbody></table>

>**Note**: Reset `SYSCFGCFGNS` bit in `SMU_PPUSATD0` to allow Non-secure software to access NS SYSCFG registers.

**Table**: Dedicated SYSCFG Registers for Non-secure State

<table class="classic"><colgroup><col width="50%"/><col width="50%"/></colgroup><thead><tr><th>SYSCFG Non-secure Registers</th><th>Description</th></tr></thead><tbody><tr><td><p>SYSCFG_CFGNS_CFGNSTCALIB</p></td><td><p>NS SysTick calibration value register</p></td></tr><tr><td><p>SYSCFG_CFGNS_ROOTNSDATA0</p></td><td><p>NS root data register 0</p></td></tr><tr><td><p>SYSCFG_CFGNS_ROOTNSDATA0</p></td><td><p>NS root data register 1</p></td></tr></tbody></table>

### Security Management Unit (SMU)

Except for EFR32xG21 devices, the following tables apply to all Series 2 devices.

**Table**: Dedicated Bitfield to Configure Access for Non-secure SMU Registers

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>Bitfield (Register)</th><th>Description</th></tr></thead><tbody><tr><td><p>SMUCFGNS (SMU_PPUPATD1)</p></td><td><p>Bitfields (privileged if set) for privileged access configuration on NS SMU registers</p></td></tr><tr><td><p>SMUCFGNS (SMU_PPUSATD1)</p></td><td><p>Bitfields (Secure if set) for security access configuration on NS registers</p></td></tr></tbody></table>

>**Note**: Reset `SMUCFGNS` bit in `SMU_PPUSATD1` to allow Non-secure software to access NS SMU registers.

The SMU_CFGNS register file is for the TrustZone Non-secure state and has its register lock (`NSLOCK`). It allows hardware to maintain the privileged assignments for the NS state. The privileged configuration within the NS state is the same as the Secure state, except it has an "NS" to differentiate the registers.

**Table**: Dedicated SMU Registers for Non-secure State

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>SMU Non-secure Registers</th><th>Description</th></tr></thead><tbody><tr><td><p>SMU_CFGNS_NSSTATUS</p></td><td><p>Lock status of SMU_CFGNS registers</p></td></tr><tr><td><p>SMU_CFGNS_NSLCOK</p></td><td><p>Lock and unlock the SMU_CFGNS registers</p></td></tr><tr><td><p>SMU_CFGNS_NSIF</p></td><td><p>Interrupt flags for NS privilege (PPUNSPRIVIF) and instruction (PPUNSINSTIF) faults</p></td></tr><tr><td><p>SMU_CFGNS_NSIEN</p></td><td><p>Interrupt enable flags for NS privilege (PPUNSPRIVIEN) and instruction (PPUNSINSTIEN) faults</p></td></tr><tr><td><p>SMU_CFGNS_PPUNSPATD0</p></td><td><p>Bitfields (privileged if set) for NS privileged access configuration on peripheral group 0</p></td></tr><tr><td><p>SMU_CFGNS_PPUNSPATD1</p></td><td><p>Bitfields (privileged if set) for NS privileged access configuration on peripheral group 1</p></td></tr><tr><td><p>SMU_CFGNS_PPUNSFS</p></td><td><p>ID (PPUFSPERIPHID) of the NS peripheral that caused the fault</p></td></tr><tr><td><p>SMU_CFGNS_BMPUNSPATD0</p></td><td><p>Bitfields (privileged if set) for privileged attribute configuration on NS Bus Master group 0</p></td></tr></tbody></table>

**Table**: Fault Statuses Only for Secure State

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>Bitfield (Register)</th><th>Description</th></tr></thead><tbody><tr><td><p>PPUPRIV (SMU_IF)</p></td><td><p>Fault status now limited only to Secure state</p></td></tr><tr><td><p>PPUINST (SMU_IF)</p></td><td><p>Fault status now limited only to Secure state</p></td></tr><tr><td><p>PPUPRIV (SMU_IEN)</p></td><td><p>Fault status now limited only to Secure state</p></td></tr><tr><td><p>PPUINST (SMU_IEN)</p></td><td><p>Fault status now limited only to Secure state</p></td></tr><tr><td><p>PPUFSPERIPHID (SMU_PPUFS)</p></td><td><p>Fault status now limited only to Secure state</p></td></tr></tbody></table>

**Table** Dedicated SMU Interrupt for Non-secure State

<table class="classic"><colgroup><col width="40%"/><col width="60%"/></colgroup><thead><tr><th>Interrupt</th><th>Description</th></tr></thead><tbody><tr><td><p>SMU_NS_PRIVILEGED_IRQHandler()</p></td><td><p>An interrupt flag in the SMU_CFGNS_NSIF register can generate an NS privileged interrupt when its corresponding interrupt enable bit in the SMU_CFGNS_NSIEN register is set and SMU_NS_PRIVILEGED_IRQn is enabled, and in which the peripheral (ID) that triggers the fault is in the SMU_CFGNS_PPUNSFS register.</p></td></tr></tbody></table>
