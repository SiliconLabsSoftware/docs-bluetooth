# Implementations of Non-Volatile Data Storage

This chapter introduces some of the challenges and design options when implementing non-volatile data storage in flash memory. It describes at a high level how non-volatile data storage is implemented in flash memory in PS Store, SimEEv1/v2, and NVM3.

## Basic Implementations

One of the characteristics of flash memory is that it is writable in smaller pieces, usually 32-bit words, while it can only be erased in larger chunks, usually pages of several kilobytes. When using flash as data storage, the most straightforward implementation option would be to store each data object in its own flash page, as shown in the following figure. This way each object can be erased and re-written without influencing the other data objects. Usually the data objects are much smaller than the page size, and this solution is not an effective way of using the available flash space.

![One Object Per Page](resources/one-object-per-page.png)

To avoid wasting flash space we can store several data objects in one flash page, as shown in the following figure. This solution then introduces a challenge when we want to write a new value to one of the data objects. In that case the page must be erased before all objects are written back to the page again, including the object we changed. As flash memory can only endure a limited amount of flash erases before the flash cells are worn out, this solution results in a very limited device lifetime.

![Multiple Objects in One Flash Page](resources/multiple-objects-in-one-flash-page.png)

To avoid erasing the flash page for every object write, we can instead write new versions of each object to new empty locations in the flash page. This is a simple form of wear-levelling that reduces the number of page erases. However, this requires that we store some identification information along with the object data that tells us what object the data belongs to, so we know how to find the latest version of the data object. This is illustrated in the following figure, where a key is added to each version of the object data to identify what object the data belongs to. When accessing an object we then need to search through the flash page for the most recent version of the object. In this case the newest version is the one with the highest address, as we start writing from the lowest address of the page.

![Object Versions With Keys](resources/object-versions-with-keys.png)

## Handling Resets and Power Failures

As we fill up the flash page with new versions of the data objects, eventually no room is left to write new object data. At this point we need to erase the page and start over by writing only the latest versions of each object to the flash page. In many applications, however, power failures or resets can happen at any time, and we should not risk losing data if this occurs. If a reset occurs after the flash page is being erased, but before the data objects are written back, then we will lose this data. To handle this case we introduce a second page, to which we copy the latest version of the data objects, before erasing the original page, as shown in the following figure Then we can start filling the second page with data. When the second page is full we move the latest data back to the first page and so on. This mechanism, where the storage is alternated between two flash pages, is how PS Store operates.

![Latest Data Copied to New Page Before Erase](resources/latest-data-copied-to-new-page-before-erase.png)

## Introducing Virtual Pages

In some applications we write to data objects frequently, and the flash pages therefore also need to be erased frequently. As the data objects in the implementation so far are only spread across two flash pages, each page will frequently get erased and the flash lifetime will be limited. To increase the lifetime we can use more flash pages to store the data objects. In this example, instead of two physical pages, we operate with two virtual pages (A and B) that each consist of several physical flash pages. The virtual pages are erased and written to as if they were one larger flash page. The difference is simply that each virtual page is bigger and we can write more data before we need to erase the virtual page, hence the lifetime is extended. In addition to increasing flash lifetime, using several flash pages per virtual page allows you to store more or larger objects. SimEEv1 uses this design, with each virtual page consisting of two flash pages, A and B, as shown in the following figure.

![Virtual Pages](resources/virtual-pages.png)

In some applications the time it takes to write a non-volatile data object must be minimized so as to not interfere with the timing of other critical operations. If an object write is triggered when a virtual page is full, the latest version of all objects must first be copied to the new virtual page before writing the new object version in question to the new page. All objects must be copied over immediately to allow the first page to be quickly erased so we can move data there in case of a failure. Copying all objects at once increases the worst-case object write time.

To reduce the write times, a third virtual page can be introduced that is always kept erased. Instead of copying over the latest version of every object when the first page is full, we can instead copy over only some of the objects. The rest of the objects are copied over as a part of subsequent write operations. This way we spread the copy process to the new page over more write operations, hence each write operation takes less time to complete. With this approach we have live object data spread out across two virtual pages and the third page is always kept erased, so we have somewhere to move the data to in case of a failure. SimEEv2 uses this implementation with 3 virtual pages where each virtual page consists of 6 flash pages.

## Basic Storage

Basic storage is defined as the size of all the latest version of all objects, including any overhead stored with them. Except for NVM3, every time a flash page or virtual page is erased we must first move the basic storage over to a new page. The basic storage size is important, as it determines how much flash space is left over in a page to store any new versions of the object data. If the basic storage takes up almost the entire page, we can only write a few new object versions before we need to move to a new page and erase the old one. This leads to frequent page erases and a short flash lifetime.

NVM3 only copies unique objects that are stored in the page that will be erased. If newer versions of the object exist in other pages, the object will not be copied.

## FIFO Model

The flash data storage implementations can be modelled as a First-In First-Out (FIFO) buffer (see the following figure), where we write new versions to the input of the FIFO. As the FIFO fills up, we need to free up space by erasing one or more pages at the end of the FIFO. Before we erase a page, we need to copy any object versions for which no newer version of the same object exists in the rest of the flash pages. Other object data can be discarded, because newer versions exist. To maximize flash lifetime we want to copy as few object versions as possible, so that most of the writes to the flash memory are new versions of the objects. If the FIFO is implemented over a large flash space, it is more likely that the new version of the object has been written and that object versions at the end of the FIFO can be discarded. In this case the flash page can be erased with few or no object versions copied.

![FIFO Buffer](resources/fifo-buffer.png)

A drawback of using a few virtual pages is that the available memory for the FIFO is limited to only the virtual pages that can hold live data. For implementations using two virtual pages, like SimEEv1, this means only half the storage space is used for the FIFO, while for SimEEv2 two thirds of the storage space is used for the FIFO.

To allow a higher portion of the storage space to be used for the FIFO, we can instead implement the FIFO as a circular buffer over the entire flash storage space allocated, as shown in the following figure. In this implementation we always need to keep enough erased pages in front of the leading edge of the buffer to write the largest-sized object in case of a failure. When the FIFO fills up to reach the critical number of erased flash pages, we copy over any object versions that have not been superseded and erase the page at the back of the FIFO. This means that, instead of keeping a full virtual page erased, we only need to keep enough space erased to fit the largest-sized object. We can use the rest of the space for the FIFO. NVM3 is implemented as a circular buffer implemented over the entire storage space, thus increasing flash lifetime compared to implementations using smaller virtual pages.

![Circular Buffer](resources/circular-buffer.png)

## Counter Objects

For some types of data, the storage format can be optimized for the flash medium. EFR32 Series 0 and 1 can write each word two times, allowing for half word updates.  Series 2 devices allow just one write per word.

For example, counter values are usually incremented by 1 or some other low value every time they are written. Normally this means that we would have to write the entire counter value in addition to identification bytes every time the counter is incremented. We can optimize this by only storing a start value in addition to the identification value the first time a counter is written. Then we reserve a number of the words following the initial value for writing increments. The flash words in Silicon Labs devices EFR32 Series 0 and 1 can write words twice between each erase by keeping a 1 in the bits that are not to be changed.

As an example, assume we are writing two 16 bit values, 0xAAAA and 0x5555. To safely write them in the same flash word this method can be used:

- Write 0xFFFFAAAA (word in flash becomes 0xFFFFAAAA)

- Write 0x5555FFFF (word in flash becomes 0x5555AAAA)

Note that there is a maximum of two writes to the same word between each erase due to a physical limitation of the flash.

To find the current counter value, we start with the initial value and add the increment values in the halfwords following the initial value. This means that we only need to write one halfword for each increment, instead of the whole counter value and identification value. NVM3 and SimEEv1/v2 support counter objects.

## Indexed Objects

When storing data such as arrays in flash, we often only update one index at a time. If we store the whole array as one regular object, we must write the whole array to flash even if just one index is updated. Instead of storing the entire array in one object, we can divide each data array over multiple objects and only update the objects holding the changed array indexes. While it is possible to split arrays into multiple objects manually for all the Silicon Labs storage implementations, SimEEv1/v2 allow all of the indexes to share one object key. The index entry into the array to be looked up is then provided as a separate parameter.

NVM3 does not support reading and writing parts of an object. If the application wants to access indexes individually, each index must be accessed using a unique key.
