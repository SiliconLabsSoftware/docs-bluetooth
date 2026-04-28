# Bluetooth Link Layer Scheduling

Link Layer scheduling defines how Bluetooth radio tasks share airtime on the controller. This document describes the available scheduling algorithms, Link Layer task types, anchor placement concepts, and configuration parameters. The goal is to help developers choose the most appropriate scheduling mode for their application.

## Link Layer Tasks

A *task* is any Link Layer radio activity that must occur at scheduled intervals. The Bluetooth controller treats the following as tasks:

- One or more connection events
- Advertising
- Scanning
- Channel Sounding
- Periodic Advertising with Responses (PAwR) transmissions and receptions
- Multiple PAwR trains, where each train counts as a separate task

Each task has an interval, a required runtime, and an anchor point. The scheduler divides radio time between tasks while attempting to avoid overlaps.

## Anchor Concept

An *anchor point* is the reference time for a repeating Link Layer procedure, such as a connection event. This is a Bluetooth Core Specification term. For the formal definition, see the Core Spec.

Anchor placement matters for the following reasons:

- It affects throughput, latency, and coexistence between tasks.
- Efficient anchor distribution uses radio time without unnecessary conflicts.
- Misaligned anchors can reduce available airtime or increase the likelihood of a collision.

## Scheduling Algorithms

The Bluetooth Link Layer supports three selectable scheduling algorithms. Choose the one that best matches your device role, resource budget, and traffic requirements.

### Basic Scheduling

Basic Scheduling is enabled when the **Bluetooth Controller Anchor Selection** component is not installed.

Basic scheduling does not attempt to optimize latency or throughput. It ensures tasks share the radio and resolves conflicts only when overlaps occur.

Use Basic Scheduling for:

- Advertising or Peripheral low-throughput applications
- Applications where latency is not critical
- PAwR synchronizers
- Designs that require the lowest RAM, flash, and power usage

### Even Anchor Selection Algorithm

This algorithm is enabled when the **Bluetooth Controller Anchor Selection** component is installed and **Anchor selection is done in an even manner** is selected.

Use Even Anchor Selection for:

- A local device acting as **Central** that is connected to multiple peripherals
- Central devices that perform **Channel Sounding** as the initiator

Behavior:

- Tries to maximize airtime for connection events
- Distributes anchors evenly across the interval to balance use across connections

Tradeoffs:

- Uses more RAM, flash, and power than Basic Scheduling
- Not recommended for simple Peripheral-only devices
- Does not fully utilize maximum possible radio time

### Empty Center Anchor Selection Algorithm

This algorithm is enabled when the **Bluetooth Controller Anchor Selection** component is installed and **Anchors are placed in alternating manner** is selected.

Use Empty Center Anchor Selection for:

- Devices acting as **Central + PAwR Advertiser**
- Use cases where PAwR subevents and connections share the same interval

Behavior:

- Places anchors to reserve "center" airtime for PAwR operations.
- Works best when all task runtimes fit comfortably inside a single interval.

Multiple PAwR trains:

- Each train is treated as a separate task.
- Developers must ensure **system‑level timing** prevents PAwR response slot overlap.

## Task Runtime Considerations

Scheduling quality depends on the relationship between task runtimes and
their intervals:

- When total runtime is small compared to the interval, the scheduler can space anchors optimally.
- As total runtime approaches the interval:
  - Anchor placement becomes constrained.
  - Throughput and latency can degrade.
  - PAwR slot alignment becomes more difficult.
  - Lower-priority tasks can lose airtime.

A full PAwR use‑case example is available internally.

## Configuration Parameters

The **Bluetooth Low Energy Controller** component provides parameters that affect scheduling.

### Bluetooth Controller Minimum Connection Event Duration

Hints to the scheduler to reserve a minimum runtime for each connection.

Example: Transmitting and receiving 251-byte packets on the 1M PHY requires ~4.3 ms.

If the Host sets a larger minimum event length, the scheduler uses the larger value.

### Bluetooth Controller Connection Event Extension

Allows connection events to extend beyond their scheduled window if they still have data to exchange and have not exceeded the maximum connection event length. This can temporarily overrun lower‑priority tasks.

### Bluetooth Controller Scanner Reception Early Abort

Allows the scanner to abort packet reception if continuing would delay a higher-priority task. Use this setting to prevent extended advertisements from overrunning upcoming scheduled tasks.

## Choosing a Scheduling Algorithm

Choose **Basic Scheduling** when:

- The device has Peripheral-only roles.
- Throughput is low and latency is not critical.
- The device acts as a PAwR synchronizer.
- You need minimal RAM, flash, and power usage.

Choose **Even Anchor Selection** when:

- The device is a Central with multiple Peripherals.
- The Central performs Channel Sounding.
- You need balanced airtime allocation across connections.

Choose **Empty Center Anchor Selection** when:

- The device acts as Central + PAwR Advertiser.
- PAwR and connections share intervals.
- You can ensure that PAwR train timing does not overlap.

## Summary

Link Layer scheduling determines how radio airtime is shared between tasks. The right scheduler depends on device role, task mix, throughput and latency requirements, and resource constraints.

The anchor selection algorithms allow improved radio‑time distribution when needed, while Basic Scheduling provides the lowest resource footprint.
