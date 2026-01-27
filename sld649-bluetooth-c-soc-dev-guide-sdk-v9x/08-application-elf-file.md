# Application ELF-file

ELF (Executable and Linkable Format) is a standard file format for executable files. This section describes the sections in the ELF file related to the application and the Bluetooth stack.

Some linkers provide output describing the consumed flash, but what it contains is not obvious. A Bluetooth project might contain a bootloader and the Bluetooth AppLoader, and the device might have separate flash for the bootloader. The ELF-file provides exact information about RAM and flash usage.

Simplicity Studio provides the GCC toolchain, which contain command line tool *objdump*. This tool can be used to get section information from the ELF-file.

*objdump* requires input ELF-file. If the parameter `-h` is used, *objdump* dumps the section header information.

**IAR**

Calling *objdump* from the command line for an example application:

```C
arm-none-eabi-objdump -h ewarm-iar/exe/bt_soc_empty.out
```

*objdump* then gives the following output:

```C
Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 application   0002f468  08012000  08012000  00000034  2**9
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 storage_regions 0000a000  08174000  08174000  0002f49c  2**13
                  ALLOC
  2 application_ram 0000279c  20000000  20000000  0002f49c  2**3
                  ALLOC
  3 application_heap 0003d860  200027a0  200027a0  0002f49c  2**3
                  ALLOC
  4 .debug_abbrev 0000c8ba  00000000  00000000  0002f49c  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
  5 .debug_aranges 00008de0  00000000  00000000  0003bd58  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
  6 .debug_frame  000186fa  00000000  00000000  00044b38  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
  7 .debug_info   00131969  00000000  00000000  0005d234  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
  8 .debug_line   0008a871  00000000  00000000  0018eba0  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
  9 .debug_loc    00028c5c  00000000  00000000  00219414  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 10 .debug_macinfo 0000d5be  00000000  00000000  00242070  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 11 .debug_pubnames 0000de9f  00000000  00000000  0024f630  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 12 .debug_ranges 00005b28  00000000  00000000  0025d4d0  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 13 .debug_types  0000461e  00000000  00000000  00262ff8  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 14 .iar.debug_frame 0001e0b7  00000000  00000000  00267618  2**0
                  CONTENTS, READONLY
 15 .iar.debug_line 000413fd  00000000  00000000  002856d0  2**0
                  CONTENTS, READONLY
 16 .comment      0015748a  00000000  00000000  002c6ad0  2**0
                  CONTENTS, READONLY
 17 .iar.rtmodel  00000042  00000000  00000000  0041df5c  2**0
                  CONTENTS, READONLY
 18 .ARM.attributes 00000032  00000000  00000000  0041dfa0  2**0
                  CONTENTS, READONLY
```

`application` contains the application code and read-only data. Size of the application in this example is 0x2f468 in hexadecimal, and 193640 bytes in decimal.

`application_ram` is a RAM section for the call stack.

`application_heap` is the RAM section for heap.

Refer to IAR documentation for description of the remaining sections.

**GCC**

Calling *objdump* from the command line for an example application:

```C
arm-none-eabi-objdump -h build/release/bt_soc_empty.out
```

*objdump* then gives the following output:

```C
Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .vectors      00000170  08012000  08012000  00002000  2**9
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .stack        00000ac0  20000000  20000000  00060000  2**3
                  ALLOC
  2 .bss          00001844  20000ac0  20000ac0  00060000  2**3
                  ALLOC
  3 text_application_ram 000001ac  20002304  08012170  00002304  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  4 .text         000302a8  08012320  08012320  00012320  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  5 .ARM.exidx    00000008  080425c8  080425c8  000425c8  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .copy.table   0000000c  080425d0  080425d0  000425d0  2**0
                  CONTENTS, ALLOC, LOAD, DATA
  7 .zero.table   00000000  080425dc  080425dc  0005269c  2**0
                  CONTENTS
  8 .data         000001ec  200024b0  080425dc  000524b0  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  9 .memory_manager_heap 00000004  2000269c  080427c8  0005269c  2**0
                  ALLOC
 10 .nvm          0000a000  080427c8  080427c8  0005269c  2**0
                  CONTENTS
 11 .ARM.attributes 00000036  00000000  00000000  0005c69c  2**0
                  CONTENTS, READONLY
 12 .comment      00000045  00000000  00000000  0005c6d2  2**0
                  CONTENTS, READONLY
 13 .debug_line_str 000001c0  00000000  00000000  0005c717  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 14 .debug_info   00101e0a  00000000  00000000  0005c8d7  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 15 .debug_abbrev 0001afa4  00000000  00000000  0015e6e1  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 16 .debug_loclists 00036269  00000000  00000000  00179685  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 17 .debug_aranges 00009550  00000000  00000000  001af8ee  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 18 .debug_rnglists 00009d21  00000000  00000000  001b8e3e  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 19 .debug_line   00055c06  00000000  00000000  001c2b5f  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 20 .debug_str    00034194  00000000  00000000  00218765  2**0
                  CONTENTS, READONLY, DEBUGGING, OCTETS
 21 .debug_frame  00015f48  00000000  00000000  0024c8fc  2**2
                  CONTENTS, READONLY, DEBUGGING, OCTETS
```

`.text` contains the application code and read-only data. The size of the application in this example is 0x302a8 in hexadecimal and 197288 bytes in decimal.

`.ARM.exidx` is used for debugging.

`.stack` is a RAM section for the call stack

`.data` is the RAM section for initialized variables.

`.bss` is the RAM section for uninitialized variables.

`.memory_manager_heap` is the RAM section for heap.

Refer to GCC documentation for a description of the remaining sections.
