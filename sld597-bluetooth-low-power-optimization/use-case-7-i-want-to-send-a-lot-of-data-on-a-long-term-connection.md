# Use Case #7: I want to send a lot of data on a long-term connection

Sending a lot of data on a Bluetooth connection is not a low energy use case by its nature. However, there are a few things to consider to save energy:

- First, using a 2 Mbps data rate with 2M PHY is not only good because it ensures a higher transmission speed, but it also decreases power consumption because of the shorter transmission time. Therefore it is strongly suggested to apply 2M PHY with `sl_bt_connection_set_default_preferred_phy()`, unless you need long-range communication.

- Another important setting is the TX power. To use optimal TX power take advantage of the LE Power Control feature of the Bluetooth stack, which automatically adjusts the TX power based on the RSSI level reported by the receiver. LE Power Control is described in [Use Case #6: I want to maintain a long-term connection with occasional data exchange](./use-case-6-i-want-to-maintain-a-long-term-connection-with-occasional-data-exchange).

- In a busy environment, it may be worth enabling Adaptive Frequency Hopping (AFH). AFH scans through the 2.4 GHz frequency band, and disables Bluetooth channels where interference (for example with Wi-Fi) is detected. Although the scanning means additional energy consumption, avoiding packet retransmission caused by interference may save more than the added energy. See [Adaptive Frequency Hopping](/bluetooth/{build-docspace-version}/bluetooth-fundamentals-system-performance/afh) for more details about AFH.
