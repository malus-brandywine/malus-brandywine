### Preparing file systems

The project maintains two filesystems that correspond with “boot” and “root” partitions that RPi boot procedure relies on.

“Boot directory” contains number of boot files and a directory with device tree overlays.</br>
“Root directory” contains root filesystem. 


I chose to use Alpine Linux since its a minimal distro with aarch64 userland and openRC init scripts.

“Root directory”. Default installation of Alpine Linux is diskless (RAM based). initramfs is loaded into RAM, then init scripts uncompress apk packages into root filesystem. For my project I need file tree deployed on laptop disk which I will mount via NFS.
So, I made disk-based installation following the original instructions (https://wiki.alpinelinux.org/wiki/Raspberry_Pi), then copied the deployed filesystem on a laptop. I write down short version of instructions suitable for my case in the next chapter.

“Boot directory”. Bootfiles & dt overlays are copied from Alpine distro.

Directories supporting network boot:

* /home/bpart/alpine			- “boot directory”</br>
* /home/rpart/alpine			- “root directory”</br>
* /mnt/tftpboot -> /home/bpart/alpine	- TFTP root directory</br>
* /mnt/rpart -> /home/rpart/alpine		- directory shared via NFS


#### Root filesystem

Creating disk-based installation

I use Alpine Linux 3.12.0 (alpine-rpi-3.12.0-aarch64.tar.gz)
</br>
</br>

----

Side notes. Structure of installation disk briefly:

Root directory</br>
* RPi boot files (start.elf, … except Linux image)</br>
* directory “overlays” (dt overlays)</br>
* directory “boot”</br>
* directory “apks”

Directory “boot”</br>
* kernel images “vmLinuz-rpi”, “vmLinuz-rpi4”</br>
* kernel config files “config-rpi”, “config-rpi4”</br>
* “modloop-rpi”, “modloop-rpi4” - disk images containing kernel modules and firmware</br>
* “initramfs-rpi”, “initramfs-rpi4” - base root fs

Directory “apks”</br>
* /aarch64/*.apk - packages archived</br>
* /aarch64/APKINDEX.tar.gz - index of local packages


----
</br>

##### Brief instructions

</br>

Full instruction set is on the official Alpine wiki:  https://wiki.alpinelinux.org/wiki/Raspberry_Pi

</br>

On an SD card create 2 primary partitions, 150M and 200M (minimums). Make first one FAT 32, the second one Linux.

* Create both filesystems and mount the first partition:</br>
	* mkdosfs -F 32 /dev/sdX1</br>
	* mkfs.ext4 /dev/sdX2

* Download alpine-rpi-x.x.x-aarch64.tar.gz, unpack it, copy the extracted file tree into the first partition. The SD card is now bootable
* Run target with the SD card inserted, login “root” with empty password, then run</br>
	* setup-alpine
		* on request “Enter mirror number (1-48) or URL …” choose “r” to add  random mirror
		* memorize hostname you choose</br>
	* apk update; apk upgrade</br>
	* rc-update add wpa_supplicant boot</br>
	* apk add nano</br>
	* lbu commit -d</br>
	* reboot
				
* Diskless installation is ready. Next, continue to disk-based one.</br>
	* mount   /dev/mmcblk0p2  /mnt</br>
	* setup-disk  -o /media/mmcblk0p1/{hostname}.apkovl.tar.gz   /mnt</br>
	* poweroff target, copy the second partition of SD card into /home/rpart/alpine
				
* Some adjustments to make init scripts working:</br>
	* mkdir /home/rpart/alpine/.modloop</br>
	* ln -s  ../lib/modules  /home/rpart/alpine/.modloop/modules</br>
	* Disable auto bringup of eth0. In file /etc/network/interfaces/ comment line “auto eth0”


* Important: don’t forget to copy /lib/modules/{version} which correspond with your kernel into /home/rpart/alpine/lib/modules 

</br>

Note. Additional changes to enable console on uart0:
* /etc/inittab: add line "hvc0::respawn:/sbin/getty -L hvc0 115200 vt102"
* /etc/securetty: add a line "ttyS0"
* /etc/ssh/sshd_config: set "PasswordAuthentication yes" and "PermitRootLogin yes"


</br>
Directory /home/rpart/alpine is ready to be mounted via NFS

</br>
</br>

I put a log of bootloader into [small note](https://github.com/malus-brandywine/malus-brandywine/blob/master/Articles/RPi-netboot/docs/rpi4-netboot-aarch64-alpine-notes-2.md)

</br>

### Shortcuts to official pages

#### raspberrypi.org

Debugging the network boot mode:</br>
https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/net.md

Bootloader parameters and command to update bootloader configuration in EEPROM:</br>
https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2711_bootloader_config.md

How to update firmware:</br>
https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md


#### wiki.alpinelinux.org

RaspberryPi installation:</br>
https://wiki.alpinelinux.org/wiki/Raspberry_Pi

