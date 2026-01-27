# Introduction

Gecko Software Development Kit suite (GSDK) version 3 (GSDK v3.0) introduced a new underlying Gecko Platform architecture based on components. Beginning with GSDK 4.0, the Gecko Bootloader now uses this underlying architecture. With Simplicity Studio 5 (SSv5) and GSDK 4.0, developers working with Gecko Bootloader will benefit from the following component-based project configuration features:

- Search and filter to find and discover software components that work with the target device

- Automatically pull in all component dependencies and initialization code

- Configurable software components including peripheral inits, drivers, middleware, and stacks

- All configuration settings are in C header files for usage outside of Simplicity Studio

- Configuration validation to alert developers to errors or issues

- Easily manage all project source via git or other SCM tools

- Managed migration to future component and SDK versions

- Simplified transitions from Silicon Labs development kits to custom hardware

Other features of the SSv5/GSDK 4.x development environment include:

- Project source management options (link to SDK sources or copy all contents to user folder)

- Graphical pin configuration through the Pin Tool

- Redesigned Radio Configurator with a fresh UI that’s more intuitive for single- and multi-PHY customization

- Iterative development (configure components, edit sources, compile, debug) using SSv5 configuration tools and third-party IDEs

- GNU makefiles as a build option

This document summarizes the differences between the Gecko Bootloader v2.x in GSDK 4.0 and earlier AppBuilder-based versions. These differences include:

- Differences between AppBuilder and the new component-based Project Configurator

- Comparison of the new Project Configurator components with AppBuilder plugins

- Features of the Pin Tool, now used instead of Hardware Configurator

- Other differences including:

  - Linker file

  - Additional Macros

  - Postbuild steps

  - Main bootloader in main flash

  - Callbacks

  - Storage Slots

  - App Properties
