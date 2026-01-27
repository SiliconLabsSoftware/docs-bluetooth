# Precautions

## Device Erase for Secure Debug

Disabling the Device Erase is mandatory for secure debug as described in the following table.

<table>
	<thead>
		<tr>
			<th>Device Erase</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Enabled</td>
			<td>By using <code>erasedevice</code> command, entire Flash contents can be erased, except the OTP bits. This could potentially expose the device to security risks, as it allows malicious actors to flash different or vulnerable image to the device.</td>
		</tr>
		<tr>
			<td>Disabled</td>
			<td>
				<ul>
					<li>Command <code>erasedevice</code> will not work as JTAG (debug port) is closed</li>
					<li>Since JTAG is disabled, only way to upgrade the firmware is via boot loader.</li>
					<li>If <strong>secure Debug</strong> and <strong>Debug Lock</strong> are enabled, devices allow only authorized parties to debug the device.</li>
					<li>If secure boot is also enabled boot loader will accept only authenticated (signed) firmware image, thereby ensures authenticity and integrity of the firmware.</li>
				</ul>
			</td>
		</tr>
	</tbody>
</table>

**Notes**:

1. Advised not to disable **Device Erase** during development phase as it is one time operation.

2. Without enabling **Secure Debug** disabling **Device Erase** will make the device permanently not usable.

Run the `security lockconfig --secure-debug-unlock disable` command to disable **Device Erase**.

```sh
commander security lockconfig --secure-debug-unlock disable --device sixg301 --serialno 440328778
```

```sh
================================================================================
WARNING: Device erase is disabled and secure debug access is locked.
If disabling secured debug access, there is no way to regain debug access to this device if continuing with this command.
Type 'continue' and hit enter to proceed or Ctrl-C to abort:
================================================================================
continue
Secure debug unlock was disabled
DONE
```

## Secure Boot and Debug Lock

The following table describes the different debug lock scenarios on the secure boot-enabled device.

| **Secure Debug** | **Device Erase** | **Debug Lock** | **State** | **Recover from Secure Boot Failure** |
|------------------|------------------|----------------|-----------|--------------------------------------|
| Disabled | Enabled | Disabled | Standard debug unlock | Flash a correctly signed image. |
| Disabled | Enabled | Enabled | Standard debug lock | Flash a correctly signed image after standard debug unlocking the device. |
| Disabled | Disabled | Enabled | Permanent debug lock | There is no way to recover the device. Make sure the programmed image is correctly signed before locking the device. |
| Enabled | Disabled | Enabled | Secure debug lock | Flash a correctly signed image after secure debug unlocking the device. |

**Note**: See *Recover Devices when Secure Boot Fails* in [Series 2 and Series 3 Secure Boot with RTSL](https://docs.silabs.com/mcu-bootloader/latest/series2-secure-boot-with-rtsl/) to flash a correctly signed image on different debug lock scenarios.

## Limitation on Roll Challenge

On Series 2 devices, challenge can be rolled any number of times, whereas on Series 3, challenge can be rolled a maximum of 128 times.
