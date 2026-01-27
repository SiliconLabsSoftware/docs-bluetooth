# Configuring AXiP/EXiP (Series 3 only)

The Authenticated eXecute in-Place feature is available for Series 3 devices with QSPI flash. This feature is provided to resist attempts to

extract the contents of the executable code in the QSPI flash and modify it. Series 3 can configure up to 8 code regions in flash with

the protection level set separately for each region. By default, SixG301 devices have 2 code regions as follows:

| Flash size | Region 0 | Region 1(2) | Protection |
|------------|----------|-------------|------------|
| 2 MB | 32 kB | 864 kB | AXiP |
| 3 MB | 32 kB | 1408 kB | AXiP |
| 4 MB | 32 kB | 1984 kB | AXiP |
| External | 32 kB | 1. | AXiP |

**Notes**:

1. Region 1 size = (size of flash – 192 kB (SE Firmware) – 32 kB)/2

2. Sizes shown in this column are logical, not physical, sizes

To verify the AXiP configuration for your device, run the following command:

```sh
commander security readregionconfig --device SixG301 --outfile region-config.yaml
```

```sh
Writing parsed configuration to file region-config.yaml...
DONE
```

To view the configuration, open `region-config.yaml` in a text editor. The default configuration looks like this:

```sh
regions:
  - size_kb: 32
    protection: encrypted_authenticated
  - size_kb: 1408
    protection: encrypted_authenticated
```

This is the recommended configuration for most applications. For instructions on modifying the default configuration, refer to AN1509.

To write a new region configuration to the device, run the following command:

```sh
commander security writeregionconfig region-config.yaml -d simg301
```

```sh
Reading configuration from file region-config.yaml...
Writing region configuration to device...
DONE
```
