# Comparing Non-Volatile Data Storage Implementations

The following table presents an overview of the main features of the various Silicon Labs Non-Volatile Data Storage implementations.

| Feature |NVM3 |SimEEv1 |SimEEv2 |PS Store |
|-|-|-|-|-|
| Compatible devices |EFM32, EFR32 |EFR32 Series 1 | EFR32 Series 1 |EFR32 Series 1 |
| Compatible radio protocols |EmberZNet, Connect, Z-Wave, OpenThread, Bluetooth |EmberZNet, Connect |EmberZNet, Connect |Bluetooth |
| Flash used for storage |3 or more flash pages |8 KB |36 KB |4 KB |
| Virtual pages |NA |2 |3 |NA |
| Max basic storage (Sum of all object data with overhead) |Variable (see [Using Third Generation Non-Volatile Memory (NVM3) Data Storage](https://docs.silabs.com/gecko-platform/latest/using-third-generation-nonvolatile-memory/) for details) |2 KB |8 KB |2 KB |
| Max number of objects |Limited by basic storage |254 |254 |Limited by basic storage |
| Max object size (bytes) |Configurable 208-4096 |254 |254 |56 |
| Object creation/deletion |Runtime |Compile time |Compile time |Runtime |
| Compiled flash size excluding data storage |9.1 KB |3.5 KB |5.4 KB |1.6 KB |
| Counter object support |Yes |Yes |Yes |No |
| Indexed object support |Partial (note 1) |Yes |Yes |No |
| Overhead per object (bytes) |4 (size \<= 128 bytes); 8 (size > 128 bytes) |2 |6 |10 |
| Counter object size including overhead (bytes) |212 |60 |56 |NA |
| Counter increments before rewrite |100 (EFR32 Series 1); 50 (EFR32 Series 2) |25 |25 |NA |

Note 1: When using NVM3, indexed objects are implemented by storing each index in a separate NVM3 object.

## Flash Lifetime

All Silicon Labs Flash Data Storage implementations use some form of wear-levelling to prolong flash lifetime. The effectiveness of the wear-levelling depends on the implementation, the type of data stored, and how often it is updated. The main factors that affect wear-levelling and thereby flash lifetime are:

- Size of flash used for data storage: More flash area gives longer flash lifetime. For NVM3, the number of flash pages used for data storage can be configured, while the rest of the implementations use fixed storage sizes.

- Stored overhead per object: When writing data to the object storage, some overhead bytes are added to identify the data. Implementation with less overhead means the data objects take up less space in flash, and gives longer flash lifetime.

- Alignment to minimum object size: Objects are stored in multiples of the smallest object size. If the data size does not align with this size, padding bytes are added, which adds to the stored data and reduces flash lifetime. For instance, when storing 16-bit objects, NVM3 and PS Store add two extra bytes of padding in addition to the overhead bytes. SimEEv1/v2 are able to store 16-bit data objects without padding.

- Remaining storage after basic storage: For implementations using virtual pages, when switching to a new virtual page one instance of each object is written to the page. The rest of the virtual page can then be used to store new writes of the objects. If a lot of space is used to store one instance of each object, little space is left in the virtual page to use for wear-levelling the subsequent object writes. The flash lifetime will therefore be reduced when the total amount of object data is large relative to the virtual page size. Even for NVM3, where virtual pages are not used, the flash lifetime is limited by the available space of the total NVM3 storage.

To help monitor the actual flash wear, NVM3 and SimEEv1/v2 include function calls for reporting the number of page erases of the data storage flash pages. These erase counters can be read during accelerated lifetime testing of a product to verify if the flash wears at an acceptable rate.
