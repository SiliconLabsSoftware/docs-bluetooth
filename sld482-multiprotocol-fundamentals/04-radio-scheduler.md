# Radio Scheduler

A radio scheduler is an essential component of a dynamic multiprotocol implementation. It is a system for on-demand scheduling of radio tasks as requested by wireless stacks and the manufacturer’s applications. This page introduces basic radio scheduler concepts. For details, including examples, of the Silicon Labs radio scheduler operation, see the [Dynamic Multiprotocol User’s Guide](https://docs.silabs.com/connect/latest/multiprotocol-dynamic-ug/).

The Silicon Labs Radio Scheduler is part of the RAIL library. It operates above the radio hardware and below the RAIL API, as shown in the figure:

![Silicon Labs Radio Scheduler](resources/sld482-image8.png)

Different radio events in each protocol may be more or less important, or more or less time sensitive, depending on the situation. The Radio Scheduler can take those into account when making decisions about conflicts and how to adjudicate them.

Micrium OS is an RTOS that allows stacks and application logic to share CPU execution time.

The Radio Scheduler uses the following concepts.

**Radio Operation**: A radio operation is a specific action to be scheduled that has both a radio configuration and a priority. Each stack can request that the radio scheduler perform three radio operations:

- Background receive: Continuous receive, intended to be interrupted by other radio operations

- Scheduled receive: Receive at a future time with a minimum duration

- Scheduled transmit: Transmit at a future time with a minimum duration

**Radio Config**: Radio config determines the state of the hardware that must be used to perform a radio operation.

**Priority**: Each operation from each stack has a default priority. An application can change default priorities.

**Slip Time**: Slip time is the maximum time in the future when the operation can be started if it cannot begin at the requested start time.

**Yield**: A stack must voluntarily yield at the end of an operation or sequence of operations, unless it is performing a background receive. Until the stack yields, the scheduler will not schedule lower priority tasks.

The Radio Scheduler can interrupt a scheduled radio operation if a higher priority task conflicts with it. This could occur in the following two circumstances:

   1. A scheduled radio operation takes longer than expected, and the corresponding stack does not yield before the higher priority radio operation must start.

   2. A higher priority radio operation has just been scheduled to occur in the future and conflicts with a lower priority operation already scheduled.

Certain long-lived radio operations can have an outsized impact on the correct operation of the product. The application may need to coordinate these tasks between the protocols. If the application does not, then the radio scheduler priorities will take precedence. This can result in the task being interrupted prematurely.
