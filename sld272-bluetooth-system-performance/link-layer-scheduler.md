# Bluetooth Link Layer Scheduling

It is possible to choose between three different scheduling algorithms depending from the different use cases.

## Basic Scheduling

Basic scheduling is enabled by default, when the `Bluetooth Controller Anchor Selection` component is **not** installed.

Basic scheduling without using the anchor selection algorithm does not try to improve throughput or latency. It ensures that all tasks can share the radio, and it resolves scheduling conflicts if there is an overlap.  This is good for advertising / peripheral low throughput applications where latency is not an issue. It is also fit for PAwR synchronizers.

### Even Anchor Selection Algorithm

Enabled when the `Bluetooth Controller Anchor Selection` component is installed and `Anchor selection is done in an even manner` selected, this is the component default setting.

Even anchor selection algorithm is best for single central multi peripheral use case where all connections have the same intervals. It tries to give connections the most airtime without interfering with other link layer tasks.

### Empty Center Anchor Selection Algorithm

Enabled when the `Bluetooth Controller Anchor Selection` component is installed and `Anchors are placed in alternating manner` selected.

Empty center anchor selection algorithm is best for PAwR advertiser use case. This algorithm is best when the PAwR subevent and connections have the same interval. The algorithm relies on the runtime of the tasks, and it works best if there is enough time for all the tasks to execute within one interval.

### Configuration Parameters

- `Bluetooth Low Energy Controller` component has some configuration parameters related to Link Layer Scheduling under `Bluetooth Controller Configuration`.

- `Bluetooth Controller Minimum Connection Event Duration` is helpful to give some idea to the link layer’s scheduling algorithm to reserve the minimum runtime for each connection.

  For example if the system designer knows that they will be transmitting and receiving the 251 bytes for all connections and they are all using 1M PHY, then the minimum event length configuration should be ~4.3ms.
  
  This configuration acts as the bare minimum for the connection event length. If the host set a different minimum connection event length when creating the connection, then the link layer will use the bigger of `Bluetooth Controller Minimum Connection Event Duration` and minimum connection event length.

- `Bluetooth Controller Connection Event Extension` allows connections to overrun lower priority tasks as long as there is data to transmit or receive on the connection, and the maximum connection event length is not reached.

- `Bluetooth Controller Scanner Reception Early Abort` feature allows the controller to control the scanner to abort the reception of a packet if it will conflict with another scheduled higher priority task.

  This configuration is useful to ensure that reception of extended advertisements over secondary advertising channels won’t overrun the next scheduled task.
