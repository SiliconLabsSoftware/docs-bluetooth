# Series 2 and Series 3 Secure Debug

> **Note: This section replaces *AN1190: Series 2 and Series 3 Secure Debug*. Further updates to this application note will be provided here**.

To protect intellectual property and proprietary algorithms from malicious actors, it is important to lock the debug port. This prevents attackers from reading flash contents through the debug  interface  and  reverse  engineering  the application. However, there are situations where it becomes necessary to debug a device already deployed in the field. Silicon Labs Series 2 and 3 devices offer a feature called Secure Debug. This feature allows only authenticated users to unlock the debug port and securely debug the application.

This application note explains the different debug lock and unlock features available in Series 2 and Series 3 devices and their capabilities.

## Key Points

- Basic overview of the Secure Engine
- Debug port access by Debug Challenge Interface (DCI) or Mailbox Interface
- Locking and unlocking features for Series 2 and Series 3 devices
- Examples for Public Command Key provisioning and Secure Debug Unlock

> **Note**: This application note covers both Series 2 and Series 3 devices. Unless explicitly stated otherwise, all items are applicable to both. If an item is not applicable to one of the series, it will be mentioned separately.
