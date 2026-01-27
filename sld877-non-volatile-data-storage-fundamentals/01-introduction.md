# Introduction

Non-volatile memory (NVM) is memory that persists even when the device is power-cycled. On Silicon Labs microcontrollers and radio SoCs, the NVM is implemented as flash memory. In many applications the flash is not only used to store the application code but also to store data objects that are written and read frequently by the application. As flash memory can only be erased a limited number of times, several methods exist to efficiently read and write non-volatile data without wearing out the flash.

Some data is considered manufacturing data that is written only once at manufacturing time. This document is concerned with dynamic data that changes frequently over the life of the product.

This document provides an introduction to the main design options for dynamic data storage in microcontrollers and radio SoCs, along with guidelines on what factors affect flash lifetime. In addition it introduces the main flash data storage implementations offered by Silicon Labs:

- NVM3

- Simulated EEPROM version 1 (SimEEv1) and version 2 (SimEEv2)

- Persistent Store (PS Store)
