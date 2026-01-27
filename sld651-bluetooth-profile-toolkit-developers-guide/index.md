# Bluetooth Profile Toolkit Developer's Guide

**NOTE: This section replaces *UG118: Bluetooth Profile Toolkit Developer's Guide*. Further updates to this user guide will be provided here.**

Bluetooth GATT services and characteristics are the basis of the Bluetooth data exchange. They are used to describe the structure, access type, and security properties of the data exposed by a device, such as a heart-rate monitor. Bluetooth services and characteristics have a well-defined and structured format, and they can be easily described using XML mark-up language.

The Profile Toolkit is an XML-based mark-up language for describing the Bluetooth services and characteristics, also known as the GATT database, in both easy human-readable and machine-readable formats. This guide walks you through the XML syntax used in the Profile Toolkit and instructs you how to easily describe your own Bluetooth services and characteristics, configure the access and security properties, and how to include the GATT database as a part of the firmware. This guide also contains practical examples showing the use of both standardized Bluetooth and vendor-specific proprietary services. These examples provide a good starting point for your own development work.

## Key Points

- Understanding Bluetooth GATT profiles, services, characteristics, attribute protocol
- Building the GATT database with the Profile Toolkit
