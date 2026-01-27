# Important Software Components and Files

With the **ot-ble-dmp** project open in Project Configurator, click the Software Components tab to see all the software components in the Component Library. Filter on Installed Components to see the components used in the project. Four key component categories pertain to building a DMP application:

- Bluetooth components

- OpenThread components

- Rail Library Multiprotocol component (under Platform > Radio)

- FreeRTOS components (under Third Party)

You need to include these components in any project to add OpenThread and Bluetooth DMP functionality.

When the FreeRTOS component is added to a project, the Project Configurator automatically takes care of adding the CMSIS RTOS2-based adaptation layers necessary to run the OpenThread and Bluetooth stacks over FreeRTOS. The adaptation files for OpenThread and Bluetooth are located in the following Simplicity Studio 5 locations:

- developer/sdks/gecko_sdk_suite/\<version>/protocol/openthread/platform-abstraction/rtos/sl_ot_rtos_adaptation.c

- developer/sdks/gecko_sdk_suite/\<version>/protocol/bluetooth/src/sl_bt_rtos_adaptation.c

The three application source files for this project (the only source files that are not part of a Gecko Platform component) are stored at the top level of the project and are named:

- main.c

- app.c

- bluetooth_event_callback.c

## The Main Function and Initialization

The **ot-ble-dmp** app uses the same main function definition as used by other OpenThread sample applications. The call to `sl_system_init()`, which is defined in `sl_system_init.c`, initializes the entire system, including calls to `sl_bt_rtos_init()` and `sl_ot_rtos_init()` that are responsible for creating the Bluetooth and OpenThread threads.

The application can use the `app_init()` function to perform any necessary initialization steps prior to starting the kernel. The call to `sl_system_kernel_start()` starts the FreeRTOS scheduler.

The OpenThread instantiation and CLI initialization is handled by the OpenThread initialization thread by calling `sl_ot_init()` defined in `sl_ot_init.c`. More details on the different threads and their priorities are provided in the next section.

## FreeRTOS Tasks

The **ot-ble-dmp** app creates the following five RTOS threads by default:

- OpenThread initialization thread (priority 53)

- OpenThread main thread (priority 24)

- Bluetooth link layer thread (priority 52)

- Bluetooth stack event thread (priority 51)

- Bluetooth event handler thread (priority 50)

The OpenThread threads are created in sl_ot_rtos_adaptation.c, and the Bluetooth tasks are created in sl_bt_rtos_adaptation.c.

The OpenThread initialization thread handles the OpenThread instantiation and CLI initialization. As the Bluetooth event callback `sl_bt_on_event()` utilizes OpenThread CLI for prints (discussed in [Handling Bluetooth Events](#handling-bluetooth-events)), the OpenThread initialization thread starts with the highest priority.

Once the initialization is complete, the initialization thread also creates the main (operating) thread for OpenThread. By default, the main thread uses a low priority compared to Bluetooth, thus enabling Bluetooth threads to take over.

The priorities for the different Bluetooth threads and the OpenThread main thread are configurable and are defined in `sl_bt_rtos_config.h` and `sl_openthread_rtos_config.h`.

Silicon Labs Bluetooth has a serialized API which allows for commands and events to be passed between RTOS tasks in a thread-safe manner. OpenThread does not have a serialized API. For this reason, it is most convenient for the application logic to run in the OpenThread task. An application tick callback is provided for this purpose, and is called from within the OpenThread task's run loop: `sl_ot_rtos_application_tick()`. The **ot-ble-dmp** app includes a simple implementation of the tick callback in the app.c file.

OpenThread API calls made from within the application tick are thread-safe because they are executed within the OpenThread task. Because the Bluetooth API is serialized, Bluetooth API calls may be made from any task. The Bluetooth task is responsible for consuming and processing these serialized events. This happens transparently to the application.

## Handling Bluetooth Events

Bluetooth events are dispatched to the application via the `sl_bt_on_event()` callback. For the **ot-ble-dmp** app, an implementation of this callback is located in bluetooth_event_callback.c. This example handler simply prints out some information about the event. In a real application, these events would be processed by application handlers.

Bluetooth events are processed within a dedicated Bluetooth Event Handler thread. This is a separate thread whose sole purpose is to check for waiting Bluetooth events, and call `sl_bt_on_event()` when they become available. This thread is automatically created during initialization.

## Power Manager Integration

The **ot-ble-dmp** app also includes the Power Manager component (under Platform > Service > System), which is responsible for putting the system to sleep when possible.

The Power Manager component includes seamless FreeRTOS integration. It runs automatically from within the FreeRTOS Idle Task. When all threads have suspended (because they are pending on some event to continue processing), the Idle Task runs, and the power manager code can then put the system to sleep.

The application informs the power manager what sleep level it would like by adding and removing energy requirements via the API calls `sl_power_manager_add_em_requirement()` and `sl_power_manager_remove_em_requirement()`. Adding an EM1 requirement tells the Power Manager that the lowest energy level allowed is EM1, which only idles the processor and does not go to sleep. Removing the EM1 requirement allows the power manager to enter energy level EM2, which is deep sleep. See the reference for your MCU on [Silicon Labs Developer Documentation](https://docs.silabs.com/) under Modules\>Platform Services\>Power Manager.
