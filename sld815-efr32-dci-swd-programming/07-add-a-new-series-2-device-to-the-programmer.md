# Add a New Series 2 Device to the Programmer

The programmer in this application note uses EFR32MG22C224F512IM40 as a host controller. The defines in the following table for EFR32MG22C224F512 should usually apply to the new Series 2 devices.

**Defines for EFR32MG22C224F512**

| **Item** | **Define** | **Value** |
|----------|------------|-----------|
| Main flash base address | FLASH_MEM_BASE in efr32mg22c224f512im40.h | 0x00000000 |
| User data base address | USERDATA_BASE in efr32mg22c224f512im40.h | 0x0FE00000 |
| MSC base address | MSC_BASE in efr32mg22c224f512im40.h | 0x40030000 |
| Offset and bitfields of MSC registers | MSC Register Map in xG22 reference manual | — |
| DEVINFO base address | DEVINFO_BASE in efr32mg22c224f512im40.h | 0x0FE08000 |
| Offset and bitfields of DEVINFO registers | DEVINFO Register Map in xG22 reference manual | — |
| CMU base address | CMU_BASE in efr32mg22c224f512im40.h | 0x40008000 |
| Offset of CMU_CLKEN1_SET register | CMU Register Map in xG22 reference manual | 0x00001068 |
| Bitmask to enable MSC clock | CMU_CLKEN1_MSC in efr32mg22_cmu.h | 0x00020000 |
| AP IDR (Cortex-M33) | S2_AHBAP_ID in app_dci_swd.h | 0x84770001 |
| TAR wrap mask (Cortex-M33) | TAR_WRAP_1K in app_swd_task.h | 0x3FF |

Users need to define new items if any defines in the table above do not match the new Series 2 device. And users may require tuning the settings in [Compile Options](06-series-2-dci-and-swd-programming-examples.md#compile-options) for the new Series 2 device.

The following procedures describe how to add the xG23 device to the programmer.

1. Add `XG23_FAMILY` (`0x17` for 23) to `app_swd_task.h.` It is `0x15` for xG21, `0x16` for xG22, `0x17` for xG23, etc.

    ```sh
    /// Device family of xG23
    #define XG23_FAMILY (0x00170000UL)
    ```

2. The  items  below  are  different  from  the  EFR32MG22C224F512  after  checking  the  xG23  header  files  (e.g., efr32fg23b010f512im48.h) and reference manual.

    - Main [flash base address](04-serial-wire-debug-swd-interface.md#flash-erase) — Add `FLASH_BASE_XG23` to `app_swd_task.h.`

    ```sh
    /// Flash start address of xG23
    #define FLASH_BASE_XG23 (0x08000000UL)
    ```

    - Bitmask to enable MSC clock — Add `CLKEN1_MSC_XG23` to `app_swd_task.h.`

    ```sh
    /// MSC bit of xG23 CMU_CLKEN1_SET
    #define CLKEN1_MSC_XG23 (0x00010000UL)
    ```

3. Add code to `app_swd_task.c` to enable [MSC clock](04-serial-wire-debug-swd-interface.md#serial-wire-debug-swd-interface) and set flash base address.

    ```sh
    // Enable MSC clock if device is xG23
    if (buf0 == XG23_FAMILY) {
    write_mem((uint32_t)&(CMU->CLKEN1_SET), CLKEN1_MSC_XG23);
    // Check UDLOCKBIT on xG23
    if (read_mem((uint32_t)&(MSC->MISCLOCKWORD)) & MSC_MISCLOCKWORD_UDLOCKBIT) {
    RAISE(SWD_ERROR_USERDATA_LOCK);
    }
    // Flash start address for xG23
    flash_start_addr = FLASH_BASE_XG23;
    }
    ```

4. Add `DEVICE_XG23` (ASCII code of character "3") to `app_process.h.` It is "1" for xG21, "2" for xG22, "3" for xG23, etc.

    ```sh
    /// xG23 device
    #define DEVICE_XG23 (0x33)
    ```

5. The [hse_svm_conf[]](./06-series-2-dci-and-swd-programming-examples.md#otp-settings) can apply to xG23A (HSE-SVM) devices. Add || `defined(DEVICE_XG23)` to `hse_svm_conf[]` in `app_dci_ta sk.c.`

    ```sh
    #if defined(DEVICE_XG21) || defined(DEVICE_XG23)
    /// HSE-SVM user configuration
    static const uint32_t hse_svm_conf[HSE_USER_CONF_SIZE] = {
    0x00000018, // 24 bytes data below
    SECURE_BOOT_ENABLE_MASK + ANTI_ROLLBACK_MASK, // MCU settings
    0x00000000, // 20 bytes reserved data
    0x00000000,
    0x00000000,
    0x00000000,
    0x00000000
    };
    #endif
    ```

6. The [hse_svh_xg21b_conf[]](./06-series-2-dci-and-swd-programming-examples.md#otp-settings) cannot apply to xG23B (HSE-SVH) devices. Add [hse_svh_other_conf[]](./06-series-2-dci-and-swd-programming-examples.md#otp-settings) to `app_dci_task.c.`

    ```sh
    #if defined(DEVICE_XG23)
    /// Other HSE-SVH user configuration
    static const uint32_t hse_svh_other_conf[HSE_USER_CONF_SIZE] = {
    0x00000018, // 24 bytes data below
    SECURE_BOOT_ENABLE_MASK + ANTI_ROLLBACK_MASK, // MCU settings
    0x40440410, // Tamper settings
    0x14040104,
    0x22414224,
    0x74422112,
    0x0500060A
    };
    #endif
    ```

7. Add code (based on `case DEVICE_XG21:`, replace `hse_svh_xg21b_conf` with `hse_svh_other_conf`) to `app_dci_task.c` to set up a command buffer for [OTP configuration](./05-se-command-list.md#initialize-otp).

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    // Check SVM or SVH
    if (*(cmd_buf + 2) == 'A') {
    *cmd_buf = INIT_HSE_OTP_LENGTH;
    *(++cmd_buf) = COMMAND_INIT_OTP;
    // Calculate parity of OTP configuration
    *(++cmd_buf) = 0;
    for (i = 1; i < HSE_USER_CONF_SIZE; i++) {
    *cmd_buf ^= hse_svm_conf[i];
    }
    // Copy OTP configuration to buffer
    memcpy((uint32_t *)(++cmd_buf), (uint32_t *)hse_svm_conf,
    INIT_HSE_OTP_LENGTH);
    } else if (*(cmd_buf + 2) == 'B') {
    *cmd_buf = INIT_HSE_OTP_LENGTH;
    *(++cmd_buf) = COMMAND_INIT_OTP;
    // Calculate parity of OTP configuration
    *(++cmd_buf) = 0;
    for (i = 1; i < HSE_USER_CONF_SIZE; i++) {
    *cmd_buf ^= hse_svh_other_conf[i];
    }
    // Copy OTP configuration to buffer
    memcpy((uint32_t *)(++cmd_buf), (uint32_t *)hse_svh_other_conf,
    INIT_HSE_OTP_LENGTH);
    } else {
    RAISE(SWD_ERROR_UNKNOWN_DEVICE);
    }
    break;
    #endif
    ```

8. Add [firmware images](./06-series-2-dci-and-swd-programming-examples.md#firmware-images) to `app_firmware_image.c` and `app_firmware_image.h.`

    ![Add firmware image](./resources/sld815-image21.png)

9. Add functions to `app_firmware_image.c` and `app_firmware_image.h` to get the address and size of firmware images.

    ![Add functions](./resources/sld815-image22.png)

10. The `tamper_source_xg21b[]` cannot be applied to xG23B (HSE-SVH) devices. Add `tamper_source_other[]` to `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    /// Strings for tamper sources of other HSE-SVH devices
    static const char *tamper_source_other[TAMPER_SIGNAL_NUM] = {
    NULL,
    "Filter counter : ",
    "SE watchdog : ",
    NULL,
    "SE RAM ECC 2 : ",
    "SE hard fault : ",
    NULL,
    "SE software assertion : ",
    "SE secure boot : ",
    "User secure boot : ",
    "Mailbox authorization : ",
    "DCI authorization : ",
    "OTP Read : ",
    NULL,
    "Self test : ",
    "TRNG monitor : ",
    "Secure lock : ",
    "Digital glitch : ",
    "Voltage glitch : ",
    "SE ICACHE : ",
    "SE RAM ECC 1 : ",
    "BOD : ",
    "Temperature sensor : ",
    "DPLL lock fail low : ",
    "DPLL lock fail high : ",
    "PRS0 : ",
    "PRS1 : ",
    "PRS2 : ",
    "PRS3 : ",
    "PRS4 : ",
    "PRS5 : ",
    "PRS6 : "
    };
    #endif
    ```

11. Add  || `defined(DEVICE_XG23)` and  code  (based  on  `case DEVICE_XG21:`,  replace  `tamper_source_xg21b` with `tamper_source_other`) to function `print_otp_conf()` in `app_process.c` to print out the [tamper configuration](./05-se-command-list.md#anti-tamper-configuration).

    ```sh
    #if defined(DEVICE_XG21) || defined(DEVICE_XG23)
    uint32_t i;
    uint32_t j;
    uint32_t k;
    #endif
    ...
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    for (i = 0, j = 0, k = 2; i < TAMPER_SIGNAL_NUM; i++, j += 4) {
    if (j == 32) {
    j = 0;
    k++;
    }
    cmd_resp_buf[8] = (cmd_resp_buf[k] >> j) & 0x0f;
    if (tamper_source_other[i] != NULL) {
    printf(" %s %lu\n", tamper_source_other[i], cmd_resp_buf[8]);
    }
    }
    break;
    #endif
    ...
    #if defined(DEVICE_XG21) || defined(DEVICE_XG23)
    // Common tamper parameters
    printf(" + Reset period for the tamper filter counter: ~32 ms x %u\n",
    1 << (cmd_resp_buf[6] & COUNTER_PERIOD_MASK));
    printf(" + Activation threshold for the tamper filter: %d\n",
    256 / (1 << ((cmd_resp_buf[6] & COUNTER_THRESHOLD_MASK) >> COUNTER_THRESHOLD_SHIFT)));
    if (cmd_resp_buf[6] & GLITCH_DETECTOR_MASK) {
    printf(" + Digital glitch detector always on: Enabled\n");
    } else {
    printf(" + Digital glitch detector always on: Disabled\n");
    }
    if (device_name[DEVICE_INDEX] > DEVICE_XG22) {
    if (cmd_resp_buf[6] & SLEEP_ALIVE_MASK) {
    printf(" + Keep tamper alive during sleep: Enabled\n");
    } else {
    printf(" + Keep tamper alive during sleep: Disabled\n");
    }
    }
    printf(" + Tamper reset threshold: %lu\n", cmd_resp_buf[6] >> TAMPER_RESET_SHIFT);
    #endif
    ```

12. Add code (take `case DEVICE_XG21:` as reference) to function `prog_main_flash_app()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf(" + The xG23 application firmware image size is %lu bytes "
    "and start address is 0x%08lX.\n", get_xg23_app_size(),
    get_flash_start_addr());
    printf(" + Erase-Program-Verify the xG23 main flash for "
    "application firmware image... ");
    cmd_resp_buf[0] = prog_flash(get_flash_start_addr(),
    get_xg23_app_size(),
    (uint32_t *)get_xg23_app_addr());
    print_cycle_time();
    break;
    #endif
    ```

13. Add code (take `case DEVICE_XG21:` as reference) to function `prog_main_flash_se()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf(" + The xG23 HSE firmware image version: %08lX\n",
    *((uint32_t *)get_xg23_hse_addr() + 3));
    printf(" + The xG23 HSE firmware image size is %lu bytes and start "
    "address is 0x%08lX.\n", get_xg23_hse_size(),
    SE_START_ADDR + get_flash_start_addr());
    printf(" + Erase-Program-Verify the xG23 main flash for HSE "
    "firmware image... ");
    cmd_resp_buf[0] = prog_flash(SE_START_ADDR + get_flash_start_addr(),
    get_xg23_hse_size(),
    (uint32_t *)get_xg23_hse_addr());
    print_cycle_time();
    break;
    #endif
    ```

14. Add code (take `case DEVICE_XG21:` as reference) to function `prog_main_flash_se_app()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf(" + The xG23 HSE upgrade application firmware image size is "
    "%lu bytes and start address is 0x%08lX.\n",
    get_xg23_hse_upgrade_app_size(), get_flash_start_addr());
    printf(" + Erase-Program-Verify the xG23 main flash for "
    "application to upgrade \n");
    printf(" HSE firmware... ");
    cmd_resp_buf[0] = prog_flash(get_flash_start_addr(),
    get_xg23_hse_upgrade_app_size(),
    (uint32_t *)get_xg23_hse_upgrade_app_addr());
    print_cycle_time();
    break;
    #endif
    ```

15. Add code (take `case DEVICE_XG21:` as reference) to function `prog_main_flash_signed()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf(" + The xG23 signed firmware image size is %lu bytes "
    "and start address is 0x%08lX.\n",
    get_xg23_signed_size(),
    get_flash_start_addr());
    printf(" + Erase-Program-Verify the xG23 main flash for "
    "signed firmware image... ");
    cmd_resp_buf[0] = prog_flash(get_flash_start_addr(),
    get_xg23_signed_size(),
    (uint32_t *)get_xg23_signed_addr());
    print_cycle_time();
    break;
    #endif
    ```

16. For xG23 devices, the user data can be erased the same way as any page in the main flash. Add code (take `case DEVICE_XG22:` as reference) to function `erase_user_data()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf("\n . Erase xG23 user data... ");
    cmd_resp_buf[0] = erase_flash(true);
    print_cycle_time();
    hard_reset_target();
    break;
    #endif
    ```

17. For xG23 devices, the user data can be written the same way as any page in the main flash. Add code (take `case DEVICE_XG22:` as reference) to function `prog_user_data()` in `app_process.c.`

    ```sh
    #if defined(DEVICE_XG23)
    case DEVICE_XG23:
    printf("\n . Program xG23 user data.\n");
    printf(" + User data size is %lu bytes and start address is 0x%08lX.\n",
    get_userdata_size(), USERDATA_BASE);
    printf(" + Erase-Program-Verify the xG23 user data... ");
    cmd_resp_buf[0] = prog_flash(USERDATA_BASE,
    get_userdata_size(),
    (uint32_t *)get_userdata_addr());
    print_cycle_time();
    hard_reset_target();
    break;
    #endif
    ```

18. There is no need to change the default settings in [Compile Options](./06-series-2-dci-and-swd-programming-examples.md#compile-options) for xG23 devices.

    >**Note**: The 512 kB flash of EFR32MG22C224F512 is not enough to store the [firmware images](./06-series-2-dci-and-swd-programming-examples.md#firmware-images) when the programmer needs to support more Series 2 devices. Users can selectively comment out the defines in `app_process.h` to save memory for the required Series 2 devices.

    ``` sh
    /// xG21 device
    #define DEVICE_XG21 (0x31)

    /// xG22 device
    define DEVICE_XG22 (0x32)

    /// xG23 device
    #define DEVICE_XG23 (0x33)

    ```
