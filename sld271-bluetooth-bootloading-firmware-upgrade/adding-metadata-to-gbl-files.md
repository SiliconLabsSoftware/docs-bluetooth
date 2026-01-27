
# Adding Metadata to GBL Files

## Introduction

### Metadata

Metadata can be anything that can be put into a byte array. You can add text, structures, or binary data to your GBL files. Hence, when you update your device with a new firmware, you can also update other settings, e.g., a parameter set, that is stored in the persistent storage. You can also send extended version information along with the firmware or you can send new security keys in a signed and encrypted GBL file. You can even send an extra firmware image as metadata intended to be sent to another device.

### Adding Metadata to the GBL File

Metadata can be added when you create the GBL file with the `commander gbl create` command. Add the --metadata switch and the name of the file, that contains the metadata. For example:

`commander gbl create application.gbl --app application.s37 --metadata metadata.bin`

If you use the create_bl_files.bat/sh/py file to create GBL files, edit the file and extend the commands with the --metadata switch.

### Reading Metadata from the Uploaded Files

You can read the content of the metadata that was put into the GBL file as follows:

- You can read data on-the-fly while uploading the image.
- You can read data from the fully uploaded GBL file after upload has finished.

**Note:** Neither version is supported by the Bluetooth stack AppLoader because AppLoader decodes the GBL file on-the-fly, does not store the full GBL file anywhere, and does not allow accessing the metadata. To access metadata, implement an application level OTA DFU, as described in this [code example](https://github.com/SiliconLabs/gecko_sdk/blob/gsdk_4.2/app/bluetooth/example/bt_soc_app_ota_dfu/readme.md). 

> Note that the application level OTA DFU can be used only in a device with an internal flash larger than 256 k or with a device that has an external flash.

### Reading Metadata On-the-Fly

When you upload a new firmware image, you will receive the image in chunks. These chunks are received by the application and they can be written into any storage slot using the API of Gecko Bootloader. At the same time, when you get a chunk of the uploaded image, you can run the GBL parser on it, using the API of Gecko Bootloader. When the parser finds metadata in the GBL file, it calls a callback function and passes metadata to this function. Note the following:

- Metadata is sent in chunks to the callback function, so it may be called several times with small portions of the whole metadata.
- If the GBL file is encrypted, the parser will decrypt it and the callback function gets the decrypted metadata.

To initialize the GBL parser, use the following code:

``` c
#define BTL_PARSER_CTX_SZ  0x200
static uint8_t parserContext[BTL_PARSER_CTX_SZ];
static BootloaderParserCallbacks_t parserCallbacks;

parserCallbacks.applicationCallback = NULL;
parserCallbacks.bootloaderCallback = NULL;
parserCallbacks.metadataCallback = metadataCallback;
bootloader_init();
bootloader_initParser((BootloaderParserContext_t *)parserContext,BTL_PARSER_CTX_SZ);
```

Note that the parser context size must be at least 384 bytes.

To run the parser on a received chunk of data (data,len), use the following command:

```c
bootloader_parseBuffer((BootloaderParserContext_t *)parserContext,&parserCallbacks,data,len);
```

Finally, define the callback function, as follows:

```c
void metadataCallback(uint32_t address, uint8_t *data, size_t length, void *context)
{
       //process the received chunk of metadata (data,length)
}
```

### Reading Metadata from a Stored GBL File

A much easier solution, compared to on-the-fly parsing, is to retrieve the metadata from an already stored GBL file. To do so, first upload the GBL file without decoding it into a storage slot, defined by the bootloader. To retrieve the metadata from a GBL file stored in a storage slot, call the following function:

`bootloader_verifyImage(slotID,metadataCallback);`

and define the callback function, as follows:

```c
void metadataCallback(uint32_t address, uint8_t *data, size_t length, void *context)
{
       //process the received chunk of metadata (data,length)
}
```

Note that, although the whole GBL file is available in the storage slot, you will still get metadata in chunks because the parser has a limited buffer. The maximum size of a chunk is 64 bytes.

### Implementing the Callback Function

The metadata callback function is called several times and provides metadata in chunks. The callback function receives four parameters, as follows:

- **Address**: this is the offset address of the given chunk within the whole metadata
- **Data**: the pointer to the chunk of data
- **Length**: the length of the chunk. This varies between 1 and 64.
- **Context**: the parser context. Usually this can be ignored.

To assemble metadata from the chunks, use the following code snippet:

```c
#define MAX_METADATA_LENGTH   512
uint8_t metadata[MAX_METADATA_LENGTH];

void metadataCallback(uint32_t address, uint8_t *data, size_t length, void *context)
{
    uint8_t i;

    for (i = 0; i < MIN(length , MAX_METADATA_LENGTH - address); i++)
    {
        metadata[address + i] = data[i];
    }
}
```