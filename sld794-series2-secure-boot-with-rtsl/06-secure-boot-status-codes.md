# Secure Boot Status Codes

Boot status codes can be used to know the status of the boot mechanism. The `security status` command can be used to get the boot status.

```sh
commander security status --device EFR32MG21A010F1024 --serialno 440048205
```

```sh
SE Firmware version : 1.2.9
Serial number : 000000000000000014b457fffe045afd
Debug lock : Disabled
Device erase : Enabled
Secure debug unlock : Disabled
Tamper status : OK
Secure boot : Enabled
Boot status : 0x20 - OK
DONE
```

The following table shows Status codes in Secure Boot mechanism and their description:

| **STATUS CODE** | **DESCRIPTION** |
|-----------------|-----------------|
| 0x00 | Start PUF |
| 0x01 | Fetch OTP for bootloader |
| 0x02 | Tamper test |
| 0x03 | Self-tests |
| 0x04 | TRNG Initialization failed |
| 0x05 | NVM Initialization failed |
| 0x0B | Jump to main loop |
| 0x0C | Fetch OTP for host boot |
| 0x0D | Fetch pointers host firmware |
| 0x0E | Fetch header host firmware |
| 0x0F | Fetch host firmware |
| 0x10 | Check version host firmware |
| 0x11 | Check signature on certificate for host firmware |
| 0x12 | Check signature host firmware |
| 0x13 | Failed to get data from internal NVM |
| 0x14 | Finding host application properties pointer |
| 0x15 | Validating host application properties structure |
| 0x16 | Validating host application signature pointer |
| 0x17 | Getting SecureBoot key |
| 0x18 | SecureBoot requires cert, but none found |
| 0x19 | Updating required certificate version failed |
| 0x1A | Certificate is of an older version as the last cert we validated |
| 0x1B | Certificate structure version is not supported by this firmware |
| 0x1C | Certificate pointer is out of range |
| 0x1D | Region 0 is not closed |
| 0x20 | Main loop entered |
| 0x80 | PUF AC was somehow cleared |
| 0x81 | PUF failed to reconstruct after the longest delay |
| 0x90 | ESEC aborted booting due to catching too many successive tamper resets |
| 0xFF | Finished verifying host app |
