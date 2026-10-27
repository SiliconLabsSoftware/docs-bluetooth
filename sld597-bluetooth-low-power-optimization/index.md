# Introduction

**NOTE: This section replaces *AN1366: Bluetooth® LE Use Case-Based Low Power Optimization* for Simplicity SDK 2024.12.2 and up. Further updates to this application note will be provided here.**

These pages describe how to exploit the different features of Bluetooth technology to achieve the minimum possible energy consumption for a given use case. What follows lists the most common use cases a Bluetooth developer may encounter and suggests a solution for each of them. This provides an ideal starting point if you are already somewhat familiar with Bluetooth but still not sure about where to start your development.

Bluetooth Low Energy has the potential to add wireless connectivity to your device while keeping its energy consumption low. To achieve low energy consumption, however, you must be aware of the possibilities, and choose wisely. Different use cases may require different approaches and different Bluetooth features. In many cases, for example, you do not even need to create a Bluetooth connection to communicate, and even if you choose to create a connection, you should tune a number of parameters for optimum power reduction.

This guide lists the most common use cases a Bluetooth developer may encounter and suggests a solution for each of them. If you find one on this list that suits your needs, we strongly recommend you start your development here. The following use cases are discussed:

- I want to signal the presence of a device.

   This use case discusses Bluetooth beacons, where a single device broadcasts a short message (usually an ID) to be picked up by other Bluetooth-capable devices. Often used for finding a given asset that transmits the message or for geo-fencing (where the presence of the message signals that you entered an area).

- I want to broadcast data (such as information about a product/artwork).

   In this use case not only an ID is broadcasted by the device, but also some additional information, for example the price of a showcased product, a description of a painting, a URL that helps you find more information about a company that advertises itself, and so on. The data may also be dynamic, changing slowly or rapidly.

- I want to broadcast / send data to low-power receivers.

   You may have hundreds of devices that need to be updated with some information from time to time (for example. updating their real time). Data broadcast can be solved with Bluetooth beacons (advertisements). But what if the receivers are not smartphones and they also need to be low-power? For example, you have hundreds of asset tags that are being tracked with Bluetooth Direction Finding, and you want to update each of them with some information.

- I want to turn lamps on and off.

   This is a very common use case, where you want to issue short commands to nearby devices over Bluetooth. Your first thought might be to use connections, but sometimes it is much faster and less energy-demanding to send commands in advertisements. Low-power switches/controllers are discussed as well as low-power controlled devices.

- I want to connect to a device occasionally to exchange some data.

   Sometimes you need a connection even for short messages, either because of security, privacy, reliability, or simply because you need two-way communication. In this use case we discuss short-term connections, when a connection is only made to exchange some data quickly.

- I want to maintain a long-term connection with occasional data exchange.

   Keeping a connection alive may actually require less power than disconnecting, advertising, and then creating a new connection. This approach may be useful between fixed installation devices (such as a thermostat and an air-conditioner) that exchange some data from time to time, or between a smart phone and a smart watch. Manually-controlled connections (such as configuring a device with a smartphone app) are also considered as long-term connections with occasional data exchange.

- I want to send a lot of data on a long-term connection.

   You may need a continuous wireless communication, for example with a wireless keyboard or other HID (human interface device). File transfer and audio transfer also fall under this case, although they are not typical Bluetooth Low Energy use cases. In this case you can still tune and dynamically change the connection parameters and the TX power to achieve the lowest possible energy consumption.

- I want to exchange data with many peripheral devices.

   A very common use case is that your central device needs to communicate with many peripheral devices. The ideal solution in this case depends on the number of peripheral devices, which may be two or two thousand.

If you have a different use case, and you need some tips for low energy consumption, you can always find help at [https://community.silabs.com/](https://community.silabs.com/).

The figures in Use cases 2, 3, and 5 are from the Bluetooth Core Specification at [https://www.bluetooth.com/specifications/specs/core-specification/](https://www.bluetooth.com/specifications/specs/core-specification/).
