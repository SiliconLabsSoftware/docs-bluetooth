# Custom API Support

This chapter introduces how to implement a custom binary protocol between an NCP target and host using specific features of  the BGAPI. The Silicon Labs Bluetooth SDK  provides the following commands and events for that purpose:

- `cmd_user_message_to_target`

- `evt_user_message_to_host`

The command and event details are documented in the API reference manual.

The `cmd_user_message_to_target` command can be used by an NCP host to send a message to the target application on a device. To send a custom message with this API command, the host must send the byte sequence specified below to the target. Byte 4..255 can be the custom message itself.

The custom message can be interpreted in any specific way, but the default implementation uses the first byte as the custom command type, and the rest handled as parameters for that command. This is the recommended way of using custom APIs.

Command Byte Sequence:

:::custom-table{width=15%,15%,70%}
| Byte Number |Value/Type |Description |
|:-:|:-:|-|
| 0 |0x20 |Message type: Command |
| 1 |payload length |The size of the uint8array struct including its length and payload members.  |
| 2 |0xFF |Message class: User messaging |
| 3 |0x00 |Message ID |
| 4..255 |uint8array |The user message. The first byte is the length of the message. The next bytes are the message bytes  |
:::

Once the target receives this byte sequence, it must response with the byte sequence specified below. Byte 6 to 255 can be used for the custom response.

Response Byte Sequence:

:::custom-table{width=15%,15%,70%}
| Byte Number |Value/Type |Description |
|:-:|:-:|-|
| 0 |0x20 |Message type: Command |
| 1 |payload length |The size of the uint8array struct including its length and payload members.  |
| 2 |0xFF |Message class: User messaging |
| 3 |0x00 |Message ID |
| 4-5 |uint16 |Result code: 0: Success / Non-0: An error occurred |
| 6..255 |uint8array |The user message. The first byte is the length of the message. The next bytes are the response message bytes.  |
:::

Additionally, the `evt_user_message_to_host event` can be used by the target to send a message to NCP host. The target must send the byte sequence specified below. Byte 4..255 can be the custom message itself.

Event Byte Sequence:

:::custom-table{width=15%,15%,70%}
| Byte Number |Value/Type |Description |
|:-:|:-:|-|
| 0 |0xA0 |Message type: Event |
| 1 |payload length |The size of the uint8array struct including its length and payload members.  |
| 2 |0xFF |Message class: User messaging |
| 3 |0x00 |Message ID |
| 4..255 |uint8array |The user message. The first byte is the length of the message. The next bytes are the message bytes.  |
:::

## ncp_user_command_cb

The NCP Target calls `sl_ncp_user_cmd_message_to_target_cb` if a command ID equals to `sl_bt_cmd_user_message_to_target_id`.

You can find the default implementation of `sl_ncp_user_cmd_message_to_target_cb` in the *app.c* file and the declaration it in the *ncp.h* file.

In the first case, the Target echoes back the command to the Host, as a reply for the `USER_CMD_1` command. 

`USER_CMD_2` sends back the same command as an event, to demonstrate how events can be sent to the Host.

```C
sl_ncp_app_user_cmd_message_to_target_cb(void *data)
{
  uint8array *cmd = (uint8array *)data;

  switch (cmd->data[0]) {
    // -------------------------------
    // Example: user command 1.
    case USER_CMD_1_ID:
      //////////////////////////////////////////////
      // Add your user command handler code here! //
      //////////////////////////////////////////////

      // Send response to user command 1 to NCP host.
      // Example: sending back received command.
      sl_ncp_user_cmd_message_to_target_rsp(SL_STATUS_OK, cmd->len, cmd->data);
      break;

    // -------------------------------
    // Example: user command 2.
    case USER_CMD_2_ID:
      //////////////////////////////////////////////
      // Add your user command handler code here! //
      //////////////////////////////////////////////

      // Send response to user command 2 to NCP host.
      // Example: sending back received command.
      sl_ncp_user_cmd_message_to_target_rsp(SL_STATUS_OK, cmd->len, cmd->data);
      // Send user event too.
      // Example: sending back received command as an event.
      sl_ncp_user_evt_message_to_host(cmd->len, cmd->data);
      break;

    // -------------------------------
    // Unknown user command.
    default:
      // Send error response to NCP host.
      sl_ncp_user_cmd_message_to_target_rsp(SL_STATUS_FAIL, 0, NULL);
      break;
  }
}

```

## Host Side

The example below shows how to send custom APIs from the Host side, and how to handle custom events. The custom command structure is defined as a struct for easier access:

```C
#define USER_CMD_2_ID     0x02
#define DATA_LENGTH       0x02
PACKSTRUCT(struct user_cmd {
  uint8_t hdr;
  uint8_t data[DATA_LENGTH];
});
typedef struct user_cmd user_cmd_t;
```

The command packet is filled with Command ID and custom data, and then sent with the sl_bt_user_message_to_target. The response is logged to the console.

```C
user_command.hdr = USER_CMD_2_ID;
user_command.data[0] = 0xAA;
user_command.data[1] = 0xBB;
sl_bt_user_message_to_target(sizeof(user_command), (uint8_t*)&user_command, sizeof(user_command),
                               &resp_length, (uint8_t*)&custom_response);
app_log_info("Custom response received. Response data: %x %x", custom_response->data[0],
 custom_response->data[1]);
```

If the target also sends an event, it can be handled from the main event handler loop as any other NCP events. The `evt->data.evt_user_message_to_host.message.data` field contains the user message:

```C
case sl_bt_evt_user_message_to_host_id:
custom_event = (user_cmd_t*)evt->data.evt_user_message_to_host.message.data;
app_log_info("Custom event received. Response data: %x %x", custom_event->data[0], custom_event->data[1]);
break;
```
