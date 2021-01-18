## Setting up RPi4 network boot for laboratory project
## (aarch64, Alpine Linux) 
</br>

### Setting
</br>

For my task, I created topologically simple setting: RPi 4 - client - and my laptop - server - are connected via Ethernet.
</br>

----

Details of the use case


My goal was to create laboratory setting to explore Xen hypervisor internals and Xen toolstack.</br>
It means I will add diagnostic messages and experimental code, add/remove current features into the hypervisor and fix bugs if needed. The same scope of tasks is  applied to PV part of Linux kernel and device tree.</br>The nature of the work suggests booting new composite image (Xen binary + Linux kernel) everytime I boot the device, so network boot was crucial for the case.

----

</br>

### Client Side Configuration

I used Raspberry Pi 4B RAM 2GB.

Rasbperry Pi bootloader: pieeprom-2020-04-16.bin.</br>(https://github.com/raspberrypi/rpi-eeprom/tree/master/firmware)

Bootloader configuration parameters I changed:
* BOOT_UART=1
* BOOT_ORDER=0x21			(SD card - the first, network - the second)
* SD_BOOT_MAX_RETRIES=3
* NET_BOOT_MAX_RETRIES=5
* TFTP_PREFIX=1				(1 - “custom prefix for filenames”, prefix is a directory name)
* TFTP_PREFIX_STR=			(It’s empty string, because bootfiles are in root of tftp directory)


Original instructions how to change bootloader configuration are in the official documentation</br> (https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711_bootloader_config.md)

----

Side notes</br>The configuration mentioned above is enough to use DHCP.</br>Setting parameters CLIENT_ID, SUBNET and GATEWAY along with parameter TFTP_IP skips DHCP procedure as mentioned in the documentation.</br>Setting parameter TFTP_IP without CLIENT_ID, SUBNET and GATEWAY leads to the following scenario: firmware successfully finalizes 4-step DHCP exchange with a proper DHCP server (the one that provides expected PXE options) and than sends requests to TFTP server set in TFTP_IP.

----

</br>

### Server side configuration


I use minimal Ubuntu 18.04 on my laptop.

(http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/current/legacy-images/netboot/mini.iso)

Packages to support network boot:

* isc-dchp-server
* tftpd-hpa
* nfs-kernel-server


It’s worth to prepare scripts-helpers to restart the services:

dhcp.restart:

_sudo systemctl stop isc-dhcp-server.service_</br>
_sudo systemctl start isc-dhcp-server.service_</br>
_sudo systemctl status isc-dhcp-server.service_

tftp.restart:

_sudo systemctl restart tftpd-hpa.service_

nfs.restart:

_sudo systemctl restart nfs-kernel-server_

</br>

#### Network topology

Both Ubuntu laptop and RPi access home network (192.168.125.0) via wireless interface and get IP addresses with DHCP service provided by the third server.</br>
At the same time both the devices are connected with each other into subnet via Ethernet (192.168.7.0). In the latter subnet Ubuntu laptop is a DHCP server for RPi.

</br>

#### Configuration of dhcp server (isc-dchp-server)

Configuration file for DHCP server (/etc/dhcp/dhcpd.conf):

> allow booting;</br>
allow bootp;</br>
</br>subnet 192.168.7.0 netmask 255.255.255.0 {</br>
range dynamic-bootp 192.168.7.2   192.168.7.254;</br>
	option tftp-server-name “192.168.7.1";</br>
	option vendor-encapsulated-options  "Raspberry Pi Boot";</br>
	option vendor-class-identifier "PXEClient";</br>
}

Option “routers” (  option routers 192.168.7.1;  ) should NOT be filled in, otherwise route table on RPi will have 2 lines with “default” destination.
Option tftp-server-name is a string ( ! )

</br>

I put some side notes on setting DHCP in separate [small article](https://github.com/malus-brandywine/malus-brandywine/blob/master/Articles/RPi-netboot/docs/rpi4-netboot-aarch64-alpine-notes-1.md)

</br>

#### Configuration of tftp server (tftpd-hpa)

Configuration file /etc/default/tftpd-hpa:

> TFTP_USERNAME="tftp"</br>
TFTP_DIRECTORY="/mnt/tftpboot"</br>
TFTP_ADDRESS=":69"</br>
TFTP_OPTIONS="--secure"

/mnt/tftpboot is a symlink to “boot directory” (explained later)

</br>

#### Configuration of nfs server (nfs-kernel-server)

Configuration file /etc/exports:

/mnt/rpart \*(rw, sync,no_subtree_check,no_root_squash)

/mnt/rpart is a symlink to “root directory” (explained later)

