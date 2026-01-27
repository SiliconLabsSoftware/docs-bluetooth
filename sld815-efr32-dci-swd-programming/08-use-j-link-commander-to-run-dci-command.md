# Use J-Link Commander to Run DCI Command

J-Link Commander from [Segger](https://www.segger.com/downloads/jlink/) is a command line-based utility that supports simple commands to verify and communicate with the target connection. There are two ways to run the DCI command through the J-Link Commander.

1. [J-Link Command File](https://wiki.segger.com/J-Link_Commander#Using_J-Link_Command_Files) (\*.jlink)

2. [J-Link Script File](https://wiki.segger.com/J-Link_script_files#Using_J-Link_script_files) (\*.JLinkScript)

This application note uses J-Link Commander v7.66g. The J-Link Commander's command line interface is invoked by JLink.exe located in `C:\Program Files\SEGGER\JLink_V766g` (Windows).

### J-Link Command File

The J-Link Command file below is an example to invoke the J-Link Commander in batch processing mode for the [Erase Device](05-se-command-list.md#) operation. The delay (`sleep 500`) in milliseconds after writing the command word is device dependent. It may be shorter or longer.

```sh
connect
swdwritedp 2 0x01000000 ; Select DCI AP
swdwriteap 1 0x1008 ; DCI STATUS
swdreadap 3 
swdreaddp 3 ; Poll till DCI_STATUS.WPENDING (bit 0) is low
swdwriteap 1 0x1000 ; DCI WDATA
swdwriteap 3 0x08 ; Command word 0
sleep 500 ; Delay if necessary
swdwriteap 1 0x1008 ; DCI STATUS
swdreadap 3 
swdreaddp 3 ; Poll till DCI_STATUS.WPENDING (bit 0) is low
swdwriteap 1 0x1000 ; DCI WDATA
swdwriteap 3 0x430F0000 ; command word 1 (Erase Device)
sleep 500 ; Delay if necessary
swdwriteap 1 0x1008 ; DCI STATUS
swdreadap 3 
swdreaddp 3 ; Poll till DCI_STATUS.RDATAVALID (bit 8) is high
swdwriteap 1 0x1004 ; DCI RDATA
swdreadap 3 ; First word
swdreaddp 3 ; Status code is upper 16 bits, total length is lower 16 bits
exit
```

The example below is to run the J-Link Command file (assume `EraseDevice.jlink` is in the J-Link Commander folder) via the Windows DOS command prompt. The target device is EFR32MG21A, and the target interface is 1000 kHz SWD. The SWD speed is hardware dependent; for example, the length of the wires between the programmer and the device debug pins. The speed (frequency) can be higher for shorter wires and lower for longer wires.

```sh
JLink.exe -device EFR32MG21AXXXF1024 -if SWD -speed 1000 -CommandFile EraseDevice.jlink
```

```sh
SEGGER J-Link Commander V7.66g (Compiled Jul 7 2022 10:44:29)
DLL version V7.66g, compiled Jul 7 2022 10:42:43
J-Link Command File read successfully.
Processing script file...
J-Link>connect
J-Link connection not established yet but required for command.
Connecting to J-Link via USB...O.K.
Firmware: Silicon Labs J-Link Pro OB compiled Sep 16 2020 17:10:58
Hardware version: V4.00
S/N: 440048205
License(s): RDI, FlashBP
IP-Addr: DHCP (no addr. received yet)
VTref=3.326V
Device "EFR32MG21AXXXF1024" selected.
Connecting to target via SWD
Found SW-DP with ID 0x6BA02477
DPv0 detected
CoreSight SoC-400 or earlier
Scanning AP map to find all available APs
AP[3]: Stopped AP scan as end of AP map has been reached
AP[0]: AHB-AP (IDR: 0x84770001)
AP[1]: APB-AP (IDR: 0x54770002)
AP[2]: AHB-AP (IDR: 0x84770001)
Iterating through AP map to find AHB-AP to use
AP[0]: Core found
AP[0]: AHB-AP ROM base: 0xE00FE000
CPUID register: 0x410FD213. Implementer code: 0x41 (ARM)
Feature set: Mainline
Found Cortex-M33 r0p3, Little endian.
FPUnit: 8 code (BP) slots and 0 literal slots
Security extension: implemented
Secure debug: enabled
CoreSight components:
ROMTbl[0] @ E00FE000
[0][0]: E00FF000 CID B105100D PID 000BB4C9 ROM Table
ROMTbl[1] @ E00FF000
[1][0]: E000E000 CID B105900D PID 000BBD21 DEVARCH 47702A04 DEVTYPE 00 Cortex-M33
[1][1]: E0001000 CID B105900D PID 000BBD21 DEVARCH 47701A02 DEVTYPE 00 DWT
[1][2]: E0002000 CID B105900D PID 000BBD21 DEVARCH 47701A03 DEVTYPE 00 FPB
[1][3]: E0000000 CID B105900D PID 000BBD21 DEVARCH 47701A01 DEVTYPE 43 ITM
[1][5]: E0041000 CID B105900D PID 002BBD21 DEVARCH 47724A13 DEVTYPE 13 ETM
[1][6]: E0042000 CID B105900D PID 000BBD21 DEVARCH 47701A14 DEVTYPE 14 CSS600-CTI
[0][1]: E0040000 CID B105900D PID 000BBD21 DEVARCH 00000000 DEVTYPE 11 Cortex-M33
[0][2]: E00FD000 CID B105F00D PID 001BB101 TSG
Cortex-M33 identified.
J-Link>swdwritedp 2 0x01000000 ; Select DCI AP
Write DP register 2 = 0x01000000
J-Link>swdwriteap 1 0x1008 ; DCI STATUS
Write AP register 1 = 0x00001008
J-Link>swdreadap 3
Read AP register 3 = 0x000000B1
J-Link>swdreaddp 3 ; Poll till DCI_STATUS.WPENDING (bit 0) is low
Read DP register 3 = 0x00000000
J-Link>swdwriteap 1 0x1000 ; DCI WDATA
Write AP register 1 = 0x00001000
J-Link>swdwriteap 3 0x08 ; Command word 0
Write AP register 3 = 0x00000008
J-Link>sleep 500 ; Delay if necessary
Sleep(500)
J-Link>swdwriteap 1 0x1008 ; DCI STATUS
Write AP register 1 = 0x00001008
J-Link>swdreadap 3
Read AP register 3 = 0x00000000
J-Link>swdreaddp 3 ; Poll till DCI_STATUS.WPENDING (bit 0) is low
Read DP register 3 = 0x00000000
J-Link>swdwriteap 1 0x1000 ; DCI WDATA
Write AP register 1 = 0x00001000
J-Link>swdwriteap 3 0x430F0000 ; command word 1 (Erase Device)
Write AP register 3 = 0x430F0000
J-Link>sleep 500 ; Delay if necessary
Sleep(500)
J-Link>swdwriteap 1 0x1008 ; DCI STATUS
Write AP register 1 = 0x00001008
J-Link>swdreadap 3
Read AP register 3 = 0x00000000
J-Link>swdreaddp 3 ; Poll till DCI_STATUS.RDATAVALID (bit 8) is high
Read DP register 3 = 0x00000100
J-Link>swdwriteap 1 0x1004 ; DCI RDATA
Write AP register 1 = 0x00001004
J-Link>swdreadap 3 ; First word
Read AP register 3 = 0x00000100
J-Link>swdreaddp 3 ; Status code is upper 16 bits, total length is lower 16 bits
Read DP register 3 = 0x00000004
J-Link>exit
Script processing completed.
```

### J-Link Script File

The J-Link Script file below is an example to customize some actions performed by the J-Link Commander for the [Read Serial Number](./05-se-command-list.md#read-serial-number) operation.

```sh
// Function to select DCI AP register bank 0
int SelectDciAp(void)
{
 int stat;
 int value;
 JLINK_SYS_Report("- Select DCI AP register bank 0\n");
 value = 0x01000000; 
 stat = JLINK_CORESIGHT_WriteDP(2, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- DP 2 write failed: ", value);
 }
 return stat;
}
// Function to write command to DCI_WDATA (0x1000) 
int WriteCommandWord(int command)
{
 int stat;
 int value;
 // Poll DCI_STATUS (0x1008) WPENDING bit (bit 0)
 do {
 value = 0x00001008;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
 }
 JLINK_CORESIGHT_ReadAP(3);
 value = JLINK_CORESIGHT_ReadDP(3);
 if (value == -1) {
 JLINK_SYS_Report("- DP 3 read failed");
 return value;
 }
 if ((value & 0x0100) != 0) {
 JLINK_SYS_Report("- RDATAVALID is high, command aborted");
 return -1;
 } 
 } while ((value & 0x01) != 0);
 
 // Write command
 value = 0x00001000;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
 }
 value = command;
 stat = JLINK_CORESIGHT_WriteAP(3, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 3 write failed: ", value);
 return stat;
 }
 JLINK_SYS_Report1("- Write command word ", command); 
 return stat;
}
// Function to read response from DCI_RDATA (0x1004)
int ReadResponse(void)
{
 int stat;
 int value;
 int count;
// Poll DCI_STATUS (0x1008) RDATAVALID bit (bit 8) 
 do {
 value = 0x00001008;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
 }
 JLINK_CORESIGHT_ReadAP(3);
 value = JLINK_CORESIGHT_ReadDP(3);
 if (value == -1) {
 JLINK_SYS_Report("- DP 3 read failed");
 return value;
 }
 } while ((value & 0x0100) != 0x0100);
 
 // Read first 32-bit response word 
 value = 0x00001004;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
 }
 JLINK_CORESIGHT_ReadAP(3);
 value = JLINK_CORESIGHT_ReadDP(3);
 if (value == -1) {
 JLINK_SYS_Report("- DP 3 read failed");
 return value;
 }
 // Get response count
 count = value & 0x00FF; 
 count = count >> 2;
 JLINK_SYS_Report1("- Number of 32-bit response word: ", count);
 // Get response code
 JLINK_SYS_Report1("- Response: ", value);
 stat = value >>16; 
 if (stat != 0) {
 JLINK_SYS_Report1("- Command error: ", stat);
 return -1;
 }
 
 // Read following 32-bit response word
 count = count - 1;
 while (count != 0) {
 // Poll DCI_STATUS (0x1008) RDATAVALID bit (bit 8) 
 do {
 value = 0x00001008;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
 }
 JLINK_CORESIGHT_ReadAP(3);
 value = JLINK_CORESIGHT_ReadDP(3);
 if (value == -1) {
 JLINK_SYS_Report("- DP 3 read failed");
 return value;
 }
 } while ((value & 0x0100) != 0x0100);
 
 // Read 32-bit response word 
 value = 0x00001004;
 stat = JLINK_CORESIGHT_WriteAP(1, value);
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 1 write failed: ", value);
 return stat;
  }
 JLINK_CORESIGHT_ReadAP(3);
 value = JLINK_CORESIGHT_ReadDP(3);
 if (value == -1) {
 JLINK_SYS_Report("- DP 3 read failed");
 return value;
 }
 JLINK_SYS_Report1("- Response: ", value);
 count = count - 1;
 }
 return 0;
}
// Function to connect DCI
int ConnectDci(void)
{ 
 int stat;
 int value;
 
 JLINK_SYS_Report("\n******************************************************");
 JLINK_SYS_Report("Connect to Series 2 Device DCI\n");
 JLINK_SYS_Report("- Select SWD by sending SWD switching sequence\n");
 stat = JLINK_CORESIGHT_Configure(""); 
 if (stat < 0) {
 JLINK_SYS_Report("- SWD connection failed");
 return stat;
 }
 // Manually select debug interface 
 JLINK_SYS_Report("- Clear sticky error flags\n");
 value = 0x0000001E; 
 stat = JLINK_CORESIGHT_WriteDP(0, value); 
 if (stat < 0) {
 JLINK_SYS_Report1("- DP 0 write failed: ", value);
 return stat;
 }
 JLINK_SYS_Report("- Power up system and debug\n");
 value = 0x50000000; 
 stat = JLINK_CORESIGHT_WriteDP(1, value); 
 if (stat < 0) {
 JLINK_SYS_Report1("- DP 1 write failed: ", value);
 return stat;
 }
 
 value = JLINK_CORESIGHT_ReadDP(0); 
 if (value != 0x6BA02477) {
 JLINK_SYS_Report1("- The connected device does not have a Secure Element (ID Code): ", value);
 return -1;
 }
 JLINK_SYS_Report1("- Read SWD-DP ID code: ", value);
 JLINK_SYS_Report("- Select DCI AP register bank 0\n");
 value = 0x01000000; 
 stat = JLINK_CORESIGHT_WriteDP(2, value); 
 if (stat < 0) {
 JLINK_SYS_Report1("- DP 2 write failed: ", value);
 return stat;
 }
 
 JLINK_SYS_Report("- Set up AP defaults\n");
 value = 0x22000002; 
 stat = JLINK_CORESIGHT_WriteAP(0, value); 
 if (stat < 0) {
 JLINK_SYS_Report1("- AP 0 write failed: ", value); 
 return stat;
 }
 JLINK_SYS_Report("Connect to Series 2 Device DCI OK\n");
 JLINK_SYS_Report("******************************************************");
 return 0;
}
// Function to run DCI command
void RunDciCommand(void)
{
 JLINK_SYS_Report("\n******************************************************");
 JLINK_SYS_Report("Read Serial Number\n");
 if (SelectDciAp() == -1) {
 JLINK_SYS_Report("- Select DCI AP failed\n");
 JLINK_SYS_Report("******************************************************\n\n");
 return;
 }
 if (WriteCommandWord(0x00000008) == -1) {
 JLINK_SYS_Report("- Write DCI command (length) failed\n");
 JLINK_SYS_Report("******************************************************\n\n");
 return;
 }
 if (WriteCommandWord(0xFE000000) == -1) {
 JLINK_SYS_Report("- Write DCI command (ID) failed\n");
 JLINK_SYS_Report("******************************************************\n\n");
 return;
 }
 if (ReadResponse() == -1) {
 JLINK_SYS_Report("- Read DCI command response failed\n");
 JLINK_SYS_Report("******************************************************\n\n");
 return;
 }
 JLINK_SYS_Report("Read Serial Number Done\n");
 JLINK_SYS_Report("******************************************************\n\n");
}
// Replace ConfigTargetSettings() in J-Link DLL
void ConfigTargetSettings(void)
{
 if (ConnectDci() == -1) {
 JLINK_SYS_Report("- Connect to DCI failed\n");
 JLINK_SYS_Report("******************************************************\n\n");
 return;
 }
 RunDciCommand();
}
```

The example below is to run the J-Link Script file (assume `ReadSerialNo.JLinkScript` is in the J-Link Commander folder) via the Windows DOS command prompt. The target device is EFR32MG21A, and the target interface is 1000 kHz SWD. The SWD speed is hardware dependent; for example, the length of the wires between the programmer and the device debug pins. It can be higher for shorter wires and lower for longer wires.

```sh
JLink -device EFR32MG21AXXXF1024 -if SWD -speed 1000 -autoConnect 1 -JLinkScriptFile ReadSerialNo.JlinkScript

```

```sh
SEGGER J-Link Commander V7.66g (Compiled Jul 7 2022 10:44:29)
DLL version V7.66g, compiled Jul 7 2022 10:42:43
Connecting to J-Link via USB...O.K.
Firmware: Silicon Labs J-Link Pro OB compiled Sep 16 2020 17:10:58
Hardware version: V4.00
S/N: 440048205
License(s): RDI, FlashBP
IP-Addr: DHCP (no addr. received yet)
VTref=3.328V
Device "EFR32MG21AXXXF1024" selected.
Connecting to target via SWD
ConfigTargetSettings() start
******************************************************
Connect to Series 2 Device DCI
- Select SWD by sending SWD switching sequence
- Clear sticky error flags
- Power up system and debug
- Read SWD-DP ID code: 0x6BA02477
- Select DCI AP register bank 0
- Set up AP defaults
```

```sh
Connect to Series 2 Device DCI OK
******************************************************
******************************************************
Read Serial Number
- Select DCI AP register bank 0
- Write command word 0x00000008
- Write command word 0xFE000000
- Number of 32-bit response word: 0x00000005
- Response: 0x00000014
- Response: 0x00000000
- Response: 0x00000000
- Response: 0xFF818E58
- Response: 0xE53470FE
Read Serial Number Done
******************************************************
ConfigTargetSettings() end
```
