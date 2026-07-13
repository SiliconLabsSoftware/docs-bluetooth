# Use Case #10: I want to exchange data with up to 32 peripheral devices

This use case describes an EFR32 central that scales to 32 simultaneous EFR32 peripheral links. It provides a characterization baseline for a Bluetooth Low Energy central that scans for peripherals advertising a known custom service, opens links as peers are found, discovers the required GATT objects, enables notifications, and maintains up to 32 simultaneous connections with selectable PHY operation. A general prerequisite for handling multiple connections is described [here](https://docs.silabs.com/bluetooth/latest/bluetooth-fundamentals-connections/multi-peripheral-topology).

The reference topology is one central plus up to 32 peripheral links. The software baseline is SiSDK 2025.12.3, and the hardware baseline is EFR32MG26 on BRD4116A with BRD4002A.

**Bluetooth features used**: connections, legacy passive scanning, GATT service discovery, notifications, optional write-without-response traffic, and selectable 1M, 2M, or Coded PHY operation.

## System Model and Assumptions

### Topology

- One EFR32 central maintains N simultaneous connections, where N can scale up to 32.
- The central uses a fixed-size peer table sized for the target connection count and treats each discovered peripheral as a managed slot in that table.
- All links are intended to remain active once opened. Application does not deliberately disconnect.
- Baseline runtime behavior is continuous multi-link management, GATT service discovery, notification reception from peripherals, and automatic reconnect handling after disconnects.
- Scanning restarts after connection events and during maintenance so discovery and reconnection can continue while other connections are active.

### Baseline Connection Configurations

- ATT MTU = 247.
- 1M PHY on all links.
- Peripheral latency = 0.
- Connection interval = 50 ms.
- No Bluetooth security used.
- Supervision timeout = 6000 ms.
- DLE enabled, preferred data length = 251.
- Even Connection Scheduling Algorithm enabled.
- Legacy passive scanning is used to discover peripherals that advertise the custom 128-bit service UUID in the advertisement payload.
- The central expects the peripheral GATT database to expose one custom service containing a notify characteristic for peripheral-to-central data and a write-without-response characteristic for optional central-to-peripheral traffic.
- Additional features used in the application are listed in [Appendix A. Application features](#appendix-a-application-features).


### General Configuration Tips

- Choose the right PHY profile: 1M, 2M, or Coded, based on throughput, airtime, and range requirements. More information on the trade-offs between PHYs can be found [here](https://docs.silabs.com/bluetooth/latest/bluetooth-fundamentals-connections/coded-phy).
- Size the connection budget for 8, 16, 20, 24, or 32 maintained links and set connection parameters and stack memory accordingly.
- Tune minimum and maximum connection event length to control scheduler margin.
- Align data length, ATT MTU, and available buffering with the intended traffic profile. Generic throughput optimization techniques are available [here](https://docs.silabs.com/bluetooth/latest/bluetooth-fundamentals-connections/multi-peripheral-topology).
- An appropriate Bluetooth security mechanism may be implemented using [Security Manager](https://docs.silabs.com/bluetooth/latest/bluetooth-stack-api/sl-bt-sm).
- For cases where 2.4 GHz congestion is expected, such as with coexistence, the following mitigations can be helpful:
    - Use longer connection intervals.
    - Reduce payload size.
    - Use [AFH](https://docs.silabs.com/bluetooth/latest/bluetooth-fundamentals-system-performance/afh).
    - Use [Channel Classification](https://docs.silabs.com/bluetooth/latest/bluetooth-fundamentals-system-performance/le-channel-classification).
    - Switch to a more robust PHY.

### Test Conditions

- Use peripherals that expose the required custom service and characteristic contract expected by the central.
- The test passes if all target links stay connected for the full test duration with no unexpected disconnects.
- Use good RF conditions, low external interference, fixed node placement, and identical peripheral firmware/configuration.

## Memory Footprint

### Component-wise Static Memory Breakdown

| Component / Section | Static RAM (B) | Flash (B) |
| --- | ---: | ---: |
| NVM | 0 | 40,960 |
| Platform drivers / HAL / emlib | 4,059 | 18,897 |
| Stack | 2,752 | 0 |
| Bluetooth stack - controller / link layer | 2,151 | 45,203 |
| RAIL / radio support | 1,660 | 51,748 |
| Application | 1,615 | 2,468 |
| Bluetooth stack - host / common | 746 | 53,476 |
| Toolchain / C runtime | 37 | 1,342 |
| Embedded Security | 225 | 18,046 |
| Platform services | 220 | 9,469 |
| Miscellaneous + linker/alignment overhead | 39 | 119 |
| **Total static RAM excluding heap** | **13,504** |  |
| Reserved heap | 248,640 | 0 |
| **Total Flash** |  | **241,728** |

> Note: The memory consumption under the **Application** heading in the table above includes the features listed in [Appendix A. Application features](#appendix-a-application-features) and may vary by user application. Component grouping in the table above follows [Appendix B](#appendix-b-object-file-grouping-used-for-the-memory-breakdown).

## Failure Modes and Mitigations

| Failure Mode | Typical Trigger | What to Watch | Likely Outcome | First Mitigation |
| --- | --- | --- | --- | --- |
| Event overlap | Short intervals, long events, many active links | Link bring-up stability, scheduler margin | Delayed service | Increase interval or reduce CE length |
| Missed events | High load, RF retries, or aggressive traffic settings | Disconnect count, supervision margin | Link instability | Lower duty cycle or payload size |
| Rising retransmissions | RF congestion or weak links | Retry rate, throughput drop | Reduced throughput | Improve RF conditions or use longer intervals |
| Timeout risk | Repeated missed service windows | Disconnect reasons, supervision timeout | Disconnects | Increase interval margin and reduce link load |
| Reconnect churn | Intermittent peripherals or marginal RF conditions | Reconnect attempts and dwell-time drops | Link flapping | Implement reconnection backoff |

## Decision Tree: When to Choose PAwR Instead of Many Connections

Use this quick check:

| If your design needs... | Prefer... |
| --- | --- |
| Per-node connection state, GATT discovery, notifications, writes, or low-latency two-way exchange for each device | Many connections |
| Mostly synchronized broadcast data, with optional scheduled responses from many devices | PAwR |
| More devices than the connection scheduler can comfortably maintain, or reconnect churn is the main bottleneck | PAwR, if the application can avoid always-on per-node links |

If both rows apply, start with many connections for up to 32 managed links and evaluate PAwR only if scheduler margin, reconnection behavior, or scaling becomes the limiting factor.

## Appendix A. Application features

- Fixed 32-slot peer table with connection statistics and per-peer state machine.
- 2M and Coded PHY are compile time configurable.
- Advertisement filtering for the expected custom 128-bit control service UUID.
- Automatic scan resume after connection and discovery events to fill or restore the 32-link set.
- Standard GATT flow: service, characteristic discovery, notification enablement.
- Compile-time configurable link parameters for PHY, interval, supervision timeout, MTU, and data length.
- Disconnect handling with bounded reconnect backoff and per-peer retry state.
- Optional logging of KPIs (disabled by default): opens, failures, disconnects, discovery failures, throughput, scheduler skips, reconnect attempts, full-32-link dwell time, and heap high-water reporting.
- Central can optionally enable a round-robin mechanism to perform write-without-response across all the connected peripherals. It is disabled in baseline.
- Service UUID: `efcdab89-6745-2301-fedc-ba9876543210`.
- TX notify UUID: `efcdab89-6745-2301-fedc-ba9876543211`.
- RX write UUID: `efcdab89-6745-2301-fedc-ba9876543212`.


## Appendix B. Object File Grouping Used for the Memory Breakdown

The map-file analysis groups the object files into 12 categories.

| Category | Summary |
| --- | --- |
| Reserved heap | Linker-generated heap reservation used by the memory manager |
| NVM | NVM3 storage reservation and related non-volatile memory support |
| Platform drivers / HAL / emlib | Device startup, low-level drivers, HAL, emlib, GPIO, system support, and NVM3 platform code |
| Stack | Linker-reserved stack region |
| Bluetooth stack - controller / link layer | Link layer scheduler, connection management, advertising, scanning, LL control procedures, packet handling, and controller support |
| RAIL / radio support | RAIL libraries, RF HAL, PHY support, PA support, calibration, timing, and radio sequencer support |
| Application | Application logic, main entry points, generated Bluetooth init, and board/event handlers |
| Bluetooth stack - host / common | GAP, GATT, ATT, L2CAP, BGAPI, bonding, scanner, and common host-side Bluetooth runtime |
| Toolchain / C runtime | C runtime startup, libc/newlib support, math support, and compiler runtime helpers |
| Embedded Security | PSA crypto, SE manager, radio AES support, key handling, storage, and crypto driver integration |
| Platform services | Clock manager, power manager, sleep timer, memory manager, MPU, interrupt manager, and device init support |
| Miscellaneous + linker/alignment overhead | Small uncategorized objects and linker/alignment overhead |

This grouping is used only to make the memory breakdown easier to read. Some categories contain many object files, while others contain only a few reserved sections or support objects.
