
#### Side notes to DHCP exchange

Bootloader of 04-16-2020 (pieeprom-2020-04-16.bin) requests the following info from server:

Subnet-Mask (option 1),</br>
Default-Gateway (option 3),</br>
Vendor-Option (option 43),</br>
Vendor-Class (option 60),</br>
TFTP (“tftp-server-name”, option 66),</br>
BF (“boot file”, option 67)

DHCP server fills in Subnet-Mask from description of subnet; Default-Gateway can be skipped (and should be, in this case); Vendor-Class can be skipped (tested), but I kept it for future compatibility; BF can be skipped also.

So,</br>
* range dynamic-bootp 192.168.7.2   192.168.7.254;</br>
* option tftp-server-name “192.168.7.1";</br>
* option vendor-encapsulated-options  "Raspberry Pi Boot";

are mandatory for the bootloader to choose the DHCP server
![DHCP notes](https://github.com/malus-brandywine/malus-brandywine/blob/master/Articles/RPi-netboot/docs/notes-dhcp.jpg)
