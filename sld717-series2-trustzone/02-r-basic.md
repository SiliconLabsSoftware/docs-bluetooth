# TrustZone Basics

## Introduction

TrustZone for ARMv8-M adds extra states to the Cortex-M processor operations to ensure there is  a Secure and Non-secure state. These security states are orthogonal to the existing Thread and Handler modes, thereby having both a Thread and Handler mode in both Secure and Non-secure states. The Thread mode can also be either Privileged or Unprivileged.

![Operation States and Modes of TrustZone Implementation](./resources/sld717-operation-states-and-modes-of-trustzone-implementation.png)

*Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

TrustZone for ARMv8-M is an optional architecture extension. By default, the system starts up in a Secure state if the processor implements the TrustZone security extension. The division of Secure and Non-secure worlds is memory-map based (security state depends on the address of the fetched instruction), and the transitions happen automatically. It is also possible to leave the Non-secure state unused and execute the whole application in the Secure state.

## Memory Security Attributes

TrustZone classifies memory into four security attributes as described in the following table.

<table class="classic"><colgroup><col width="25%"/><col width="20%"/><col width="55%"/></colgroup><thead><tr><th><p>Security Attribute</p></th><th><p>Processor State</p></th><th><p>Description</p></th></tr></thead><tbody><tr><td><p>Non-secure (NS)</p></td><td><p>Non-secure</p></td><td><p>Non-secure and Secure software can access these memory regions.</p></td></tr><tr><td><p>Secure (S)</p></td><td><p>Secure</p></td><td><p>Secure software can access these memory regions. Non-secure software cannot gain access to the Secure memory.</p></td></tr><tr><td><p>Non-secure Callable (NSC)</p></td><td><p>Secure</p></td><td><p>Secure memory with an NSC attribute provides entry points for Secure APIs that can be called from a Non-secure space. It is a region of memory that contains the Secure Gateway (SG) veneers that allow Non-secure code to call secure functions that exist in Secure code. Non-secure software cannot read/write to an NSC memory but can branch into it if the branch target is an SG instruction.</p></td></tr><tr><td><p>Exempted</p></td><td><p>Secure/Non-secure</p></td><td><p>Non-secure and Secure software can access these memory regions (exempted from security checking). Exempted regions are typically used by debugging components that do not pose any security risk (e.g., system ROM table) when accessed by the Non-secure software.</p></td></tr></tbody></table>

>**Note:** The [Non-secure Callable](#switching-from-non-secure-to-secure-state) is also known as Secure Non-secure Callable (Secure NSC) to declare that this region resides in Secure memory.

## Banked Register

The concept of a banked register in ARMv8-M between Secure and Non-secure states means that there are two copies of the register, and the core automatically uses the copy that belongs to the current security state. When a register is banked, the `_S` and `_NS` suffixes are used in the ARMv8-M architecture to identify whether the resource is for the Secure state or Non-secure state.

### General-Purpose Registers

The Cortex-M processors have 16 general-purpose registers (R0 - R15) for data processing (R0 - R12) and control. The following figure shows the general-purpose register view of the ARMv8-M system with TrustZone. Refer to the [ARM Cortex-M33 Devices Generic User Guide](https://developer.arm.com/documentation/100235/0100/The-Cortex-M33-Processor/Programmer-s-model/Core-registers) for details about these registers.

![General-Purpose Register View with TrustZone](./resources/sld717-general-purpose-register-view-with-trustzone.png)

*Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

The Secure or Non-secure state can access the data processing registers R0 - R12 and special usage registers R13 - R15. The register R13 (banked SP) is the stack pointer alias, and the actual stack pointer (`MSP_NS`, `PSP_NS`, `MSP_S`, `PSP_S`) accessed depends on the state (Secure or Non-secure) and mode (Handler or Thread) as described in the following figure.

In addition, stack limit registers ([special registers](#special-purpose-registers)) enable hardware to detect stack overflow conditions. Two pairs of [stack limit registers](#special-purpose-registers) (`MSPLIM_NS` and `PSPLIM_NS`, `MSPLIM_S` and `PSPLIM_S`) are implemented, one per security state, to protect the Main Stack Pointer (MSP) and Process Stack Pointer (PSP).

![Banked Registers in the General-Purpose Registers](./resources/sld717-banked-registers-in-the-general-purpose-registers.png)

*Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

In Thread mode, execution can be privileged or unprivileged. The stack pointer used can be the MSP or PSP, depending on the `SPSEL` bit in the [CONTROL](#special-purpose-registers) register. When in Handler mode, the processor is Privileged. The stack pointer is always MSP.

It is possible to directly [access](https://arm-software.github.io/CMSIS_5/Core/html/group__Core__Register__gr.html) the stack pointers (MSP and PSP) and stack limit registers (MSPLIM and PSPLIM), providing that the processor is in a privileged state. If the processor is in a Secure privileged state, the software can also access the Non-secure stack pointers (`MSP_NS` and `PSP_NS`) through [Core Register Access Functions](https://arm-software.github.io/CMSIS_5/Core/html/group__coreregister__trustzone__functions.html) in CMSIS-Core.

### Special-Purpose Registers

Except for the general-purpose registers, there are several special-purpose registers for conditional flags, interrupt masking, control, and stack pointer limit. The following figure shows the special-purpose registers view of the ARMv8-M system with TrustZone. Refer to the [ARM Cortex-M33 Devices Generic User Guide](https://developer.arm.com/documentation/100235/0100/The-Cortex-M33-Processor/Programmer-s-model/Core-registers) for details about these registers.

![Special-purpose Registers View with TrustZone](./resources/sld717-special-purpose-registers-view-with-trustzone.png)

Image:*[https://documentation-service.arm.com/](https://documentation-service.arm.com/static/5e7cd7b67158f500bd5c4eea?token=).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

The Combined Program Status Register (xPSR) consists of the Application Program Status Register (APSR), Interrupt Program Status Register (IPSR), and Execution Program Status Register (EPSR).

Some of the special-purpose registers are banked between Secure and Non-secure states. Special-purpose registers are not memory-mapped and can be [accessed](https://arm-software.github.io/CMSIS_5/Core/html/group__Core__Register__gr.html) using Core Register Access Functions in CMSIS-Core (except for EPSR in xPSR).

Secure privileged software can also access the Non-secure interrupt masking registers (`PRIMASK_NS`, `FAULTMASK_NS`, and `BASEPRI_NS`), CONTROL register (`CONTROL_NS`), and stack limit registers (`MSPLIM_NS` and `PSPLIM_NS`) through [Core Register Access Functions](https://arm-software.github.io/CMSIS_5/Core/html/group__coreregister__trustzone__functions.html) in CMSIS-Core.

### System Private Peripheral Bus (PPB)

The banking of registers is usually used to separate the Secure and Non-secure information of the system components inside the processor. The following figure shows the System Private Peripheral Bus (PPB) registers view of the ARMv8-M system with TrustZone. Refer to the [ARM Cortex-M33 Devices Generic User Guide](https://developer.arm.com/documentation/100235/0100/The-Cortex-M33-Peripherals/About-the-Cortex-M33-peripherals) for details about the System PPB registers.

![System Private Peripheral Bus (PPB) Registers View with TrustZone](./resources/sld717-system-private-peripheral-bus-ppb-registers-view-with-trustzone.png)

**System components for debugging and trace operations (`0xE0000000` to `0xE0002FFF`)**:

- Instrumentation Trace Macrocell (ITM)

- Data Watch point and Trace unit (DWT)

- Flash Patch and Breakpoint unit (FPB)

**System Control Space (SCS)**:

- The registers in SCS address spaces are memory-mapped and can be accessed using pointers in software

- Secure SCS (`0xE000E000` to `0xE000EFFF`) - Secure software using this address space to access the banked Secure SCS registers (e.g., `SCB->CPUID`)

- Non-secure SCS (`0xE000E000` to `0xE000EFFF`) - Non-secure software using this address space to access the banked Non-secure SCS registers (e.g., `SCB->CPUID`)

- Non-secure alias SCS (`0xE002E000` to `0xE002EFFF`) - Secure software using this address space to access the Non-secure SCS registers (e.g., `SCB_NS->CPUID`)

The following table describes some core peripherals in the SCS and corresponding [data structures](https://arm-software.github.io/CMSIS_5/Core/html/annotated.html) defined in the CMSIS-Core header file to access the registers of core peripherals in two SCS address spaces.

<table class="classic"><colgroup><col width="30%"/><col width="35%"/><col width="35%"/></colgroup><thead><tr><th><p>Core Peripheral</p></th><th><p>Data Structure for Secure and NS SCS</p></th><th><p>Data Structure for NS Alias SCS</p></th></tr></thead><tbody><tr><td><p>Implementation Control Block</p></td><td><p>SCnSCB (0xE000E004)</p></td><td><p>SCnSCB_NS (0xE002E004)</p></td></tr><tr><td><p>SysTick Timer</p></td><td><p>SysTick (0xE000E010)</p></td><td><p>SysTick_NS (0xE002E010)</p></td></tr><tr><td><p>Nested Vectored Interrupt Controller</p></td><td><p>NVIC (0xE000E100)</p></td><td><p>NVIC_NS (0xE002E100)</p></td></tr><tr><td><p>System Control Block</p></td><td><p>SCB (0xE000ECFC)</p></td><td><p>SCB_NS (0xE002ECFC)</p></td></tr><tr><td><p>Memory Protection Unit</p></td><td><p>MPU (0xE000ED90)</p></td><td><p>MPU_NS (0xE002ED90)</p></td></tr><tr><td><p>Security Attribution Unit</p></td><td><p>SAU (0xE000EDD0)</p></td><td><p>-</p></td></tr><tr><td><p>Debug Control Block</p></td><td><p>CoreDebug (0xE000EDF0)</p></td><td><p>CoreDebug_NS (0xE002EDF0)</p></td></tr><tr><td><p>Software Interrupt Generation</p></td><td><p>STIR (0xE000EF00)</p></td><td><p>STIR_NS (0xE002EF00)</p></td></tr><tr><td><p>Floating-Point Extension</p></td><td><p>FPU (0xE000EF34)</p></td><td><p>FPU_NS (0xE002EF34)</p></td></tr></tbody></table>

**Notes**:

- The SCB is a group of system control registers for the various usages below.

  - System Control Register (SCR) to configure processor low power mode

  - Fault Status Register (xFSR) to provide fault status information

  - [Vector Table Offset Register (VTOR)](#type-of-exceptions) for vector table relocation

- The [SAU](./03-r-bls.md#security-attribution-unit) register is accessible from the Secure state only.

- The STIR register is not physically banked.

- Core peripherals such as SysTick, SCB, and MPU are duplicated. One instance is Secure and the other one is Non-secure.

- Secure software can use the corresponding functions for ARMv8-M in CMSIS-Core to configure the Non-secure [NVIC](https://arm-software.github.io/CMSIS_5/Core/html/group__nvic__trustzone__functions.html) and [SysTick](https://arm-software.github.io/CMSIS_5/Core/html/group__systick__trustzone__functions.html) through the Non-secure alias SCS.

**Debug or vendor specific components (`0xE0040000` to `0xE00FFFFF`)**:

- Optional debug components (e.g., ETM)

- [External Private Peripheral Bus (EPPB)](./03-r-bls.md#external-secure-attribution-unit-esau) allows designers to add their own debug or vendor-specific components

- [System ROM Table](./03-r-bls.md#external-secure-attribution-unit-esau) is a simple lookup table that enables debug tools to extract the addresses of debug and trace components

## Secure Attribution Unit (SAU), Implementation Defined Attribution Unit (IDAU), and Memory Protection Unit (MPU)

Two units determine the security attribute of an address:

1. The internal programmable [Secure Attribution Unit (SAU)](https://developer.arm.com/documentation/100690/0201/Attribution-units--SAU-and-IDAU-).

2. The external Implementation Defined Attribution Unit (IDAU), through the IDAU interface, returns the security attribute and region number of an address.

![mpu](./resources/sld717-mpu.png)

Three possible configurations to define the security attribute of an address:

1. Internal SAU only

2. External IDAU only

3. A combination of the internal [SAU](./03-r-bls.md#security-attribution-unit) and external IDAU

**Notes**:

- Series 2 devices use configuration 3.

- IDAU in Series 2 devices is the [External Secure Attribution Unit (ESAU)](./03-r-bls.md#external-secure-attribution-unit-esau).

The [Memory Protection Unit (MPU)](https://developer.arm.com/documentation/100699/0100/) is a programmable unit that allows privileged software to define memory access permission. If the TrustZone is enabled, there can be up to two MPUs, one for Secure and one for Non-secure.

- The number of [MPU regions](./01-series-2-security-features.md) for the Secure and the Non-secure MPU can be different.

- The MPU registers are memory-mapped and are placed in the [System Control Space (SCS)](#system-private-peripheral-bus-ppb).

- Secure software can use the [MPU Functions for ARMv8-M](https://arm-software.github.io/CMSIS_5/Core/html/group__mpu8__functions.html) in CMSIS-Core to configure the Non-secure MPU through the [Non-secure alias SCS](#system-private-peripheral-bus-ppb) (`0xE002ED90` - `0xE002EDC4`).

<table class="classic"><colgroup><col width="25%"/><col width="25%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>Software</th><th>Non-secure MPU Registers</th><th>Secure MPU Registers</th><th>MemManage Fault</th></tr></thead><tbody><tr><td><p>Non-secure privileged</p></td><td><p>0xE000ED90 - 0xE000EDC4</p></td><td><p>-</p></td><td><p>Non-secure MPU violation</p></td></tr><tr><td><p>Secure privileged</p></td><td><p>0xE002ED90 - 0xE002EDC4</p></td><td><p>0xE000ED90 - 0xE000EDC4</p></td><td><p>Secure MPU violation</p></td></tr></tbody></table>

## Exceptions and Interrupts

### Type of Exceptions

The following table describes the types of exceptions in the TrustZone implemented system.

<table class="classic"><colgroup><col width="20%"/><col width="30%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>Section</th><th>Guidance</th><th>Type</th><th>Default State</th></tr></thead><tbody><tr><td><p>1 (-)</p></td><td><p>Reset</p></td><td><p>Secure only</p></td><td><p>Secure</p></td></tr><tr><td><p>2 (-14)</p></td><td><p>NMI</p></td><td><p>Configurable</p></td><td><p>Secure</p></td></tr><tr><td><p>3 (-13)</p></td><td><p>HardFault</p></td><td><p>Configurable</p></td><td><p>Secure</p></td></tr><tr><td><p>4 (-12)</p></td><td><p>MemManage Fault</p></td><td><p>Banked</p></td><td><p>Banked</p></td></tr><tr><td><p>5 (-11)</p></td><td><p>BusFault</p></td><td><p>Configurable</p></td><td><p>Secure</p></td></tr><tr><td><p>6 (-10)</p></td><td><p>UsageFault</p></td><td><p>Banked</p></td><td><p>Banked</p></td></tr><tr><td><p>7 (-9)</p></td><td><p>SecureFault</p></td><td>Secure only</p></td><td><p>Secure</p></td></tr><tr><td><p>11 (-5)</p></td><td><p>SVCall</p></td><td><p>Banked</p></td><td><p>Banked</p></td></tr><tr><td><p>12 (-4)</p></td><td><p>DebugMonitor</p></td><td><p>Configurable</p></td><td><p>Secure</p></td></tr><tr><td><p>14 (-2)</p></td><td><p>PendSV</p></td><td><p>Banked</p></td><td><p>Banked</p></td></tr><tr><td><p>15 (-1)</p></td><td><p>SysTick</p></td><td><p>Banked</p></td><td><p>Banked</p></td></tr><tr><td><p>16 - 495 (0 - 479)</p></td><td><p>IRQ0 - IRQ479</p></td><td><p>Configurable</p></td><td><p>Secure</p></td></tr></tbody></table>

**Notes**:

- "Secure only" means the system exceptions can only trigger in the Secure state.

- "Configurable" means the system exceptions and interrupts can be configured to target either the Secure state or the Non-secure state.

- Banked means the system exceptions can have Secure and Non-secure versions. Both can be triggered and executed independently and have different priority level settings.

### Exception Priorities

It may cause a security issue if the Non-secure software uses high priority levels to mask the Secure interrupts. To avoid this issue, TrustZone introduces a programmable bit in the `AIRCR` register called `PRIS` (Prioritize Secure exception) for Secure software to prioritize, if needed, Secure exceptions and interrupts.

The `AIRCR.PRIS` is set to 0 out of reset, which means Secure and Non-secure exceptions/interrupts share the same configurable programmable priority level space (columns 2 and 3 in the following table). When the `AIRCR.PRIS` is set to 1, all Non-secure configurable exceptions/interrupts are placed in the lower half of the priority level space so that Secure exceptions/interrupts can potentially have higher priorities (columns 2 and 4 in the following table).

<table class="classic"><colgroup><col width="25%"/><col width="25%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>Priority Value</th><th>Secure Priority</th><th>Non-secure Priority (PRIS = 0)</th><th>Non-secure Priority (PRIS = 1)</th></tr></thead><tbody><tr><td><p>0</p></td><td><p>0</p></td><td><p>0 (0x00)</p></td><td><p>128 (0x80)</p></td></tr><tr><td><p>1</p></td><td><p>32</p></td><td><p>32 (0x20)</p></td><td><p>144 (0x90)</p></td></tr><tr><td><p>2</p></td><td><p>64</p></td><td><p>64 (0x40)</p></td><td><p>160 (0xA0)</p></td></tr><tr><td><p>3</p></td><td><p>96</p></td><td><p>96 (0x60)</p></td><td><p>176 (0xB0)</p></td></tr><tr><td><p>4</p></td><td><p>128</p></td><td><p>128 (0x80)</p></td><td><p>192 (0xC0)</p></td></tr><tr><td><p>5</p></td><td><p>160</p></td><td><p>160 (0xA0)</p></td><td><p>208 (0xD0)</p></td></tr><tr><td><p>6</p></td><td><p>192</p></td><td><p>192 (0xC0)</p></td><td><p>224 (0xE0)</p></td></tr><tr><td><p>7</p></td><td><p>224</p></td><td><p>224 (0xE0)</p></td><td><p>240 (0xF0)</p></td></tr></tbody></table>

>**Note**: This table uses three bits (Bit [7:5]) of the group priority level (`AIRCR.PRIGROUP`) to limit the maximum number of preemption levels to 8. A lower priority value indicates a higher priority.

### Vector Tables

The following figure shows two vector tables for Secure and Non-secure exceptions and interrupts. The vector table offset is defined by a Vector Table Offset Register (`VTOR` at `0xE000ED08`), which can only be programmed in the privileged state.

![vector-table](./resources/sld717-vector-table.png)

Image: *[Vector Table](https://developer.arm.com/documentation/100235/0100/The-Cortex-M33-Processor/Exception-model/Vector-table).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

**Notes**:

- The `VTOR_S` defines the address of the Secure vector table in Secure memory, and the [Secure Main Stack Pointer](#general-purpose-registers) (`MSP_S`) is the default stack for the Secure exception handler.

- The `VTOR_NS` defines the address of the Non-secure vector table in Non-secure memory, and the [Non-secure Main Stack Pointer](#general-purpose-registers) (`MSP_NS`) is the default stack for the Non-secure exception handler.

- Secure privileged software can access the `VTOR_NS` using the [Non-secure SCB alias](#system-private-peripheral-bus-ppb) (`0xE002ED08`).

- The [System Control Space](#system-private-peripheral-bus-ppb) contains registers for the SysTick timer, NVIC, and SCB.

- The interrupt masking registers (PRIMASK, FAULTMASK, and BASEPRI) are [banked](#special-purpose-registers) between security states. The priority level space is shared between the Secure and the Non-secure world, setting an interrupt mask register on one side can block some, or all, of the exceptions on the other side.

- Interrupts (`IRQ0` - `IRQ479`) are defined as Secure by default. Each interrupt can be configured as Secure or Non-secure and is determined by the Interrupt Target Non-secure (`NVIC_ITNS`) register, which is only programmable in the Secure software.

### State Transitions in Exceptions and Interrupts

The following figure shows transitions between the [processor states](#introduction) in ARMv8-M TrustZone.![State Transitions](./resources/sld717-state-transitions.png)

Image (left): *[Switching-between-Secure-and-Non-secure-states](https://developer.arm.com/documentation/100690/0201/Switching-between-Secure-and-Non-secure-states).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

1. Secure Thread → Secure Handler or Non-secure Thread to Non-secure Handler

   - No security state transition.

   - The exception sequence is almost identical to the exception stacking mechanism of current Cortex-M processors.

   - The Interrupt Service Routine (ISR) is executed in the current security state (either Secure or Non-secure).

2. Non-secure Thread → Secure Handler or Non-secure Handler → Secure Handler

   - The transition from Non-secure to Secure state.

   - The exception sequence is almost identical to the exception stacking mechanism of current Cortex-M processors.

   - The ISR is executed in a Secure state.

3. Secure Thread → Non-secure Handler or Secure Handler → Non-secure Handler

   - The transition from Secure to Non-secure state.

   - To avoid an information leak when transitioning from the Secure to Non-secure state. The processor automatically pushes all general-purpose registers into the Secure stack and erases the contents of all general-purpose registers before executing the Non-secure ISR. The processor pops the contents of all general-purpose registers from the Secure stack when returning from the Non-secure ISR (right side in [Figure 2.6 State Transitions on page 12](#state-transitions-in-exceptions-and-interrupts)). It incurs a slightly longer interrupt latency.

   - The ISR is executed in a Non-secure state.

4. Secure Privileged Thread ↔ Non-secure Privileged Thread or Secure Unprivileged Thread ↔ Non-secure Unprivileged Thread

   - The transition from Secure to Non-secure state or Non-secure to Secure state.

   - The [Function calls and returns](#switching-between-secure-and-non-secure-states) can be used when the privileged level remains the same.

>**Note**: Subject to interrupt priority, there are no restrictions regarding whether a Non-secure or Secure interrupt can occur when the processor runs Non-secure or Secure code.

## Switching Between Secure and Non-secure States

The TrustZone allows direct calling between Secure and Non-secure software. The following figure shows how to use an API function call to trigger security state transitions. The state transitions can also happen because of [exceptions and interrupts](#state-transitions-in-exceptions-and-interrupts).

![swich state](./resources/sld717-switch-state.png)

Image: *[Switching-between-Secure-and-Non-secure-states](https://developer.arm.com/documentation/100690/0201/Switching-between-Secure-and-Non-secure-states).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

### Switching from Non-secure to Secure State

When the Non-secure program calls a Secure software, the first instruction must be a Secure Gateway (`SG`) instruction residing in Non-secure Callable memory. The Secure Gateway entry points (veneers) decouple the address of the `SG` instructions in the Non-secure Callable memory region from the rest of the Secure code. It can eliminate the risk of having inadvertent entry points when the Secure software contains a pattern that matches the opcode of the `SG` instruction.

![switch-image](./resources/sld717-switch-s.png)

Image (right): *Whitepaper - ARMv8-M Architecture Technical Overview. Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

The bit 0 of the [Link Register (LR)](#special-purpose-registers) is cleared to zero by `SG` instruction to indicate that returning from this function transits from Secure to Non-secure. The processor is still in the Non-secure state when the `SG` instruction is executed. The `BXNS LR` instruction is used when returning since a normal `BX LR` instruction interprets it as an unsupported execution mode change. A [SecureFault](#vector-tables) exception is triggered if the processor returns to a Secure address. It prevents a hacker from calling a Secure API with a fake return address pointing to a Secure program location. If bit 0 of LR is 1, the `BXNS LR` instruction behaves like a normal `BX LR`. Therefore, Secure code can call a Secure API in the NSC region even it is not a usual practice.

<table class="classic"><colgroup><col width="25%"/><col width="25%"/><col width="25%"/><col width="25%"/></colgroup><thead><tr><th>Program</th><th>Call Instruction</th><th>SG Instruction</th><th>Return Instruction</th></tr></thead><tbody><tr><td><p>Non-secure call Non-secure</p></td><td><p>BL or BLX</p></td><td><p>-</p></td><td><p>BX LR (Return to Non-secure state)</p></td></tr><tr><td><p>Non-secure call Secure</p></td><td><p>BL or BLX</p></td><td><p>Clear bit 0 of LR</p></td><td><p>BXNS LR (Return to Non-secure state)</p></td></tr><tr><td><p>Secure call Secure</p></td><td><p>BL or BLX</p></td><td><p>Set bit 0 of LR</p></td><td><p>BXNS LR (Return to Secure state)</p></td></tr></tbody></table>

To help software developers create Secure APIs in C/C++, the [Cortex-M Security Extension (CMSE)](https://developer.arm.com/documentation/ecm0359818/latest) defines a C function attribute called `cmse_nonsecure_entry`.

- GCC — `__attribute__((cmse_nonsecure_entry))`

- IAR — `__cmse_nonsecure_entry`

### Test Target (TT) Instruction

The software can use an ARMv8-M instruction called Test Target (TT) and the region number generated by the SAU or the IDAU to determine if a contiguous range of memory shares common security attributes and privilege levels.

The TT instruction returns the [SAU/IDAU](#secure-attribution-unit-sau-implementation-defined-attribution-unit-idau-and-memory-protection-unit-mpu) region number, security attributes (S/NS), and MPU region number after passing the start and end addresses of the memory range to the TT instruction. The software can determine whether the memory range has required security attributes and resides in the same region number.

![tt-instruction](./resources/sld717-tt-instruction.png)

Image: *[Test-target-instruction](https://developer.arm.com/documentation/100690/0201/Test-target-instruction).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

This mechanism allows security checking at the beginning of the API service (instead of during the operation) to determine if the memory referenced by a pointer from Non-secure software points to the Non-secure address. It prevents Non-secure software from using those APIs in Secure software to access or modify Secure data.

To make these operations easier in a C/C++ programming environment, the [Cortex-M Security Extension (CMSE)](https://developer.arm.com/documentation/ecm0359818/latest) has defined a range of [intrinsic functions](https://www.keil.com/support/man/docs/armclang_ref/armclang_ref_pge1446715440722.htm) for dealing with pointer checks with the TT instructions.

### Switching from Secure to Non-secure State

When the Secure program calls a Non-secure software, the Secure program must use a `BLXNS <reg>` instruction to invoke the process. If bit 0 of the `<reg>` is 0, the processor must switch to the Non-secure state when branching to the target address. During the state transition, the return address and some processor state information are pushed onto the Secure stack, while the return address on the [Link Register (LR)](#special-purpose-registers) is set to a special value called `FNC_RETURN` (`0xFEFFFFFF`).

The Non-secure function completes by performing a branch (`BX LR`) to the `FNC_RETURN` address (bit 0 is 1 to indicate the function was called from the Secure state). It automatically triggers the unstacking of the actual return address from the Secure stack and returns to the calling function. The `FNC_RETURN` hides the return address of the Secure program from the Non-secure software to avoid the leakage of any secret information. It also prevents Non-secure software from modifying the Secure return address stored in the Secure stack.

![switch_ns](./resources/sld717-switch-ns.png)

Image: *[Switching-between-Secure-and-Non-secure-states](https://developer.arm.com/documentation/100690/0201/Switching-between-Secure-and-Non-secure-states).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

To help software developers declare Non-secure function pointers in C/C++, the [Cortex-M Security Extension (CMSE)](https://developer.arm.com/documentation/ecm0359818/latest) defines a C function attribute called `cmse_nonsecure_call`.

- GCC: `__attribute__((cmse_nonsecure_call))`

- IAR: `__cmse_nonsecure_call`

## Software Flow

The following figure describes a software flow example in a TrustZone implemented system.

![Execution Flow of a TrustZone Implemented System](./resources/sld717-execution-flow-of-a-trustzone-implemented-system.png)

Image: *[Software Development in ARMv8-M Architecture](https://community.arm.com/cfs-file/__key/telligent-evolution-components-attachments/01-2142-00-00-00-01-27-19/ARM-Cortex-_2D00_-session-11-_2D00_-Yiu-_2D00_-Software-Development-in-ARMv8_2D00_M-Architecture.pdf).Copyright © 1995-2022 Arm Limited (or its affiliates). All rights reserved.*

1. The system starts executing code in the Secure state after a power-on or reset (Secure boot).

   - The Secure [stack pointer](#general-purpose-registers) (`MSP_S`) is set from the address of the Secure vector table (`VTOR_S`).

   - The Secure Reset Handler pointed by the `VTOR_S` is called.

   - Perform various initialization tasks such as C startup code.

   - Place peripherals and associated interrupts in either Secure or Non-secure applications.

   - Program [SAU/IDAU](./04-r-programming.md#) to partition the entire memory into Secure, Non-secure Callable, and Non-secure regions.

   - Program the address of the Non-secure vector table (`VTOR_NS`).

   - Initialize the two first entries of the table for the Non-secure stack pointer (`MSP_NS`) and Reset Handler to emulate a Non-secure reset.

2. The Secure firmware branches to the entry point (Reset Handler pointed by the `VTOR_NS`) of the Non-secure application.

   - The Non-secure software has its Reset Handler.

   - Perform various initialization tasks such as C startup code and hardware initialization (e.g., Non-secure peripherals).

   - It does not conflict with initialization from the Secure code as the stack and heap spaces of Secure and Non-secure code are separated.

3. During the execution of Non-secure applications, the application could call Secure APIs through the [Secure Gateway (SG) veneer](#switching-from-non-secure-to-secure-state) in the Non-secure Callable region.

4. In some cases, Secure APIs might need to call [Non-secure call-back functions](#switching-from-non-secure-to-secure-state) (e.g., a hardware driver).
