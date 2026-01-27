# Radio Scheduler Examples

All examples use Bluetooth LE and Zigbee, but the principles apply to other Bluetooth/802.15.4 combinations.

The scheduler starts out by having a low priority Zigbee background receive operation. This represents an always-on router that may need to receive IEEE 802.15.4 packets at unknown times. A Bluetooth LE connection is also active and requires the stack to be ready to receive every 30 ms. The Bluetooth LE stack may schedule this well in advance due to the connection’s predictable nature.

## Priority Scheduling

This provides a basic example of adjudicating priorities of the different radio operations.

![Priority Scheduling](resources/sld485-image9.jpeg)

The Zigbee stack decides that it needs to send a packet. It may do this as an on-demand event, meaning the stack decides that it wants to send a packet *now* without informing the scheduler well in advance. This is in contrast to how Bluetooth LE operates, where the scheduled operations are known reasonably far in advance. The scheduler evaluates that it is possible to perform the Zigbee TX 1 radio operation and still service the higher priority Bluetooth LE reception event in the future. Therefore, the scheduler allows the transmit event to occur. The Zigbee stack performs all the pieces of this transmit operation (waiting for a MAC ack) and then voluntarily yields. The estimated transaction time of the Zigbee transmit radio operation does NOT include retries.

In this example, Bluetooth LE is *already* scheduled to receive in the future and the Zigbee stack wants to transmit. For the first Zigbee TX 1 radio operation there is enough time before the Bluetooth LE RX 1 radio operation, so the scheduler allows the stack to perform the operation. Later, when the Zigbee stack tries to schedule Zigbee TX 2 the scheduler determines there is not enough time before the high priority Bluetooth LE RX 2 event. However, the Zigbee stack has indicated that this action may slip its start time. The radio scheduler determines that given the expected duration of the Bluetooth LE radio operation the Zigbee operation can start after that event and still be within the slip time indicated by the Zigbee stack.

If all goes as expected, the Zigbee transmit operation will have its first attempt occur without any failures due to scheduling.

## Priority Interruption Example

This example illustrates a higher priority operation interrupting a lower priority one.

![Priority Interruption Example](resources/sld485-image10.jpeg)

This example starts in the same way as the previous example. Zigbee and Bluetooth LE both have a radio operation that is scheduled without any collision.

Later, the Zigbee stack decides it wants to send another packet for the Zigbee TX 2 event. The scheduler determines that it should be possible to schedule this event and service the Bluetooth LE RX 2 event later, based on the minimum time that the Zigbee TX 2 event must take. However, the Zigbee TX 2 event takes longer than expected due to a long random backoff and does not yield in time. This causes the event to collide with a higher priority radio operation, and so the Radio Scheduler interrupts the Zigbee event and returns a failure to the higher-level stack. The Bluetooth LE event occurs normally and when it is complete it voluntarily yields to any lower-priority operations.

Upon receiving the failure from the radio scheduler, the Zigbee stack immediately attempts to retry the MAC message. It schedules the operation and includes a slip time. At this point the Bluetooth LE stack has priority over the radio and thus the operation cannot be started yet, but the scheduler accepts the new radio operation. The Bluetooth LE stack completes its scheduled receive and yields the radio. The scheduler then triggers the Zigbee transmit operation to occur because it is still within the slip time of the initial start operation. After the transmit completes the scheduler returns to the background receive operation.

## Higher Priority Operation that is Extended

This example shows what happens when a higher priority operation takes longer than originally anticipated and causes a lower priority operation to miss its opportunity.

![Higher Priority Operation that is Extended](resources/sld485-image11.jpeg)

In this case, Bluetooth LE has a Scheduled receive that is currently taking place. Zigbee decides to send a packet but it cannot be run right now. The scheduler accepts the operation under the assumption that the Bluetooth LE event will complete before the end of the slip time of the Zigbee event. However, the Bluetooth LE event extends longer since additional packets are sent between the devices. The Bluetooth LE operation has priority, so the Zigbee operation eventually runs out of slip. An error is returned to the stack. Zigbee decides to re-transmit the packet. Again, the Zigbee stack indicates the operation should start now but may slip into the future. The scheduler is in the middle of changing the radio config so it cannot begin the operation immediately. Instead, it slips the radio operation start time a small amount and then executes the operation.

## Higher Priority Operation Without Interruption

In this example the radio scheduler is running on a node acting as a Bluetooth LE peripheral and that node has a number of connections to different central devices. It also has a periodic advertising beacon that is transmitted. The following figure shows a case where these events are occurring virtually back-to-back and do not allow for enough time to switch back to the Zigbee radio config. Therefore it will create a period where the Zigbee stack is unable to transmit even with the slip time.

![Higher Priority Operation Without Interruption](resources/sld485-image12.jpeg)

Zigbee asks the scheduler to schedule a transmit radio operation. Even though the scheduler knows that the event will fail due to scheduled higher priority operations, it still accepts the scheduled event. This is done for two reasons. First, circumstances may change and the event can be executed. Second, the stack sitting on top of the radio scheduler may try to retry the action. If the result of the failed scheduling was returned immediately then the stack's attempt to retry would be unlikely to succeed since no time has passed. Instead, by queuing the event and returning the failure *after* the slip time has expired, a retry (with its own slip time) has a better chance of success as the set of upcoming radio operations will be different.

## Receive When a Higher Priority Operation is Running

This example illustrates what happens when Bluetooth LE is active and a lower priority operation will be receiving data.

![Receive When a Higher Priority Operation is Running](resources/sld485-image13.jpeg)

In the first case, when an IEEE 802.15.4 message is sent and the Bluetooth LE stack is utilizing the radio for an active receive the Zigbee stack will not be online to receive the message. However, the Zigbee sender of the message will retry in most cases and with backoffs and other timing alterations is not going to conflict with another higher priority scheduled Bluetooth receive events unlikely to collide. The Zigbee message is received successfully.

The second case shows that, in the case of an active receive, the Zigbee stack may still be interrupted and not receive (or ACK) the message. Successful communication relies on retries at the MAC or higher layer to send this message again and verify the Dynamic Multiprotocol device receives the message.

While there may be considerations for whether or not active receive should be interrupted, it is difficult for the scheduler to make that determination. In general, the robustness of the protocols should allow for messages to be successfully received even with interruptions.
