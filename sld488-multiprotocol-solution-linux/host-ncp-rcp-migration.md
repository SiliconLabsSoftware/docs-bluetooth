# Zigbee Host+NCP to Host+Zigbeed+RCP Migration Note

This section describes one method of migrating a Zigbee Host+NCP system to a Host+Zigbeed+RCP system by backing up and restoring the token data. This method is similar to backing up and restoring Z3Gateway explained in [Backing Up and Restoring a Z3 Green Power Combo Gateway](https://docs.silabs.com/zigbee/latest/gp-combo-gateway-backup-restore/), with certain differences as noted here. One of the major differences is that in this case the same NCP hardware is used as RCP hardware.

The non-volatile Zigbee network stack context on an NCP is stored using the on-chip token system. By moving that stack context from the NCP to Zigbeed on the host, we can migrate a Zigbee Host+NCP application to Host+Zigbeed+RCP application.

The migration procedure requires the NCP using the NVM3 token system and the `zigbee\_token\_interface` component. If it is not already using these, then the NCP must first be updated to do so. Similarly, the Zigbee host app needs to be updated to add the `zigbee\_trust\_center\_backup` component with EMBER_AF_PLUGIN_TRUST_CENTER_BACKUP_POSIX_FILE_BACKUP_SUPPORT configuration set to 1.

With the above upgrades to NCP and host, the host can read the stack tokens from NCP and save them to a file.

![NCP host](./resources/figure-6-1-ncp-host.png)

The host then reads the saved tokens and updates the tokens on Zigbeed. Zigbeed’s default configuration includes the `zigbee\_token\_interface` component, which allows the host to write the saved network stack tokens to it.

![Zigbeed host](./resources/figure-6-2-host-zigbeed.png)

For simplicity, the `trust\_center\_backup` component provides command line interfaces to read and save ncp tokens and write to Zigbeed tokens. They are, respectively:

```C
plugin trust-center-backup backup-tokens <file name to save the tokens>

plugin trust-center-backup update-zigbeed <the file that has saved the tokens above>
```

There are certain limitations:

- This method only works for migrating from NCPs with NVM3 tokens.

- It migrates the stack tokens from NCP to Zigbeed. It presently does not migrate the custom tokens.

- This method assumes the same NCP hardware is used as RCP, and therefore retains the IEEE address (EUI64).

- Since the network key frame counter (NWK key FC) value is stored truncated and only incremented after initialization, you may need to perform an extra reset cycle of the Z3Gateway/Zigbeed to obtain a working value for the FC. Otherwise, the starting value after migrating may be lower than the last sent value and therefore the gateway’s packets might be ignored until the FC value exceeds the value before migration. Please note that, on initialization, the stack code will read the truncated FC and increment it by 0x1000. This incremented value will only be used at the next initialization (after a reset of the Z3Gateway).
