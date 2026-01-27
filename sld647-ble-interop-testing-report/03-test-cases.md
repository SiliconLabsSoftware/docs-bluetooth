# Test Cases

<table>
<colgroup>
    <col width="8%"/>
    <col width="17%"/>
    <col width="25%"/>
    <col width="50%"/>
    </colgroup>
<thead>
<tr>
<th>Test ID</th>
<th>Test Type</th>
<th>Test Sub Type</th>
<th>Procedure</th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Bluetooth LE Scanning</td>
<td></td>
<td>
    <p>Reset device, scan on phone.</p>
    <p>Pass if device is found by phone within 1000 ms from scanning start time.</p>
    <p>Maximum number of retries: 5</p>
</td>
</tr>
<tr>
<td>2</td>
<td>Bluetooth LE Connection</td>
<td></td>
<td>
    <p>Attempt connection between phone as central and device as peripheral.</p>
    <p>Measure connection time from start to successful completion.</p>
    <p>Pass if the connection is established successfully within 1000 ms.</p>
    <p>Maximum number of retries: 5</p>
</td>
</tr>
<tr>
<td>3</td>
<td>Bluetooth LE Discovery</td>
<td></td>
<td>Pass if all GATT services from device under test are read by phone within 1200 ms.</td>
</tr>
<tr>
<td>4.1</td>
<td>GATT Operations</td>
<td>Read Only Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Read value = 0x55</li>
    </ul>
    <p>Pass if expected value is read by phone.</p>
</td>
</tr>
<tr>
<td>4.2</td>
<td>GATT Operations</td>
<td>Read Only Length 255</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Read</li>
        <li>Value = 0-254</li>
    </ul>
    <p>Pass if phone reads all 255 bytes match the value described above.</p>
</td>
</tr>
<tr>
<td>4.3</td>
<td>GATT Operations</td>
<td>Write Only Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Write</li>
    </ul>
    <p>Phone writes 1 byte to the characteristic with value 0.</p>
    <p>Pass if the operation does not generate an error on the phone.</p>
</td>
</tr>
<tr>
<td>4.4</td>
<td>GATT Operations</td>
<td>Write Only length 255</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Write</li>
    </ul>
    <p>Phone writes 255 bytes to the characteristic with value 0.</p>
    <p>Pass if the operation does not generate an error on the phone.</p>
</td>
</tr>
<tr>
<td>4.5</td>
<td>GATT Operations</td>
<td>Write Without Response Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Write Without Response</li>
    </ul>
    <p>Phone writes 1 byte to the characteristic with value 0.</p>
    <p>Pass if the operation does not generate an error on the phone.</p>
</td>
</tr>
<tr>
<td>4.6</td>
<td>GATT Operations</td>
<td>Write Without Response Length 255</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Write Without Response</li>
    </ul>
    <p>Phone writes 255 bytes to the characteristic with value 0.</p>
    <p>Pass if the operation does not generate an error on the phone.</p>
</td>
</tr>
<tr>
<td>4.7</td>
<td>GATT Operations</td>
<td>Notify length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Notify</li>
        <li>Value = 0x55</li>
    </ul>
    <p>Phone subscribes to notifications on the characteristic which shall trigger a one-shot 100 ms timer on the embedded firmware.</p>
    <p>Once the timer expires, the notification shall be sent with defined value.</p>
    <p>Pass if the subscription is done successfully, notification is received within 300 ms after subscribing, and the value is 0x55.</p>
    <p>Maximum number of retries: 5</p>
</td>
</tr>
<tr>
<td>4.8</td>
<td>GATT Operations</td>
<td>Notify length MTU - 3</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Notify</li>
    </ul>
    <p>Phone subscribes to indications on the characteristic which shall trigger a one-shot 100ms timer on the embedded firmware.</p>
    <p>Once the timer expires, the indication shall be sent.</p>
    <p>The amount of bytes sent shall be MTU - 3 bytes (maximum supported by an indication) where the first byte value is 0 and the subsequent bytes will previous one +1 (e.g., byte 0 value is 0, byte 1 value is 1,... and byte N value is N).</p>
    <p>Pass if the subscription is done successfully, indication is received within 300 ms after subscribing and the value matches as described above.</p>
    <p>Maximum number of retries: 5</p>
</td>
</tr>
<tr>
<td>4.9</td>
<td>GATT Operations</td>
<td>Indicate Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Indicate</li>
        <li>Value = 0x55</li>
    </ul>
    <p>Phone subscribes to indications on the characteristic which shall trigger a one-shot 100 ms timer on the embedded firmware.</p>
    <p>Once the timer expires, the indication shall be sent with defined value.</p>
    <p>Pass if the subscription is done successfully, indication is received within 300 ms after subscribing and the value is 0x55.</p>
    <p>Maximum number of retries: 5 times</p>
</td>
</tr>
<tr>
<td>4.10</td>
<td>GATT Operations</td>
<td>Indicate length MTU - 3</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Indicate</li>
    </ul>
    <p>Phone subscribes to indications on the characteristic which shall trigger a one-shot 100 ms timer on the embedded firmware.</p>
    <p>Once the timer expires, the indication shall be sent.</p>
    <p>The amount of bytes sent shall be MTU - 3 bytes (maximum supported by an indication) where the first byte value is 0 and the subsequent bytes will previous one +1 (e.g., byte 0 value is 0, byte 1 value is 1,... and byte N value is N).</p>
    <p>Pass if the subscription is done successfully, indication is received within 300 ms after subscribing and the value matches as described above.</p>
    <p>Maximum number of retries: 5</p>
</td>
</tr>
<tr>
<td>5.1</td>
<td>Characteristic</td>
<td>Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Read, Write</li>
        <li>Value = 0x00</li>
    </ul>
    <p>Write 1 byte with value 0x55 then read value back.</p>
    <p>Pass if value read is 0x55.</p>
</td>
</tr>
<tr>
<td>5.2</td>
<td>Characteristic</td>
<td>Length 255</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Read, Write</li>
        <li>Value = 0x00 on all bytes</li>
    </ul>
    <p>Phone writes 255 bytes where the first byte value is 0 and the subsequent bytes will previous one +1 (e.g., byte 0 value is 0, byte 1 value is 1,... and byte 254 value is 254). Read values back.</p>
    <p>Pass if all 255 bytes match the value described above.</p>
</td>
</tr>
<tr>
<td>5.3</td>
<td>Characteristic</td>
<td>Length Variable 4</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 4</li>
        <li>Variable Length = True</li>
        <li>Properties = Read, Write</li>
        <li>Value = 0x00 (only 1 byte so that length is also 1)</li>
    </ul>
    <p>Phone writes 1 byte with value 0x55 and read back.</p>
    <p>Pass if get only 1 byte with value0x55.</p>
    <p>Phone writes 4 bytes with value 0x66 and read back.</p>
    <p>Pass if get 4 bytes with value 0x66.</p>
</td>
</tr>
<tr>
<td>5.4</td>
<td>Characteristic</td>
<td>Const Length 1</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Read, Write, Const</li>
        <li>Value = 0x55</li>
    </ul>
    <p>1: Phone reads 1 byte.</p>
    <p>Pass if the value is 0x55.</p>
    <p>2: Phone writes value 0.</p>
    <p>Pass if ATT error code 0x03 "Write Not Permitted" is received.</p>
</td>
</tr>
<tr>
<td>5.5</td>
<td>Characteristic</td>
<td>Const Length 255</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 255</li>
        <li>Properties = Read, Write, Const</li>
        <li>Value = The first byte value is 0 and the subsequent bytes will previous one +1 (e.g., byte 0 value is 0, byte 1 value is 1 ... and byte 254 value is 254)</li>
    </ul>
    <p>1: Phone reads 255 byte.</p>
    <p>Pass if the value is as described above.</p>
    <p>2: Phone writes value 0 to all bytes.</p>
    <p>Pass if ATT error code 0x03 "Write Not Permitted" is received.</p>
</td>
</tr>
<tr>
<td>5.6</td>
<td>Characteristic</td>
<td>User Len 1</td>
<td>
    <ul>
        <li>Type = user</li>
        <li>Length = 1</li>
        <li>Properties = Read, Write</li>
    </ul>
    <p>Phone writes 1 byte with value 0x55 then reads value back.</p>
    <p>Pass if value read is 0x55.</p>
<td>
</tr>
<tr>
<td>5.7</td>
<td>Characteristic</td>
<td>User Len 255</td>
<td>
    <ul>
        <li>Type = user</li>
        <li>Length = 1</li>
        <li>Properties = Read, Write</li>
    </ul>
    <p>Phone writes 255 bytes where the first byte value is 0 and the subsequent bytes will previous one +1 (e.g., byte 0 value is 0, byte 1 value is 1,... and byte 254 value is 254).</p>
    <p>Pass if value read is as described above.</p>
</td>
</tr>
<tr>
<td>5.8</td>
<td>Characteristic</td>
<td>User Len Variable 4</td>
<td>
    <ul>
        <li>Type = user</li>
        <li>Length = 4</li>
        <li>Variable Length = True</li>
        <li>Properties = Read, Write</li>
    </ul>
    <p>1: Phone writes 1 byte with value 0x55 and read back.</p>
    <p>Pass if get only 1 byte (so the length is 1) with value 0x55.</p>
    <p>2: Phone writes 4 bytes with value 0x66 and read back.</p>
    <p>Pass if get 4 bytes (so the length is 4) with value 0x66.</p>
</td>
</tr>
<tr>
<td>6.1</td>
<td>OTA update</td>
<td>OTA update - Acknowledged write</td>
<td>
    <p>Mobile (OTA client) connects to target device with name "IOP Test".</p>
    <p>Mobile shows pop-up, user chooses application gbl file OR Mobile has inbuilt gbl file for updating.</p>
    <p>Mobile requests target device to reboot into DFU mode.</p>
    <p>Mobile sends image to target boards with acknowledged data transfer (write).</p>
    <p>Upload is finished and connection closed, AppLoader on the device reboots back to normal mode.</p>
    <p>Mobile scans and connect with target device.</p>
    <p>Pass: there is no error in mobile side and connect to device with name "IOP Test Update" at the end which is the name in the new image.</p>
</td>
</tr>
<tr>
<td>6.2</td>
<td>OTA update</td>
<td>OTA update - Unacknowledged write</td>
<td>
    <p>Mobile (OTA client) connects to target device with name "IOP Test Update".</p>
    <p>Mobile show pop-up, user chooses application gbl file OR Mobile has inbuilt gbl file for updating.</p>
    <p>Mobile requests target device to reboot into DFU mode.</p>
    <p>Mobile sends image to target boards with unacknowledged data transfer (write without response).</p>
    <p>Upload is finished and connection closed, AppLoader reboots back to normal mode.</p>
    <p>Mobile scans and connect with target device.</p>
    <p>Pass: there is no error in mobile side and connect to device with name "IOP Test" at the end.</p>
</td>
</tr>
<tr>
<td>7</td>
<td>Throughput</td>
<td>Throughput - GATT Notification</td>
<td>
    <p>PHY bitrate: 1Mbps</p>
    <p>Connection interval request: 15 ms</p>
    <p>Number of packets per connection interval for notification:
        <ul>
            <li>Android - 6 packets</li>
            <li>iOS - 4 packages [Max supported by iOS]</li>
        </ul>
    </p>
    <p>The parameters are set as above and device sends the data to the mobile. Mobile computes the throughput (Bytes/S).</p>
    <p>Pass if there is successful data transfer between the device and mobile.</p>
    <p>See Test Results for more details.</p>
</td>
</tr>
<tr>
<td>8.1</td>
<td>Security and Encryption</td>
<td>Security - Pairing</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Encrypted_Read</li>
        <li>Value = 0x55</li>
    </ul>
    <p>Phone writes 1 byte to the characteristic with value 01.</p>
    <p>Mobile changes secure mode on the device to be just works, disables bonding and disconnects with device.</p>
    <p>Mobile pairs with device in just works mode.</p>
    <p>Mobile reads a pre-set characteristic value (0x55) with encrypted read property.</p>
    <p>Pass if there is no error and read value matches the pre-set value (0x55).</p>
</td>
</tr>
<tr>
<td>8.2</td>
<td>Security and Encryption</td>
<td>Security - Authentication</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Authenticated_Read</li>
        <li>Value = 0x55</li>
    </ul>
    <p>Phone writes 1 byte to the characteristic with value 02.</p>
    <p>Mobile changes secure mode on the device to authenticated and disconnect with device.</p>
    <p>Mobile pairs with device with authenticated mode by entering passkey as 123456.</p>
    <p>Mobile reads pre-set characteristic value (0x55) with authenticated read property.</p>
    <p>Pass if there is no error and read value matches the pre-set value (0x55) after pairing.</p>
</td>
</tr>
<tr>
<td>8.3</td>
<td>Security and Encryption</td>
<td>Security -  Bonding</td>
<td>
    <ul>
        <li>Type = hex</li>
        <li>Length = 1</li>
        <li>Properties = Bonded_Read</li>
        <li>Value = 0x55</li>
    </ul>
    <p>Phone writes 1 byte to the characteristic with value 03.</p>
    <p>Mobile changes secure mode in firmware to bonded and disconnects with device.</p>
    <p>Mobile pairs with device with bonded mode by enter passkey is 123456.</p>
    <p>Mobile reads pre-set characteristic value (0x55) with bonded read property.</p>
    <p>Pass if there is no error and read value matches the pre-set value (0x55) after pairing.</p>
</td>
</tr>
</tbody>
</table>
