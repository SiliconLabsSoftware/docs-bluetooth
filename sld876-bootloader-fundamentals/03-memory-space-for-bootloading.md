# Memory Space for Bootloading

The first stage of the Gecko Bootloader on Series 1 devices takes up a single flash page. On devices with 2 kB flash pages, like EFR32MG1, this means that the first stage takes 2 kB.

The size of the main bootloader is dependent on the functionality required. With a typical bootloader configuration, the main bootloader for Series 1 devices takes up 14 kB of flash, bringing the total bootloader size to 16 kB.

Silicon Labs recommends reserving 16 kB for the bootloader for Series 1 and EFR32xG21 devices and 24 kB for EFR32xG22 devices.

On EFR32xG1 devices (Mighty Gecko, Flex Gecko, and Blue Gecko families), the bootloader resides in main flash.

- First stage bootloader @ 0x0

- Main bootloader @ 0x800

- Application @ 0x4000

On EFR32xG12 and later Series 1 devices, the bootloader resides in the bootloader area in the Information Block.

- Application @ 0x0

- First stage bootloader @ 0x0FE10000

- Main bootloader @ 0x0FE10800

On EFR32xG21, the main bootloader resides in main flash:

- Main bootloader @ 0x0

- Application @ 0x4000

On EFR32xG22, the main bootloader resides in main flash:

- Main bootloader @ 0x0

- Application @ 0x6000

On EFR32xG23, the main bootloader resides in main flash:

- Main bootloader @ 0x08000000

- Application @ 0x08006000

On EFR32xG24, the main bootloader resides in main flash:

- Main bootloader @ 0x08000000

- Application @ 0x08006000
