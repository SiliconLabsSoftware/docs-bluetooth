# Implementing Multiprotocol with RAIL

This section offers more information about RAIL for users who consume the RAIL API directly to develop proprietary protocols. In particular, it offers details on how to work with the RAIL APIs to handle specific radio scheduler cases.

## Examples with Background Receive, Yield Radio and State Transition

The fundamentals of the RAIL Multiprotocol priority system are straightforward: a radio event with a higher priority (that is, smaller in number) will always usurp any other radio events with lower priority. However, this topic becomes more complicated when considering state transitions and APIs such as `RAIL_StartRx()`, which put the radio into a certain state for an indefinite amount of time. This section provides some illustrations and examples to demonstrate how these time-unbounded states are handled, and how the application layer can use APIs such as `RAIL_YieldRadio()` to control them. The examples are as follows:

- [State Transitions with a Single Protocol](#state-transitions-with-a-single-protocol)

- [State Transitions with Two Protocols](#state-transitions-with-two-protocols)

- [State Transitions with Two Protocols and Monotonically Increasing Priorities](#state-transitions-with-two-protocols-and-monotonically-increasing-priorities)

In these examples, `RAIL_StartTx()` is the source of the TX event that interrupts the background RX. Note, however, that these examples are applicable to any radio API except for `RAIL_StartRx()`. In other words, the examples are applicable to any API that starts a radio event that is not a background RX.

These examples illustrate expected multiprotocol behaviors regarding state transitions. To summarize:

- In a state transition, the new state is treated as an indefinite extension of the originating event at that same priority until `RAIL_YieldRadio()` is called.

- Background RX events are not affected by `RAIL_YieldRadio()`. Only `RAIL_Idle()` can permanently remove a protocol from the background RX state.

- An event with a higher priority will always usurp an event with lower priority, regardless of any other API calls.

- Only `RAIL_StartRx()` receives can be ‘returned to’ from a higher priority event through `RAIL_YieldRadio()` or `RAIL_Idle()`.

- All radio events other than `RAIL_StartRx()` require `RAIL_YieldRadio()` in order to end and progress to the next event.

- The call to `RAIL_YieldRadio()` cannot be replaced with `RAIL_Idle()`. `RAIL_Idle()` clears out *all* events for the given protocol.

### State Transitions with a Single Protocol

This first example examines the behavior of the radio with a single protocol (that is, where the same `RAIL_Handle_t` is used for all radio function calls). The radio starts in RX with an initial call to `RAIL_StartRx()`, then moves into a TX with a higher priority call to `RAIL_StartTx()`. It is important to note that after the transmit is done, the radio transitions to the state specified by `RAIL_SetTxTransitions()`, and it stays in the state indefinitely at the same priority and channel as the TX until `RAIL_YieldRadio()` is called. After that, the radio returns to RX, with the initially specified priority and channel.

![State Transitions with Calls to RAIL_StartTx(), RAIL_StartRx(), RAIL_YieldRadio() with a Single Protocol](resources/sld485-image14.png)

The need to actively yield the radio, and thus the `RAIL_YieldRadio()` API were necessary largely due to ACK’ing. The design philosophy is that, because both a TX and a received ACK are viewed as part of the same transaction, if a node transmits and expects an ACK it should be able to both transition to RX and continue listening for the ACK as part of the same operation (and therefore same priority) as the original TX. In general, however, RAIL on its own cannot know whether or not an ACK is required. This can depend on other factors, such as packet contents, or other application logic, and so cannot be simply determined by checking whether ACK’ing has been configured with `RAIL_ConfigAutoAck()`.Therefore, discretion as to when a radio transaction is complete is left to the application/stack.

In the case that an ACK is not required, Silicon Labs recommends calling `RAIL_YieldRadio()` as part of handling the `RAIL_EVENT_TX_PACKET_SENT` event. Doing this causes the green line in the above figure to be minimized down to the interrupt latency time. If the application does expect an ACK, `RAIL_YieldRadio()` should be called when the ACK is received or has been deemed to time out.

### State Transitions with Two Protocols

This scenario is similar to the first scenario regarding state transitions after TX, but introduces another protocol.

![State Transitions with Calls to RAIL_StartTx(), RAIL_StartRx(), RAIL_YieldRadio() With Two Protocols](resources/sld485-image15.jpeg)

In this situation, it is important to note that `RAIL_StartRx()` can be called at any time during the TX transaction. As long as its priority is less than or equal to the priority of the TX, the RX will not come into effect until the application calls `RAIL_YieldRadio()` on Protocol A. When `RAIL_StartRx()` is called during the TX, the RX is merely added to the queue of events to be handled.

Another key point is that, although `RAIL_YieldRadio()` on Protocol A will transition from TX on Protocol A to RX on Protocol B, a `RAIL_Idle()` on Protocol B is required to transition from the RX on Protocol B to the RX on Protocol A. The philosophy here is that Background RXs can’t really be yielded, since the event is never really over. The only way to exit is to stop the Background RX with a call to `RAIL_Idle()`.

### State Transitions with Two Protocols and Monotonically Increasing Priorities

The final scenario is nearly identical to the previous one, except the call to `RAIL_StartRx()` on Protocol B is at a higher priority than the call to `RAIL_StartTx()` on Protocol A.

![Example of State Transitions with Calls to RAIL_StartTx(), RAIL_StartRx(), RAIL_YieldRadio() with Two Protocols and Different Priorities](resources/sld485-image16.png)

In this case, since the priority of the second `RAIL_StartRx()` is higher than the priority of the call to `RAIL_StartTx()`, a call to `RAIL_YieldRadio()` is no longer necessary. Because the second `RAIL_StartRx()` is at a higher priority, it usurps the `RAIL_StartTx()` event, taking control of the radio and removing the TX event from the state. At any time during that RX on Protocol B, `RAIL_Idle()` can be called to return to the RX on Protocol A, just as in the previous example.

Note here, that when the application calls `RAIL_Idle()` on Protocol B’s RX, the application does not return to the TX Transition of Protocol A. Instead, it goes right to the background RX, even though the application never called `RAIL_Idle()` on Protocol A’s TX. For Scheduled radio operations (that is, any radio operation started by an API other than `RAIL_StartRx()`), once a radio event is usurped by a higher priority event, it is removed entirely and will not be returned to later. Only Background receives, started by `RAIL_StartRx()`, can be maintained in the background and ‘returned to’ through a call to `RAIL_YieldRadio()` or `RAIL_Idle()`.

To emphasize the difference between `RAIL_YieldRadio()` and `RAIL_Idle()` it is important to note that, for all these examples, the call to `RAIL_YieldRadio()` cannot be replaced with `RAIL_Idle()`. `RAIL_Idle()` clears out *all* events for the given protocol – both the Background (that is, started by `RAIL_StartRx()`) and Scheduled (that is, started by APIs other than `RAIL_StartRx()`) operations. `RAIL_Idle()` would indeed still cause the application to exit out of the TX transition state, but it would also clear out the Background RX, causing the application to return to idle, not RX.
