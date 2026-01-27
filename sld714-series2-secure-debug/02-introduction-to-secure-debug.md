# Introduction to Secure Debug

## Debug Lock

All Silicon Labs Series 2 and Series 3 devices have the capability to lock debug access to the device. This prevents attackers from using the debug interface to perform the following illegal operations:

- Reprogramming the device

- Interrogating the device

- Interfering with the operation of the device

A fairly standard practice during the board-level test in production is to program, test, and lock the parts. Three different locks can be enabled on debug interface:

- Standard debug lock

- Secure debug lock

- Permanent debug lock

Silicon Labs provides Custom Part Manufacturing Service (CPMS) to securely configure the debug port of the chip to one of the three possible locks before the devices leave the factory.

## Debug Unlock

Users need to unlock parts under a number of circumstances:

- Code development

- Field failure diagnosis

- Product field service

- Existing inventory reprogramming

Two different unlocks can run on debug interface:

- Standard debug unlock

- Secure debug unlock
