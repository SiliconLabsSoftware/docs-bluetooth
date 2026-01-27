# Appendix B: Platform Examples

## Overview

Simplicity Studio 5 includes the [SE Manager platform examples](https://docs.silabs.com/simplicity-studio-5-users-guide/latest/ss-5-users-guide-getting-started/start-a-project#examples) for Secure Tamper. Refer to the corresponding readme file for details about the SE Manager example. This file also includes the procedures to create the project and run the example.

<table>
	<thead>
		<tr>
			<th>Category</th>
			<th>SE Manager Platform Example</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Tamper provisioning and runtime operations</td>
			<td>SoC SE Manager Tamper</td>
			<td>
				Sample application to:
				<ul>
					<li>provision tamper settings</li>
					<li>temporarily disable tamper and roll challenge</li>
					<li>Implement an interrupt handler for a filtered tamper source</li>
				</ul>
			</td>
		</tr>
	</tbody>
</table>

## Adapting PRS Tamper for Custom Hardware

The platform examples are designed to work with Silicon Labs development kits. To adapt the code to work with custom hardware a few steps are required as follows. In the sample application these steps are found in `init_tamper_prs` of the `app_se_manager_tamper.c` source file:

1. Set up the GPIO pin as an input. The sample application demonstrates calls to set up the GPIO. It is recommended to install the ‘simple button’ software component to ensure that the necessary drivers are installed.

2. Set up an unused PRS channel as a producer. The sample application uses PRS channels 6, 7 and 8. Refer to the reference manual for your device to determine the source and signal for GPIO. The sample application demonstrates API calls for Series 2 and Series 3 devices.

3. Connect the PRS channel chosen in the previous step to the desired tamper signal. Tamper signals are named `CONSUMER_SETAMPER_TAMPERSR\<bit\>`, where bit is the bit position of the PRS tamper signal. Refer to [Tamper Sources](./05-tamper-sources.md) for the bit positions of PRS tamper signals in the EFR32xG21B and [Appendix C: Tamper Sources Reference](./12-appendix-c-tamper-sources-reference.md) for the bit positions of PRS tamper signals in other devices. The sample application demonstrates calls to perform this task.
