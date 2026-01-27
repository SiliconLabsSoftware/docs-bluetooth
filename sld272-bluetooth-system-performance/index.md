# System Performance

These pages address a variety of different topics related to improving system performance and reducing power consumption. 

- [**Using the LFRCO as a Low-Frequency Clock Source**](./lfrco-clock-source): Describes using the LFRCO as the low-frequency clock source (instead of the LFXO) to wake up the device on each connection interval when sleep is enabled in the stack.
- [**Bluetooth TX Power Settings**](./tx-power): Discusses how to configure TX power and how the actual TX power will change.
- [**Current Consumption Variation with TX Power**](./current-consumption-tx-power): Discusses the current consumption for different TX power settings, measured on an EFR32BG13 during advertisement. The test result can be used as a generic guideline to set the TX power level in an energy-constrained system.
- [**Bluetooth Radio Task Priorities**](./radio-task-priorities): Reviews using the radio scheduler to simulate running multiple tasks that use the radio at the same time. 
- [**Setting a Custom BT Address - Production Approach**](./custom-address-production-approach.md): Explains two methods to modify the non-volatile region to set a custom BT address in production.
- [**Bluetooth LE Auto-PA Mode**](./auto-pa-mode): Describes how to implement automatic switching between the PAs, when high accuracy output is needed both above and below 0 dBm.
