# Building the GATT Database with Profile Toolkit

This section of the document describes the XML syntax used in the Bluetooth Profile Toolkit and walks you through the different options you can use when building Bluetooth services and characteristics.

A few practical GATT database examples are also shown.

## General Limitations

The table below shows the limitations of the GATT database supported by the EFR32BG devices.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="auto"/>
      <col width="auto"/>
  </colgroup>
<thead>
<tr>
<th>Item</th>
<th>Limitation</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td>Maximum number of characteristics</td>
<td>Not limited; practically restricted by the overall number of attributes in the database</td>
<td>All characteristics which do NOT have the property <strong>const=&quot;true&quot;</strong> are included in this count.</td>
</tr>
<tr>
<td>Maximum length of a <strong>type=&quot;user&quot;</strong> characteristic</td>
<td>255 bytes</td>
<td>
  <p>These characteristics are handled by the application, which means that the amount of RAM available for the application will limit this.</p>
  <p>Note: GATT procedures <strong>Write Long Characteristic Values</strong>, <strong>Reliable Writes</strong>, and <strong>Read Multiple Characteristic Values</strong> are not supported for these characteristics.</p>
</td>
</tr>
<tr>
<td>Maximum length of a <strong>type=&quot;utf-8/hex&quot;</strong> characteristic</td>
<td>255 bytes</td>
<td>
  <p>If <strong>const=&quot;true&quot;</strong> then the amount of free flash on the device defines this limit.</p>
  <p>If <strong>const=&quot;false&quot;</strong> then RAM will be allocated for the characteristic for storing its value. The amount of free flash available on the device used defines this.</p>
</td>
</tr>
<tr>
<td>Maximum number of attributes in a single GATT database</td>
<td>255</td>
<td>A single characteristic typically uses 3-5 attributes.</td>
</tr>
<tr>
<td>Maximum number of notifiable characteristics</td>
<td>64</td>
<td>--</td>
</tr>
<tr>
<td>Maximum number of capabilities</td>
<td>16</td>
<td>The logic state of the capabilities will determine the visibility of each service/characteristic.</td>
</tr>
</tbody>
</table>

## \<gatt>

The GATT database along with the services and characteristics must be described inside the XML attribute **\<gatt>**.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>out</em></td>
<td>
  <p>Filename for the GATT C source file</p>
  <p><strong>Value:</strong> Any UTF-8 string. Must be valid as a filename and end with '.c'</p>
  <p><strong>Default:</strong> gatt_db.c</p>
</td>
</tr>
<tr>
<td><em>header</em></td>
<td>
  <p>Filename for the GATT C header file</p>
  <p><strong>Value:</strong> Any UTF-8 string. Must be valid as a filename and end with '.h'</p>
  <p><strong>Default:</strong>  gatt_db.h</p>
</td>
</tr>
<tr>
<td><em>db_name</em></td>
<td>
  <p>GATT database structure name and the prefix for data structures in the GATT C source file.</p>
  <p><strong>Value:</strong> Any UTF-8 string; must be valid in C.</p>
  <p><strong>Default:</strong>  bg_gattdb_data</p>
</td>
</tr>
<tr>
<td><em>name</em></td>
<td>
  <p>Free text, not used by the database compiler</p>
  <p><strong>Value:</strong> Any UTF-8 string</p>
  <p><strong>Default:</strong> Nothing</p>
</td>
</tr>
<tr>
<td><em>prefix</em></td>
<td>
  <p>Prefix to add to each 'id' name for defining the handle macro that can be referenced from the C application.</p>
  <p><strong>Value:</strong> Any UTF-8 string. Must be valid in C.</p>
  <p><strong>Default:</strong> gattdb_</p>
  <p>For example: If prefix=&quot;gattdb_&quot; and id=&quot;temp_measurement&quot; for a particular characteristic, then the following will be generated in the GATT C header file:</p>
  <p>#define gattdb_temp_measurement X (where X is the handle number for the temp_measurement characteristic)</p>
</td>
</tr>
<tr>
<td><em>generic_attribute_service</em></td>
<td>
  <p>If it is set to true, Generic Attribute service and its service_changed characteristic will be added in the beginning of the database. The Bluetooth stack takes care of database structure change detection and will send service_changed notifications to clients when a change is detected. In addition, this will enable the GATT-caching feature introduced in Bluetooth 5.1.</p>
  <p><strong>Values:</strong>
  <p><strong>true:</strong> Generic Attribute service is automatically added to the GATT database and GATT caching is enabled.</p>
  <p><strong>false:</strong> Generic Attribute service is not automatically added to the GATT database and GATT caching is disabled.</p>
  <p><strong>Default:</strong> false</p>
</td>
</tr>
<tr>
<td><em>gatt_caching</em></td>
<td>
  <p>The GATT caching feature is enabled by default if generic_attribute_service is set to true. However, it can be disabled by setting this attribute to false.</p>
</td>
</tr>
</tbody>
</table>

**Example**: A GATT database definition.

```C
<?xml version="1.0" encoding="UTF-8" ?>
<gatt out="my_gatt_db.c" header="my_gatt_db.h" db_name="my_gatt_db_" prefix="my_gatt_"
   generic_attribute_service="true" name="My GATT database">
…
</gatt>
```

## \<capabilities_declare>

The GATT database services and characteristics can be made visible/invisible by using **capabilities**. A capability must be declared in a \<capability> element and all capabilities in a GATT database must be first declared in a \<capabilities_declare> element consisting of a sequence of \<capability> elements. The maximum number of capabilities in a database is 16.

This new functionality does not affect legacy GATT XML databases (prior to Silicon Labs Bluetooth stack version 2.4.x). Because they don't have any capabilities explicitly declared, all services and characteristics will remain visible to a remote GATT client.

**Example**: Capabilities declaration

```C
  <capabilities_declare>
    <capability enable="false">feature_1</capability>
    <capability enable="false">feature_2</capability>
  </capabilities_declare>
```

### \<capability>

Each capability must be declared individually within a \<capabilities_declare> element using the \<capability> element. The \<capability> element has one attribute named "enable" that indicates the capability's default state at database initialization.

The text value of the \<capability> element will be the identifier name for that capability in the generated database C header. Thus, it must be valid in C.

#### Inheritance of Capabilities

Services and characteristics can declare the capabilities that they want to use. If no capabilities are declared, then the following inheritance rules apply:

1. A service that does not declare any capabilities will have all the capabilities from \<capabilities_declare> element.

2. A characteristic that does not declare any capabilities will have all the capabilities from the service that it belongs to. If the service declares a subset of the capabilities in \<capabilities_declare>, then only that subset will be inherited by the characteristic.

3. All attributes of a characteristic inherit the characteristic's capabilities.

#### Visibility

Capabilities can be enabled/disabled to make services and characteristics visible/invisible to a GATT client according with the following logic:

1. A service and all its characteristics are **visible** when **at least one** of its capabilities is **enabled**.

2. A service and all its characteristics are **invisible** when **all** of its capabilities are **disabled**.

3. A characteristic and all its attributes are **visible** when **at least one** of its capabilities is **enabled**.

4. A characteristic and all its attributes are **invisible** when **all** of its capabilities are **disabled.**

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>enable</td>
<td>
  <p>Sets the default state of a capability at database initialization.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true:</strong> Capability is enabled.</p>
  <p><strong>false:</strong> Capability is disabled.</p>
  <p><strong>Default:</strong> true</p>
</td>
</tr>
</tbody>
</table>

**Example**: Capabilities declaration

```C
<capabilities_declare>
	<!-- This capability is enabled by default and the identifier is cap_light -->
	<capability enable="true">cap_light</capability>
	<!-- This capability is disabled by default and the identifier is cap_color -->
	<capability enable="false">cap_color</capability>
</capabilities_declare>
```

## \<service>

The GATT service definition is done with the XML attribute **\<service>** and its parameters.

The following table below describes the parameters that can be used for defining the related values.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>uuid</em></td>
<td>
  <p>Universally Unique Identifier. The UUID uniquely identifies a service. 16-bit values are used for the services defined by the Bluetooth SIG and 128-bit UUIDs can be used for vendor specific implementations.</p>
  <p><strong>Range:</strong></p>
  <p><strong>0x0000 – 0xFFFF</strong>: Reserved for Bluetooth SIG standardized services</p>
  <p><strong>0x00000000-0000-0000-0000-000000000000 - 0xFFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF</strong>: Reserved for vendor specific services.</p>
</td>
</tr>
<tr>
<td><em>id</em></td>
<td>
  <p>The ID is used to identify a service within the service database and can be used as a reference from other services (include statement). Typically, this does not need to be used.</p>
  <p><strong>Value:</strong></p>
  <p>Any UTF-8 string</p>
</td>
</tr>
<tr>
<td><em>type</em></td>
<td>
  <p>The type field defines whether the service is a primary or a secondary service. Typically this does not need to be used.</p>
  <p><strong>Values:</strong></p>
  <p><strong>primary</strong>: a primary service</p>
  <p><strong>secondary</strong>: a secondary service</p>
  <p><strong>Default</strong>: primary</p>
</td>
</tr>
<tr>
<td><em>advertise</em></td>
<td>
  <p>This field defines if the service UUID is included in the advertisement data.</p>
  <p>The advertisement data can contain up to 13 16-bit UUIDs or one (1) 128-bit UUID.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: UUID included in advertisement data</p>
  <p><strong>false</strong>: UUID not included in advertisement data</p>
  <p><strong>Default</strong>: false</p>
  <p><strong>Note:</strong> You can override the advertisement data with the GAP API, in which case this is not valid.</p>
</td>
</tr>
</tbody>
</table>

**Example**: A Generic Access Profile (GAP) service definition.

```C
<!-- Generic Access Service -->
<service uuid="1800">
      …
</service>
```

**Example**: A vendor-specific service definition.

```C
<!-- A vendor specific service -->
<service uuid="25be6a60-2040-11e5-bd86-0002a5d5c51b">
      …
</service>
```

**Example**: A Heart Rate service definition with UUID included in the advertisement data and ID "hrs".

```C
<!-- Heart Rate Service -->
<service uuid="180D" id="hrs" advertise="true">
      …
</service>
```

>Note: You can generate your own 128-bit UUIDs at: [http://www.itu.int/en/ITU-T/asn1/Pages/UUID/uuids.aspx](http://www.itu.int/en/ITU-T/asn1/Pages/UUID/uuids.aspx)

### \<capabilities>

A service can declare the capabilities it has with a \<capabilities> element. The element consists of a sequence of \<capability> elements whose identifiers **must also be part** of the \<capabilities_declare> element. The attribute "enable" has no effect in the capabilities declared within this context so it can be excluded.

If a service does not declare any capabilities, it will have **all the capabilities** from **\<capabilities_declare>**  per the **inheritance rules**.

A service and all its characteristics will be **visible** when **at least one** of its capabilities is **enabled** and **invisible** when **all its capabilities** are **disabled**.

**Example**: Capabilities declaration

```C
<capabilities>
	<capability>cap_light</capability>
	<capability>cap_color</capability>
</capabilities>
```

### \<informativeText>

The XML element \<informativeText> can be used for informative purposes (commenting) and is not exposed in the actual GATT database.

### \<include>

A service can be included within another service by using the XML attribute **\<include>**.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>id</em></td>
<td>
  <p>ID of the included service</p>
  <p><strong>Value:</strong></p>
  <p>ID of another service</p>
</td>
</tr>
</tbody>
</table>

**Example**: Including Heart Rate service within the GAP service.

```C
<!-- Generic Access Service -->
<service uuid="1800">
      <!-- Include HR Service -->
      <include id="hrs" />
      …
</service>
```

## \<characteristic>

All the characteristics exposed by a service are defined with the XML attribute **\<characteristic>** and its parameters, which must be used inside the **\<service>** XML attribute tags.

The table below describes the parameters that can be used for defining the related values.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>uuid</em></td>
<td>
<p>Universally Unique Identifier. The UUID uniquely identifies a characteristic.</p>
<p>16-bit values are used for the services defined by the Bluetooth SIG and 128-bit UUIDs can be used for vendor specific implementations.</p>
<p><strong>Range:</strong></p>
<p><strong>0x0000 – 0xFFFF</strong>: Reserved for Bluetooth SIG standardized characteristics.</p>
<p><strong>0x00000000-0000-0000-0000-000000000000 to 0xFFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF</strong>:</p>
<p>Reserved for vendor specific characteristics.</p>
</td>
</tr>
<tr>
<td><em>id</em></td>
<td>
  <p>The ID is used to identify a characteristic. The ID is used within a C application to read and write characteristic values or to detect if notifications or indications are enabled or disabled for a specific characteristic.</p>
  <p>When the project is built, the generated GATT C header file contains a macro with the characteristic 'id' and corresponding handle value.</p>
  <p><strong>Value:</strong> Any UTF-8 string</p>
</td>
</tr>
<tr>
<td><em>const</em></td>
<td>
  <p>Defines if the value stored in the characteristic is a constant.</p>
  <p><strong>Default:</strong> false</p>
</td>
</tr>
<tr>
<td><em>name</em></td>
<td>Free text, not used by the database compiler.    **Value:**Any UTF-8 string    <strong>Default:</strong> Nothing</td>
</tr>
</tbody>
</table>

**Example**: Adding Device name characteristic into GAP service.

```C
<!-- Generic Access Service -->
<service uuid="1800">
      <!-- Device name -->
      <characteristic uuid="2a00">
            …
      </characteristic>
      …
</service>
```

**Example**: Adding a vendor-specific characteristic into a vendor-specific service with ID.

```C
<!-- A vendor specific service -->
<service uuid="25be6a60-2040-11e5-bd86-0002a5d5c51b">
      <!-- My proprietary data -->
      <characteristic uuid="59cd69c0-2043-11e5-a717-0002a5d5c51b" id="mydata">
            …
      </characteristic>
      …
</service>
```

### \<capabilities>

A characteristic can declare the capabilities it has with a \<capabilities> element. The element consists of a sequence of \<capability> elements whose identifiers **must also be declared (or fully inherited)** by the parent service. The attribute "enable" has no effect in the capabilities declared within this context so it can be excluded.

If a characteristic does not declare any capabilities it will have **all the capabilities** from the **service** that it belongs to per the **inheritance rules**. All attributes of a characteristic inherit the characteristic's capabilities.

A characteristic and all its attributes will be **visible** when **at least one** of its capabilities is **enabled** and **invisible** when **all its capabilities** are **disabled**.

**Example**: Capabilities declaration

```C
<capabilities>
	<capability>cap_light</capability>
	<capability>cap_color</capability>
</capabilities>
```

### \<properties>

The characteristic's access properties and its permission level are defined by the children attributes of the XML attribute **\<properties>**, which must be used inside **\<characteristic>** XML attribute tags. A characteristic can have multiple access properties at the same time—for example, it can be readable, writable, or both. Each access property can have a different permission level (for example, encrypted or authenticated).

The following table lists the possible access properties. Each row in the table defines a new attribute under the **\<properties>** attribute.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Attribute</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>read</em></td>
<td>
  <p>Characteristic can be read by a remote device.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Characteristic can be read</p>
  <p><strong>false</strong>: Characteristic cannot be read</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>write</em></td>
<td>
  <p>Characteristic can be written by a remote device</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Characteristic can be written</p>
  <p><strong>false</strong>: Characteristic cannot be written</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>write_no_response</em></td>
<td>
  <p>Characteristic can be written by a remote device. Write without response is not acknowledged over the Attribute Protocol.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Characteristic can be written</p>
  <p><strong>false</strong>: Characteristic cannot be written</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>notify</em></td>
<td>
  <p>Characteristic has the notify property and characteristic value changes are notified over the Attribute Protocol. Notifications are not acknowledged over the Attribute Protocol.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Characteristic has notify property.</p>
  <p><strong>false</strong>: Characteristic does not have notify property.</p>
  <p><strong>Default</strong>: false</p>
  <p><strong>Note</strong>: The <em>notify</em> attribute is stored in the SIG defined Client Characteristic Configuration Descriptor (a descriptor with the UUID 0x2902, which will be autogenerated when notifications are enabled). If you manually add a CCCD to the characteristic, the descriptor’s value will overwrite this setting.</p>
</td>
</tr>
<tr>
<td><em>indicate</em></td>
<td>
  <p>Characteristic has the indicate property and characteristic value changes are indicated over the Attribute Protocol. Indications are acknowledged over the Attribute Protocol.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Characteristic has indicate property.</p>
  <p><strong>false</strong>: Characteristic does not have indicate property.</p>
  <p><strong>Default</strong>: false</p>
  <p><strong>Note</strong>: The <em>indicate</em> attribute is stored in the SIG defined Client Characteristic Configuration Descriptor (a descriptor with the UUID 0x2902, which will be autogenerated when indications are enabled). If you manually add a CCCD to the characteristic, the descriptor’s value will overwrite this setting.</p>
</td>
</tr>
<tr>
<td><em>reliable_write</em></td>
<td>
  <p>Allows using a reliable write procedure to modify an attribute; this is just a hint to a GATT client. The Bluetooth stack always allows the use of reliable writes to modify attributes.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Reliable write enabled.</p>
  <p><strong>false</strong>: Reliable write disenabled.</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
</tbody>
</table>

The following table lists the permission levels or security requirements. Any security requirement can be assigned to any access property.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Attribute</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>authenticated</em></td>
<td>
  <p>Accessing the characteristic value requires an authentication. To access the characteristic with this property the remote device has to be bonded using MITM protection and the connection must be also encrypted.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Authentication is required</p>
  <p><strong>false</strong>: Authentication is not required</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>encrypted</em></td>
<td>
  <p>Accessing the characteristic value requires an encrypted link. Devices with iOS 9.1 or higher must also be bonded at least with Just Works pairing.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Encryption is required</p>
  <p><strong>false</strong>: Encryption is not required</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>bonded</em></td>
<td>
  <p>Accessing the characteristic value requires an encrypted link. Devices must also be bonded at least with Just Works pairing.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Bonding and encryption are required</p>
  <p><strong>false</strong>: Bonding is not required</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
</tbody>
</table>

**Example**: Device name characteristic with *const* and *read* properties.

```C
<!-Device Name-->
<characteristic const = true uuid="2a00">
<properties>
	<read authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: Device name characteristic with *read* and *write* properties to allow the value to be modified by the remote device.

```C
<!-Device Name-->
<characteristic uuid="2a00">
<properties>
	<read authenticated="false" bonded="false" encrypted="false"/>
        <write authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: Heart Rate Measurement characteristic with *notify* property.

```C
<!-Heart Rate Measurement -->
<characteristic uuid="180D">
<properties>
	<notify authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: Characteristic with                 *encrypted read* property.

```C
<!-Device Name-->
<characteristic uuid="1234">
<properties>
        <read authenticated="false" bonded="false" encrypted="true"/>
</properties>
</characteristic>
```

**Example**: Characteristic with                 *authenticated write* property.

```C
<!-Device Name-->
<characteristic uuid="1234">
<properties>
	<write authenticated="true" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: Characteristic with                 *authenticated indicate* properties.

```C
<!-Descriptor value changed -->
<characteristic uuid="2A7D">
<properties>
	<indicate authenticated="true" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

### \<value>

The data type and length for a characteristic is defined with the XML attribute **\<value>** and its parameters, which must be used inside the **\<characteristic>** XML attribute tags.

The table below describes the parameters that can be used for defining the related values.

<table>
  <colgroup>
      <col width="25%"/>
      <col width="75%"/>
  </colgroup>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td><em>length</em></td>
<td>
  <p>Defines a fixed length for the characteristic or the maximum length if <em><strong>variable_length</strong></em> is true. If length is not defined and there is a value (e.g. data exists inside &lt;value&gt;&lt;/value&gt;), then the value length is used to define the length.</p>
  <p>If both <em><strong>length</strong></em> and value are defined, then the following rules apply:
    <ol>
      <li>If <em><strong>variable_length</strong></em> is false and <em><strong><strong>length</strong></strong></em>  is bigger than the value's length, then the value will be padded with 0's at the end to match the attribute's <em><strong>length</strong>.</em></li>
      <li>If <em><strong>length</strong></em>  is smaller than the value's length, then the value will be clipped to match <em><strong>length</strong></em>, regardless of whether <em><strong>variable_length</strong></em> is true or false.</li>
    </ol>
  </p>  
  <p><strong>Range:</strong></p>
  <p><strong>0 – 255</strong>: Length in bytes if <em><strong>type</strong></em> is 'hex', 'utf-8', or 'user'</p>
  <p><strong>0 – 512:</strong> Length in bytes if <strong>type</strong> is 'user'</p>
  <p><strong>Default</strong>: 0</p>
</td>
</tr>
<tr>
<td><em>variable_length</em></td>
<td>
  <p>Defines that the value is of variable length. The maximum length must also be defined with the <em><strong>length</strong></em> attribute or by defining a value. If both <em><strong>length</strong></em> and value are defined, then the rules described in <em><strong>length</strong></em> apply.</p>
  <p><strong>Values:</strong></p>
  <p><strong>true</strong>: Value is of variable length</p>
  <p><strong>false</strong>: Value has a fixed length</p>
  <p><strong>Default</strong>: false</p>
</td>
</tr>
<tr>
<td><em>type</em></td>
<td>
  <p>Defines the data type.</p>
  <p><strong>Values:</strong></p>
  <p><strong>hex</strong>: Value type is hex</p>
  <p><strong>utf-8</strong>: Value is a string</p>
  <p><strong>user:</strong>  When the characteristic type is marked as type=&quot;user&quot;, the application is responsible for initializing the characteristic value and also providing it, for example, when read operation occurs. The Bluetooth stack does not initialize the value or automatically provide the value when it is being read. When this is set, the Bluetooth stack generates <em><strong>gatt_server_user_read_request</strong></em> or <em><strong>gatt_server_user_write_request</strong></em>, which must be handled by the application.</p>
  <p><strong>Default</strong>: utf-8</p>
</td>
</tr>
</tbody>
</table>

**Example**: Heart Rate Measurement characteristic with *notify* property and fixed length of two (2) bytes.

```C
<!-Heart Rate Measurement -->
<characteristic uuid="180D">
<value length="2" type="hex" variable_length = "false"/>
<properties>
<notify authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: A variable length vendor-specific characteristic with maximum length of 20 bytes.

```C
<!-My proprietary data -->
<characteristic uuid="59cd69c0-2043-11e5-a717-0002a5d5c51b" id="mydata">
<value variable_length="true" length="20" type="hex" />
<properties>
	<notify authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

**Example**: The value and length of a characteristic can also be defined by typing the actual value inside the \<value> tags.

```C
<characteristic const="true" id="device_name" name="Device Name" sourceId="org.bluetooth.characteristic.gap.device_name" uuid="2A00">
<value length="17" type="utf-8" variable_length="false">EFR32 BGM111</value>
<properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
</properties>
</characteristic>
```

In the above example, the value is "**EFR32 BGM111****"** and the length is 17 bytes.

**Example**: Defining both length and value with length **bigger** than the value's length.

```C
<!-- Device name -->
<characteristic uuid="2a00">
     <properties read="true" />
     <value type="hex" length="4" variable_length="false">0102</value>
</characteristic>
```

In the example above, the value will be "**01020000****"** because the ***length*** is bigger than the value's length and the value gets padded with 0's.

**Example**: Defining both length and value with length **smaller** than the value's length.

```C
<!-- Device name -->
<characteristic uuid="2a00">
     <properties read="true" />
     <value type="hex" length="2" variable_length="false">01020304</value>
</characteristic>
```

In the example above, the value will be "**0102****"** because the ***length*** is smaller than the value's length, so the value gets clipped to match the*****length*****.

### \<descriptor>

The XML element \<descriptor> can be used to define a generic characteristic descriptor.

Descriptor properties are defined by the \<properties> element and only read and/or write access is allowed. Value is defined by \<value> element the same way as for characteristics values.

>**Note**: If you manually add a Client Characteristic Configuration Descriptor (UUID: 0x2902), its value will overwrite the *notify*/*indicate* properties of its characteristic. If no CCCD added manually, it will be generated automatically if the characteristic has enabled notification or indication.

**Example**: Adding a characteristic descriptor with type UUID 2908.

```C
<characteristic uuid="2a4d" id="hid_input">
<properties notify="true" read="true" />
<value length="3" />
      <descriptor const="false" discoverable="true" id="" name="Custom Descriptor" sourceId="" uuid="2908">
        <properties>
          <read authenticated="false" bonded="false" encrypted="false"/>
        </properties>
        <value length="0" type="hex" variable_length="false">00</value>
      </descriptor>
</characteristic>
```

### \<description>

Characteristic user description values are defined with the XML attribute \<description>, which must be used inside the \<characteristic> XML attribute tags.

Characteristic user description is an optional value. It is exposed to the remote device and can be used, for example, to provide a user-friendly description of the characteristic shown in the application's user interface.

**Example**: Constant string "Heart Rate Measurement"

```C
<characteristic uuid="2a37">
<properties>
	<notify authenticated="false" bonded="false" encrypted="false"/>
</properties>
<description> Heart Rate Measurement </description>
</characteristic>
```

Properties element can be used to allow remote modification of an attribute.

**Example:** Allow remote reading but require bonding for writing

```C
<characteristic uuid="2a37">
	<properties>
		<read authenticated="false" bonded="false" encrypted="true"/>
		<write authenticated="false" bonded="true" encrypted="false"/>
	</properties>
</characteristic>
```

>**Note**: If a description is writable, then the GATT Parser automatically adds the extended properties attribute with *writable_auxiliaries* bit set to be Bluetooth-compliant.

### \<aggregate>

The XML element \<aggregate> enables the creation of an aggregated characteristic format descriptor by automatically converting IDs to attribute handles.

Attribute IDs should refer to characteristic presentation format descriptors.

**Example:** Adding a characteristic aggregate

```C
<characteristic uuid="da8a80c0-829d-498f-b70b-e85c95e0f839">
  <properties notify="true" read="true"/>
  <value length="10" />
  <aggregate>
    <attribute id="format1" />
    <attribute id="format2" />
  </aggregate>
</characteristic>
```

## GATT Examples

**Example:** A full GAP service with device name and appearance characteristics as constant values with *read* property.

```C
<?xml version="1.0" encoding="UTF-8" ?>
<gatt>
<!--Generic Access-->
<service advertise="false" name="Generic Access" requirement="mandatory" sourceId="org.bluetooth.service.generic_access" type="primary" uuid="1800">
    <informativeText>Abstract: The generic_access service contains generic information about the device. All available
       Characteristics are readonly. </informativeText>
    <!--Device Name-->
   <characteristic const="true" id="device_name" name="Device Name" sourceId="org.bluetooth.characteristic.gap.device_name"
     uuid="2A00">
      <informativeText/>
      <value length="17" type="utf-8" variable_length="false">EFR32 BGM111</value>
      <properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
    <!--Appearance-->
    <characteristic const="true" name="Appearance" sourceId="org.bluetooth.characteristic.gap.appearance" uuid="2A01">
      <informativeText>Abstract: The external appearance of this device. The values are composed of a category (10-bits) and sub-categories (6-bits). </informativeText>
      <value length="2" type="hex" variable_length="false">0000</value>
      <properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
  </service>
</gatt>
```

**Example:** Link Loss and Immediate Alert services.

```C
<?xml version="1.0" encoding="UTF-8" ?>
<gatt>
<!--Link Loss-->
  <service advertise="false" id="link_loss" name="Link Loss" requirement="mandatory" sourceId="org.bluetooth.service.link_loss" type="primary" uuid="1803">
    <!--Alert Level-->
    <characteristic const="false" id="alert_level" name="Alert Level" sourceId="org.bluetooth.characteristic.alert_level" uuid="2A06">
      <value length="1" type="hex" variable_length="false"/>
      <properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
        <write authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
  </service>
  <!--Immediate Alert-->
  <service advertise="false" id="immediate_alert" name="Immediate Alert" requirement="mandatory" sourceId="org.bluetooth.service.immediate_alert" type="primary" uuid="1802">
    <!--Alert Level-->
    <characteristic const="false" id="alert_level" name="Alert Level" sourceId="org.bluetooth.characteristic.alert_level" uuid="2A06">
      <value length="1" type="hex" variable_length="false"/>
      <properties>
        <write_no_response authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
  </service>
</gatt>
```

**Example:** GATT database with capabilities

```C
<gatt db_name="light_gattdb" out="gatt_db.c" header="gatt_db.h" generic_attribute_service="true">
  <capabilities_declare>
    <capability enable="true">cap_light</capability>
    <capability enable="false">cap_color</capability>
  </capabilities_declare>
  <!--Light Service-->
  <service advertise="false" id="light_service" name="Light Service" requirement="mandatory" sourceId="" type="primary" uuid="257f993d-756e-baa6-e69c-8b101e4e6b3f">
    <informativeText>Info about custom service</informativeText>
    <capabilities>
      <capability>cap_light</capability>
      <capability>cap_color</capability>
    </capabilities>
    <!--Ligth Control-->
    <characteristic const="false" id="light_control" name="Ligth Control" sourceId="" uuid="85e82a1c-8423-610b-9fea-5ad999445231">
      <description>User description</description>
      <informativeText/>
      <capabilities>
        <capability>cap_light</capability>
      </capabilities>
      <value length="0" type="user" variable_length="false"/>
      <properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
        <write authenticated="false" bonded="false" encrypted="false"/>
        <write_no_response authenticated="false" bonded="false" encrypted="false"/>
        <reliable_write authenticated="false" bonded="false" encrypted="false"/>
        <indicate authenticated="false" bonded="false" encrypted="false"/>
        <notify authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
    <!--Color control-->
    <characteristic const="false" id="color_control" name="Color control" sourceId="" uuid="16b90591-c54a-e7c9-413e-a82748a1e783">
      <informativeText/>
      <capabilities>
        <capability>cap_color</capability>
      </capabilities>
      <value length="0" type="user" variable_length="false"/>
      <properties>
        <read authenticated="false" bonded="false" encrypted="false"/>
        <write authenticated="false" bonded="false" encrypted="false"/>
      </properties>
    </characteristic>
  </service>
</gatt>
```

- If the capabilities cap_light and cap_color are enabled, the entire light_service will be visible.

- If the capability cap_light is disabled, the characteristic light_control will be invisible.

- If the capability cap_color is disabled, the characteristic color_control will be invisible.
