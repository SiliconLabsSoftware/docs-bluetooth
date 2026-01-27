# Bluetooth Qualification

All products using Bluetooth technology **must go through the Bluetooth SIG's Qualification Process**, even if the product does not have the Bluetooth logo or Bluetooth is not mentioned in the packaging and the documentation. In practice this means that, **before you can sell** a Bluetooth-enabled product to the market, the **product must be qualified** also known as **Qualified Product** that has successfully completed the Bluetooth Qualification Process through the Bluetooth SIG. The qualification listing has an [administrative fee](https://www.bluetooth.com/fee-schedule/) paid by a member to Bluetooth SIG, fee value is depending on your membership type you have with Bluetooth SIG. There are online resources to learn more about the [Bluetooth Qualification Process](https://www.bluetooth.com/develop-with-bluetooth/qualification-listing/) as well as tutorials on the [Qualification Workspace](https://qualification.support.bluetooth.com/hc/en-us/sections/27694138850573-Bluetooth-Qualification-Process), which is the online tool used to complete the Bluetooth Qualification Process. If you need assistance to qualify your product consider reaching out to your nearest [Bluetooth Qualification Consultant](https://www.bluetooth.com/develop-with-bluetooth/qualification-listing/qualification-consultants/).

## Scenario A: Product Listing Using Existing Core Layer Designs

When qualifying your product based on the Silicon Labs Bluetooth stack, you will integrate the pre-qualified designs (known as Design Number - DN) listed in the diagram below, depending on which SDK version was used to build your application. In addition to these software designs, you must also have to integrate a qualified RFPHY design (hardware - SoCs/Modules/Boards) in your product listing. If you are designing with an SoC then you may need to obtain your own RFPHY qualification with the Bluetooth SIG, depending on your hardware design and Bluetooth core spec version. In the latter case, consult your nearest [Bluetooth Qualification Consultant](https://www.bluetooth.com/develop-with-bluetooth/qualification-listing/qualification-consultants/), or Silicon Labs through the support portal, to understand if an existing Silicon Labs RFPHY pre-qualification could be used/inherited. If there are no designs changes in Hardware RFPHY which is following the reference design as shown in datasheets then existing RFPHY design number can be inherited.

The above software-based pre-qualified designs are two/one out of the three/two designs to integrate when proceeding with the [Option 2b step as mentioned in QPRD v4.0 or above](https://www.bluetooth.com/specifications/specs/html/?src=QPRD/out/en/index-en.html&UUID-5b5e9584-8af4-b02c-1198-96be8369ac0b). Customers do not need to do any additional testing for each unmodified layer inherited from an included pre-qualified designs. Unmodified layers are not shown in test plan, given that the test reports are embedded in the pre-qualified designs /components for the SIG to review.

> **Note**: Silicon Labs recommends using the latest SDK or SDK's which are less than 3 years old. Using of old SDK (older than 3 year period) is not recommended unless Subsystem/Core-Controller configurations design exists or it is not affected when inheriting Silicon Laboratories' designs. It is difficult to renew the old expired component/designs in order to be compliant to latest Test Case Reference List (Errata introduced in interim to fix specifications, test specifications and any other bugfix) in core layers i.e. Link Layer, Host stack. It is customers responsibility to qualify SDK/Hardware design if planning to use old SDK. For exceptions (or any pre existing agreements), contact Silicon Labs technical support through the support portal in case there is a need to use an older SDK version.

### A.1: Combining pre-qualified core layer designs with same Bluetooth specification versions

In scenarios, where the hardware (i.e. RFPHY) is following the reference design as shown in the datasheets, and when software/firmware is used as it is, Silicon Labs recommends using appropriate hardware and software versions using the same specification version. For example: Design numbers Q332743 (RFPHY design # for xG24 range of products) and Q317849 (Other controller layers and Host design #, using BLE SDK 9.0.0 and above, SiSDK 2024.12 and above) will help to qualify your product for Bluetooth Core specification 6.0 without any additional testing. Q332743 and Q317849 are listed for Bluetooth Core specification 6.0.

Silicon Labs has created and listed a number of original designs for the software releases as shown below; however, it is recommended to use its subset design numbers in combination with an existing qualified RFPHY design # to make an end product. Refer to [Existing Silicon Labs Design Numbers (DNs) for Creating a New Product Design](#existing-silicon-labs-design-numbers-dns-for-creating-a-new-product-design) for further details.

| **Bluetooth SDK Version** | **Component** | **DN /QDID** |
|---------------------------|---------------|--------------|
| V11.0.0 and above | Channel Sounding, Link Layer (Bluetooth 6.1) and Host stack (Bluetooth 6.1) | [Qualified Design details: Q366996](https://qualification.bluetooth.com/ListingDetails/303046) |
| V9.0.0.0 and above | Channel Sounding, Link Layer (Bluetooth 6.0) and Host stack (Bluetooth 6.0) | [Qualified Design details: Q317849](https://qualification.bluetooth.com/ListingDetails/240988) |
| V6.0.0.0 and above | Link Layer (Bluetooth 5.4) and Host stack (Bluetooth 5.4) | [Launch Studio Listing Details:216508](https://launchstudio.bluetooth.com/ListingDetails/187016) |
| V3.2.x and above | Link Layer (Bluetooth 5.3) | [Launch Studio Listing Details: 178212](https://launchstudio.bluetooth.com/ListingDetails/141145) |
| V2.13.12 and above only | Link Layer (Bluetooth 5.3) | [Launch Studio Listing Details: 178212](https://launchstudio.bluetooth.com/ListingDetails/141145) |
| V3.2.x and above | Host stack (Bluetooth 5.3) | [Launch Studio Listing Details: 175341](https://launchstudio.bluetooth.com/ListingDetails/137791) |
| V2.13.12 and above only | Host stack (Bluetooth 5.3) | [Launch Studio Listing Details: 214504](https://launchstudio.bluetooth.com/ListingDetails/137791) |

### A.2: Combining pre-qualified core layer designs with different Bluetooth specification versions

Scenarios combining RFPHY design with a Bluetooth core specification version different than design, with other controller layers and Host, requires testing for RFPHY to upgrade to the latest version or vice versa.

> **Caution**: Select a specification version that will affect only RFPHY unless changes are required in other controller layers like Link Layer, Host Controller interface, or Channel Sounding. Note also that, if there is no change in test cases from one core spec version to another core spec version, then testing is not required and has to be declared in the Test declaration sheet, also known as the Test plan.

Design number for pre-qualified core layer designs that combine to make a product/products are shown below in [Existing Silicon Labs Design Numbers (DNs) for Creating a New Product Design](#existing-silicon-labs-design-numbers-dns-for-creating-a-new-product-design).

## Scenario B: Product Listing using Existing Core-Controller Configuration Design and Core-Host Configuration Design

In this scenario, the lowest version of two designs is selected to determine the Bluetooth Specification version. These types of pre-qualified designs were previously known as subsystems, where Inter Layer Dependency (ILD) is not checked between Controller and Host designs. Silicon Labs has also listed the Core-Controller Configuration design by integrating a pre-qualified RFPHY core layer design and software-based core layer design in addition to pre-qualified Core-Host Configuration design as shown below. In this case, proceed with [Option 2a step as mentioned in QPRD v4.0 or above](https://www.bluetooth.com/specifications/specs/html/?src=QPRD/out/en/index-en.html&UUID-c53ac674-5f92-d4b9-a8dc-4f3fcfe80067). In the Qualification workspace, **Specify the Design** section, enter the Design Numbers (DNs) / Qualified Design IDs (QDID) for the design on which you are basing your project with no modification. No modification implies that no changes have been made in Hardware by following the Silicon Labs reference design as shown in datasheets and unmodified software (SDK).

Qualified Core-Host Configuration and Core-Controller Configurations designs are listed in the table below.

| **Bluetooth SDK Version and Hardware Part (if any)** | **Core-Host Configuration Design or Core-Controller Configuration Design** | **DN** |
|----------------------------------------------------------|-----------------------------------------------------------------------------------|------------|
| V11.0.0 and above | Core-Host Configuration Design (Bluetooth 6.1) | [Qualified Design details: Q375690](https://qualification.bluetooth.com/ListingDetails/316168) |
| V9.0.0.0 and above with xG27 / xG29 | Core-Controller Configuration Design (Bluetooth 6.0) | [Qualified Design details: Q375771](https://qualification.bluetooth.com/ListingDetails/316281) |
| V9.0.0.0 and above | Core-Host Configuration Design (Bluetooth 6.0) | [Qualified Design details: Q333162](https://qualification.bluetooth.com/ListingDetails/259372) |
| V6.0.0.0 and above | Core-Host Configuration Design (Bluetooth 5.4) | [Qualified Design details: 215749](https://qualification.bluetooth.com/ListingDetails/186130) |
| V6.0.0.0 and above with BGM220S Radio | Core-Controller Configuration Design (Bluetooth 5.4) | [Qualified Design details: 231196](https://qualification.bluetooth.com/ListingDetails/203544) |
| V6.0.0.0 and above with EFR32xG24 series for specific combinations only | **Core-Complete Configuration** Design (Bluetooth 5.4) | [Qualified Design details: 218240](https://qualification.bluetooth.com/ListingDetails/188955) |

## Existing Silicon Labs Design Numbers (DNs) for Creating a New Product Design

In the diagram below, RFPHY Design # refers to design numbers allocated for the existing qualified products at RFPHY level (hardware) for the SOCs, Modules, or Development Kit Boards. Each column indicates the series they belong to.

Other controller layers and host Design numbers refer to design numbers allocated for Silicon Labs qualified products at the software/firmware level covering controller layers LL, HCI, CS, and Host layers. It also indicates which RFPHY features are supported.

See the diagram key below.

![diagram key](./resources/sld1027-diagram-key.png?inline)

### SoCs, Modules, and Development Kit Boards

![diagram](./resources/sld1027-socs-modules-devkits-1.png?inline)

![diagram](./resources/sld1027-socs-modules-devkits-2.png?inline)

![diagram](./resources/sld1027-socs-modules-devkits-3.png?inline)

![diagram](./resources/sld1027-socs-modules-devkits-4.png?inline)

The diagrams above are shown in tabular form below.

<table>
  <colgroup>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="9%"/>
    <col width="5%"/>
    <col width="6%"/>
    <col width="5%"/>
    <col width="9%"/>
    <col width="9%"/>
    <col width="9%"/>
  </colgroup>
  <thead>
    <tr>
      <th colspan="4"></th>
      <th colspan="7"><strong>Compatible with following RFPHY features only</strong></th>
    <tr>
      <th><strong>SoCs, Modules, Dev. Kits boards</strong></th>
      <th><strong>RFPHY Design #</strong></th>
      <th><strong>Other Controller Layers and Host Design #</strong></th>
      <th><strong>BLE SDK version and SiSDK</strong></th>
      <th><strong>Essentials - Rx/Tx</strong></th>
      <th><strong>LE 2 M</strong></th>
      <th><strong>Power Class 1</strong></th>
      <th><strong>LE Coded</strong></th>
      <th><strong>Constant Tone Extension</strong></th>
      <th><strong>Direction Finding - AoA/AoD</strong></th>
      <th><strong>Channel Sounding</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>xG24</td>
      <td>Q332743</td>
      <td>Q317849</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG24</td>
      <td>Q332752</td>
      <td>Q359924</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG22</td>
      <td>231189</td>
      <td>Q361244</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG27, xG29</td>
      <td>205393</td>
      <td>Q367209</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG21, xG13</td>
      <td>231202, 243196</td>
      <td>Q361279</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>SixG301 Series 3</td>
      <td>Q332101</td>
      <td>Q360004</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG28, xG12</td>
      <td>219348, 156906</td>
      <td>Q361415</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG12</td>
      <td>111181</td>
      <td>Q361425</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG1</td>
      <td>145740, 145424</td>
      <td>Q361514</td>
      <td>9.0.0 and above for SiSDK 2024.12 and above</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
  </tbody>
</table>

<table>
  <colgroup>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="9%"/>
    <col width="8%"/>
    <col width="8%"/>
    <col width="9%"/>
    <col width="9%"/>
    <col width="9%"/>
  </colgroup>
  <thead>
    <tr>
      <th colspan="4"></th>
      <th colspan="6"><strong>Compatible with following RFPHY features only</strong></th>
    </tr>
    <tr>
      <th><strong>SoCs, Modules, Dev. Kits boards</strong></th>
      <th><strong>RFPHY Design #</strong></th>
      <th><strong>Other Controller Layers and Host Design #</strong></th>
      <th><strong>BLE SDK version and SiSDK/GSDK</strong></th>
      <th><strong>Essentials - Rx/Tx</strong></th>
      <th><strong>2M</strong></th>
      <th><strong>Class 1</strong></th>
      <th><strong>Coded Phy</strong></th>
      <th><strong>Constant Tone Extension</strong></th>
      <th><strong>AoA/AoD</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>xG24, xG26</td>
      <td>Q306497</td>
      <td>Q301597</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG22</td>
      <td>231189</td>
      <td>Q303906</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG27, xG29</td>
      <td>205393</td>
      <td>Q361523</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG21, xG13</td>
      <td>231202, 243196</td>
      <td>Q304831</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td></td>
      <td></td>
      <td>Q305695</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG28, xG12</td>
      <td>219348, 156906</td>
      <td>Q361532</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG12</td>
      <td>111181</td>
      <td>Q361653</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG1</td>
      <td>145740, 145424</td>
      <td>Q342863</td>
      <td>6.0.0.0 and above but less than 9.0.0 for GSDK 4.3.x and above</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
  </tbody>
</table>

<table>
  <colgroup>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="9%"/>
    <col width="8%"/>
    <col width="8%"/>
    <col width="9%"/>
    <col width="9%"/>
    <col width="9%"/>
  </colgroup>
  <thead>
    <tr>
      <th colspan="4"></th>
      <th colspan="6"><strong>Compatible with following RFPHY features only</strong></th>
    </tr>
    <tr>
      <th><strong>SoCs, Modules, Dev. Kits boards</strong></th>
      <th><strong>RFPHY Design #</strong></th>
      <th><strong>Other Controller Layers and Host Design #</strong></th>
      <th><strong>BLE SDK version and SiSDK/GSDK</strong></th>
      <th><strong>Essentials - Rx/Tx</strong></th>
      <th><strong>2M</strong></th>
      <th><strong>Class 1</strong></th>
      <th><strong>Coded Phy</strong></th>
      <th><strong>Constant Tone Extension</strong></th>
      <th><strong>AoA/AoD</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>xG24</td>
      <td>184327</td>
      <td>Q302606 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG22</td>
      <td>178496, 178495</td>
      <td>Q304391 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG27, xG29</td>
      <td>205393</td>
      <td>Q361959 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG21, xG13</td>
      <td>185220, 243196</td>
      <td>Q305172 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td></td>
      <td></td>
      <td>Q305580 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG28, xG12</td>
      <td>219348, 156906</td>
      <td>Q359672 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG12</td>
      <td>111181</td>
      <td>Q362072 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG1</td>
      <td>145740, 145424</td>
      <td>Q362047 and Q303219</td>
      <td>3.2.x and above but less than 6.0.0 0 for GSDK 3.2.x and above</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
  </tbody>
</table>

<table>
  <colgroup>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="12%"/>
    <col width="9%"/>
    <col width="8%"/>
    <col width="8%"/>
    <col width="9%"/>
    <col width="9%"/>
    <col width="9%"/>
  </colgroup>
  <thead>
    <tr>
      <th colspan="4"></th>
      <th colspan="6"><strong>Compatible with following RFPHY features only</strong></th>
    </tr>
    <tr>
      <th><strong>SoCs, Modules, Dev. Kits boards</strong></th>
      <th><strong>RFPHY Design #</strong></th>
      <th><strong>Other Controller Layers and Host Design #</strong></th>
      <th><strong>BLE SDK version and SiSDK/GSDK</strong></th>
      <th><strong>Essentials - Rx/Tx</strong></th>
      <th><strong>2M</strong></th>
      <th><strong>Class 1</strong></th>
      <th><strong>Coded Phy</strong></th>
      <th><strong>Constant Tone Extension</strong></th>
      <th><strong>AoA/AoD</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>xG24</td>
      <td>184327</td>
      <td>Q304715 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG22</td>
      <td>178496, 178495</td>
      <td>Q304717 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <td>xG27, xG29</td>
      <td>205393</td>
      <td>Q362091 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG21, xG13</td>
      <td>185220, 243196</td>
      <td>Q310686 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td></td>
      <td></td>
      <td>Q362080 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG28, xG12</td>
      <td>219348, 156906</td>
      <td>Q307492 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG12</td>
      <td>111181</td>
      <td>Q362101 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
    <tr>
      <td>xG1</td>
      <td>145740, 145424</td>
      <td>Q304718 and Q305441</td>
      <td>2.13.12 and above but less than 3.2.x</td>
      <td>Yes</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
      <td>No</td>
    </tr>
  </tbody>
</table>

Silicon Labs has also prepared the [Step by Step guide](./03-step-by-step-guide-qualify-product-on-bluetooth-sig-website.md) with instructions on how to inherit product design numbers in the **Qualification Workspace Tool**, and list your product(s) on the **Bluetooth SIG** website. Seek assistance from a **BQTF lab** or **BQC** if the process is unclear or deviates from the recommended steps. Deviations can occur for various reasons; for instance, modifying the matching circuit to meet your product requirements. Note that it is not possible to cover every possible scenario in this article. It is the **member’s responsibility** to complete the Bluetooth Qualification Process for your product under your member company’s account on the Bluetooth SIG website.

> **Note**: You will be charged a **Product Qualification Fee** for the first product submission that includes a specific design. Subsequent products that include the same design will not incur an additional fee. **Silicon Laboratories** cannot pay this fee on your behalf, qualify your product for you, or be held responsible for products listed by member companies.

In general, Silicon Labs does not provide pre-qualifications for adopted profiles. You must obtain these with your own end applications that implement the functionality as per the SIG profile specification.

Silicon Labs has developed several original software designs for the Mesh X2core layers, showcased below.

:::custom-table{width=10%,10%,60%,10%,10%}
| **DN** | **DN Type** | **Products** | **Mesh Spec** | **BLE SDK Version** |
|----|---------|----------|-----------|--------------------|
| 224628 | X2Core Layers | Wireless Gecko Bluetooth Mesh 1.1; contains Mesh Protocol 1.1, Mesh Model 1.1, Mesh BLOB Transfer Model 1.0, and Mesh Device Firmware Update 1.0 specifications | 1.1 | 5.0.3 and above |
| 155722 | X2Core Layers | Wireless Gecko Mesh Model, Wireless Gecko Mesh Model, Time and Scheduler added with Lighting | 1.0.1 | 1.7.x |
| 145819 | X2Core Layers | Wireless Gecko Mesh Model, Wireless Gecko Mesh Model, Lightning only | 1.0.1 | 1.6.x |
| 145768 | X2Core Layers | Wireless Gecko Mesh Profile, Wireless Gecko Mesh Profile | 1.0.1 | 1.6.x |
| 114852 | X2Core Layers | Wireless Gecko Mesh Model, 1.x | 1.0.0 | 1.x.x |
| 114904 | X2Core Layers | Wireless Gecko Mesh Profile, 1.x | 1.0.0 | 1.x.x |
| 101318 | X2Core Layers | Wireless Gecko Mesh Profile, 1.x | 1.0.0 | 1.x.x |
:::

X2Core Layers were also known as Profile Subsystem in the previous QPRD v2.3 or below.

This article is valid as per the current QPRD v4.0 (Qualification Program Reference Document) on the Bluetooth SIG website. If there are any conflicts of opinion in the qualification process, then QPRD v4.0 (Qualification Program Reference Document) or the Bluetooth SIG latest qualification program document have precedence over this article. Alternatively, you can contact a [Bluetooth Qualification Consultant](https://www.bluetooth.com/develop-with-bluetooth/qualification-listing/qualification-consultants/) or [Support Request (requires Bluetooth.com account)](https://support.bluetooth.com/hc/en-us/requests) with the Bluetooth SIG. Contact technical support if you need more information related to Silicon Labs products.

> **Note**: There can also be newer DNs than the ones listed in the table above if there are newer software releases for Host, Controller, or X2Core layers. You can browse valid Qualified Designs and their Assessment Date by entering Silicon Laboratories in the search bar of [Product Search](https://qualification.bluetooth.com/MyProjects/ListingsSearch).
