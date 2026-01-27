# The Radio Scheduler

The Radio Scheduler is a component of RAIL (Radio Abstraction Interface Layer). RAIL provides an intuitive, easily-customizable radio interface layer and API, which supports proprietary or standards-based wireless protocols. The Radio Scheduler is designed to allow for radio operations that can be scheduled and prioritized. Different radio operations in each protocol may be more or less important, or more or less time sensitive, depending on the situation. The scheduler can take those into account when making decisions about conflicts and how to adjudicate them.

Unless you are developing applications with a custom protocol on RAIL, most radio scheduler functions are handled automatically by underlying stack and RAIL code. You only need to use the stack through its normal API.

At a high level, the stack sends a radio operation (for example a Scheduled Receive or Scheduled Transmit). The radio operations are queued and then serviced at a future time based upon their parameters. When it is time to start the radio operation the scheduler examines whether a competing event exists and whether the operation can be delayed. If the scheduler cannot run the event, it returns the result to the higher layer, which may retry with new parameters.

Once the radio operation has started, the corresponding stack can send the scheduler additional operations based on the results of the previous operation (for example waiting for an ACK). At the end of each operation or sequence of operations the stack must yield use of the radio.

## Radio Operations

Each event in the scheduler is broken up into elements called Radio Operations, which are associated with a radio config and a priority.

Every operation has a priority and is interrupted if the scheduler receives a higher priority operation that overlaps in time. Lower priority radio operations that cannot be run based on their schedule parameters will fail, and it is up to the respective stack to retry them. Once the scheduler actively runs a radio operation from the stack, the stack can continue to send additional radio operations until it voluntarily yields, or until the scheduler receives a higher priority radio operation and preempts it.

- Background Receive

- Scheduled Receive

- Scheduled Transmit

Each stack can ask the Radio Scheduler to perform up to two radio operations (background receive and either Scheduled Receive or Scheduled transmit) at a time.

Each operation has the following parameters:

| **Parameter** | **Description** |
|---------------|-----------------|
| Start Time | An indication at what point in the future this radio operation will run. This could be “run right now” or some value in microseconds in the future. |
| Priority | A number that indicates the relative priority of the operation. When using the default settings, Bluetooth LE radio operations are almost always higher priority than Zigbee operations. |
| Slip Time | An amount of time that the event can be delayed beyond its start time and still be acceptable to the stack. This may be 0, in which case the event cannot be slipped. |
| Transaction Time | The approximate amount of time that it takes to complete the transaction. Transmit events usually have a much more well-defined transaction time, while receive events are often unknown. This is used to help the radio scheduler determine whether an event can be run. |

The stack defines these various parameters appropriate to the operation being executed. For example, Bluetooth connection events are often scheduled in the future and have no allowed slip, whereas Zigbee transmit events can often be delayed a small amount and start later.

From the perspective of the RAIL Radio Scheduler, Scheduled transmit and Scheduled receive are identical. They are both simply operations that require use of the radio, and thus cannot be executed simultaneously. The difference is only apparent at RAIL API layer, where either a TX or RX API is called.

### Background Receive

This is a continuous receive mode that is intended to be interrupted by other operations and returned to after their completion. If Background Receive is higher priority than other operations, those radio operations will not be scheduled and will not run. It is up to the stacks or application to change the priority or voluntarily yield. See [Examples with Background Receive, Yield Radio and State Transition](./05-implementing-multiprotocol-with-rail#examples-with-background-receive-yield-radio-and-state-transition) for examples of how Background receive interacts with Scheduled operations.

### Scheduled Receive

This is a receive at a future time with a specified duration. The radio scheduler takes into consideration the radio switching time in deciding whether the operation will be scheduled. If it cannot be scheduled, then the scheduler sends a fail event to the calling stack. The radio operation is automatically extended until the stack voluntarily yields, or the scheduler receives a higher priority operation and interrupts it. Extending the receive allows the stack to continue a radio operation based on the requirements of the higher level protocol, for example transmission of a response based on the received data.

### Scheduled Transmit

This is a transmit at a future time with a minimum duration. This minimum duration can include expected follow-on events, for example an ACK to an IEEE 802.15.4 transmit. However, the minimum time for this operation does not have to include unexpected events that may extend the time beyond the minimum duration, for example backoffs due to CCA failures in IEEE 802.15.4. The radio scheduler takes into consideration the radio switching time in deciding whether the operation will be scheduled. If it cannot be scheduled, then the scheduler sends a fail event to the calling stack.

## Radio Config

Each radio operation is associated with a predefined radio config that determines the state of the hardware that must be used to perform the operation. The Radio Configs keep track of the stack's current state so that future radio operations will use the same radio parameters. Radio Configs may be active or dormant. If the stack changes an active Radio Config then RAIL makes an immediate change to the hardware configuration as well, for example changing a channel. If the radio config is not currently active then the next scheduled radio operation will use the new radio config.

## Priority

Each radio operation has a priority which indicates to the scheduler which operation should be executed if there is a timing overlap between multiple operations. The scheduler treats a priority of 0 as the highest priority and 255 as the lowest priority. The radio scheduler will allow the task with the highest priority to access the physical radio hardware. With most tasks control is returned to the radio scheduler only on completion, but tasks like background receive will be interrupted in case a task with higher priority becomes active.

The stacks each have a default set of priorities based on Silicon Labs’ analysis of how best to cooperate to maximize the duty cycle and avoid dropped connections for a generic use case. Specific use cases may have different needs. The priorities are as follows, from highest to lowest:

1. Bluetooth LE Scheduled Transmit

2. Bluetooth LE Scheduled Receive

3. Other protocol Scheduled Transmit

4. Other protocol Background Receive

These priorities may be overridden or changed by the application. It is up to the application to decide under what circumstances to change them. [802.15.4 Rail Priority](./04-implementing-multiprotocol-with-an-802-15-4-based-stack#802154-rail-priority) and [Bluetooth Priorities](./06-implementing-multiprotocol-with-bluetooth#bluetooth-priorities) contain more details on priorities for their specific instances.

## Slip Time

Every radio operation must have a "slip time," or maximum start time, meaning the furthest time in the future when the operation can be started if it cannot begin at the requested start time. This allows for the scheduler to work around higher priority events that are occurring at the same time, or higher priority events that extend beyond their expected duration. The protocol generally dictates what the slip time can be, but the radio scheduler is capable of handling this on a per-operation basis, allowing a stack to slip some events but not others. In general, IEEE 802.15.4 has longer slip time and Bluetooth LE has a minimal slip time.

## Yield

Once a sequence of radio operations is actively being run, the stack may continue to add operations extending the initial operation until the stack has nothing more to do for the particular message exchange. A stack must voluntarily yield unless it is performing a background receive. If a stack does not yield then it will continue to extend its radio operation, and lower priority radio operations will then trigger a failure back to the corresponding stack that requested that radio operation. A higher priority operation cannot interrupt a currently-running, lower priority radio operation that has not yielded. See [Examples with Background Receive, Yield Ration and State Transition](./05-implementing-multiprotocol-with-rail#examples-with-background-receive-yield-radio-and-state-transition) for examples of situations where explicitly yielding the radio is necessary.

## Interrupting a Radio Operation

A scheduled radio operation may be interrupted if a higher priority operation conflicts with it. This could occur in the following two circumstances:

   1. A scheduled radio operation takes longer than expected and the corresponding stack does not yield before the higher priority radio operation must start.

   2. A higher priority radio operation has just been scheduled to occur in the future and conflicts with a lower priority operation already scheduled.

## Long-Lived Radio Operations

Certain long-lived Radio Operations can have an outsized impact on the correct operation of the product. The application may need to coordinate these operations between the protocols. If the application does not, then the radio scheduler priorities take precedence. For example, an IEEE 802.15.4 energy scan can require that the radio stay on to gather sufficient energy readings. If the application does not properly coordinate the operations, the scan could be interrupted prematurely due to a higher priority Bluetooth operation.
