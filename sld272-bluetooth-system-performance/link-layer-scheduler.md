# Bluetooth Link Layer Scheduling

Link Layer scheduling defines how Bluetooth radio tasks share airtime on
the controller.\
This document describes the available scheduling algorithms, Link Layer
task types,\
anchor placement concepts, and configuration parameters. The goal is to
help developers\
choose the most appropriate scheduling mode for their application.

------------------------------------------------------------------------

## Link Layer Tasks

A *task* is any Link Layer radio activity that must occur at scheduled
intervals.\
The Bluetooth controller treats the following as tasks:

-   One or more **connection events**
-   **Advertising**
-   **Scanning**
-   **Channel Sounding**
-   **Periodic Advertising with Responses (PAwR)** transmissions and
    receptions
-   **Multiple PAwR trains**, each considered a separate task

Tasks have intervals, required runtime, and anchor points. The scheduler
divides radio\
time between tasks while attempting to avoid overlaps.

------------------------------------------------------------------------

## Anchor Concept

An *anchor point* is the reference time for a repeating Link Layer
procedure\
(such as a connection event).\
This is a Bluetooth Core Specification term; refer to the Core Spec for
the formal definition.

Why anchors matter:

-   Anchor placement affects **throughput**, **latency**, and
    coexistence between tasks.
-   Efficient anchor distribution ensures radio time is used without
    unnecessary conflicts.
-   Misaligned anchors may reduce available airtime or increase
    collision likelihood.

------------------------------------------------------------------------

## Scheduling Algorithms

The Bluetooth Link Layer supports three selectable scheduling
algorithms.\
Developers may choose the most suitable one based on device role,
resource needs,\
and traffic requirements.

------------------------------------------------------------------------

### Basic Scheduling

Enabled when the *Bluetooth Controller Anchor Selection* component is
**not installed**.

Basic scheduling does not attempt to optimize latency or throughput.\
It ensures tasks share the radio and resolves conflicts only when
overlaps occur.

Best for: - Advertising / **Peripheral low‑throughput** applications
- Applications where **latency is not critical**
- **PAwR synchronizers**
- Designs requiring **lowest RAM, flash, and power usage**

------------------------------------------------------------------------

### Even Anchor Selection Algorithm

Enabled when the *Bluetooth Controller Anchor Selection* component is
installed and\
**Anchor selection is done in an even manner** is selected.

Best for: - Local device acting as **Central** connected to **multiple
Peripherals**
- Central devices performing **Channel Sounding** where the Central is
the initiator

Behavior: - Tries to **maximize airtime for connection events**
- Distributes anchors evenly across the interval for balanced use across
connections

Tradeoffs: - Uses **more RAM, flash, and power** than Basic Scheduling
- Not recommended for **simple Peripheral-only devices**
- Does **not fully utilize maximum possible radio time**

------------------------------------------------------------------------

### Empty Center Anchor Selection Algorithm

Enabled when *Bluetooth Controller Anchor Selection* is installed and\
**Anchors are placed in alternating manner** is selected.

Best for: - Devices acting as **Central + PAwR Advertiser**
- Use cases where **PAwR subevents** and **connections** share the same
interval

Behavior: - Places anchors to reserve "center" airtime for PAwR
operations
- Works best when all task runtimes fit comfortably inside a single
interval

Multiple PAwR trains: - Each train is treated as a separate task
- Developers must ensure **system‑level timing** prevents PAwR response
slot overlap

------------------------------------------------------------------------

## Task Runtime Considerations

Scheduling quality depends on the relationship between task runtimes and
their intervals:

-   When total runtime is small compared to the interval, anchors can be
    spaced optimally
-   As total runtime approaches the interval:
    -   Anchor placement becomes constrained
    -   Throughput and latency may degrade
    -   PAwR slot alignment becomes more difficult
    -   Lower‑priority tasks may lose airtime

A full PAwR use‑case example is available internally.

------------------------------------------------------------------------

## Configuration Parameters

The *Bluetooth Low Energy Controller* component provides parameters
affecting scheduling.

### Bluetooth Controller Minimum Connection Event Duration

Hints the scheduler to reserve a minimum runtime for each connection.

Example:\
Transmitting + receiving 251‑byte packets on 1M PHY requires \~4.3 ms.

If the Host sets a larger minimum event length, the scheduler uses the
larger value.

------------------------------------------------------------------------

### Bluetooth Controller Connection Event Extension

Allows connection events to extend beyond their scheduled window if they
still\
have data to exchange and have not exceeded the maximum connection event
length.\
This may temporarily overrun lower‑priority tasks.

------------------------------------------------------------------------

### Bluetooth Controller Scanner Reception Early Abort

Allows the scanner to abort packet reception if continuing would delay a
higher‑priority task.\
Useful for ensuring extended advertisements do not overrun upcoming
scheduled tasks.

------------------------------------------------------------------------

## Choosing a Scheduling Algorithm

Choose **Basic Scheduling** when: - Peripheral-only roles\
- Low throughput and non‑critical latency
- PAwR synchronizer use cases
- Minimal RAM/flash/power usage required

Choose **Even Anchor Selection** when: - Central device with multiple
Peripherals
- Central performing Channel Sounding
- Balanced airtime allocation across connections is needed

Choose **Empty Center Anchor Selection** when: - Central + PAwR
Advertiser roles
- PAwR and connections share intervals\
- Developer can ensure PAwR train timing does not overlap

------------------------------------------------------------------------

## Summary

Link Layer scheduling determines how radio airtime is shared between
tasks.\
Choosing the correct scheduler depends on device role, task mix,
throughput and\
latency requirements, and resource constraints.

The anchor selection algorithms allow improved radio‑time distribution
when needed,\
while Basic Scheduling provides the lowest resource footprint.
