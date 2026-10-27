# Introduction

Network Analyzer is a tool for analyzing wireless network traffic. It supports a wide variety of short-range wireless protocols like Bluetooth Low Energy, Zigbee, proprietary protocols and others. It is provided as part of the Simplicity Studio tool set.

## Debugging a Wireless Network

Silicon Labs’ tool set provides the user with a comprehensive way to analyze wireless traffic. With it, the user can tap into the data buffers of the radio transceiver via a dedicated serial hardware interface called the Packet Trace Interface (PTI). PTI data can be then transferred via USB or Ethernet to a computer running Simplicity Studio. Finally, the time-stamped data can be interpreted and displayed in Network Analyzer.

Most Silicon Labs’ development kits, such as the Wireless Starter Kit (WSTK), have the PTI embedded and ready to use. Note that it is also possible to use the network analysis features when working on custom hardware if the PTI pins are exposed via a debug interface.
