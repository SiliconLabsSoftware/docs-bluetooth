# OpenThread Sleepy End Device Demo

The **ot-ble-dmp** app starts out by adding an EM1 requirement during initialization, in the `sl_ot_rtos_application_init()` callback. This prevents the device from going into EM2 sleep mode, so that the CLI is responsive, and the user can enter commands.

To demonstrate an OpenThread Sleepy End Device, first form a two node OpenThread network by following the instructions at: [OpenThread CLI Example](https://github.com/openthread/openthread/tree/master/examples/apps/cli) .

Next, on the device that joined the network (not the leader), type the following commands:

```text
> mode s
> pollperiod 1000
```

The `mode` command puts the device into sleepy child mode. The `pollperiod` command tells the child to send data polls once every second. At this point the child is still not sleeping, and the CLI is still responsive.

Pressing either button PB0 or PB1 on the WSTK development board will toggle the energy mode requirement. Specifically, the first time the button is pressed, the EM1 requirement will be removed, allowing EM2 sleep. The child will start sleeping in EM2 mode in between data polls, and the CLI will no longer be responsive. You can verify that the child is still able to send and receive messages by sending a ping from the leader node. There will be up to one second of latency due to the child's sleep cycle. Pressing either button again will add back the EM1 requirement, which will bring the device out of EM2 so that the CLI can be used.

To monitor the power consumption of the device while performing the above steps, use the Energy Profiler tool in Simplicity Studio to connect to the device and start an energy capture. See *UG343: Multi-Node Energy Profiler User’s Guide* for more information about the Energy Profiler.
