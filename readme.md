# How to flash a beaglebone via usb

## Installation
```
sudo pacman -Syu hidapi swig
python -m venv env
source env/bin/activate
./install.sh
```

Add this udev rule to be able to run the scripts without sudo
```
sudo cp 50-snagboot.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## Flashing a board

Prepare the recovery environnement.
```
sudo ./am335x_usb_setup.sh
```
Then power the board with the S2 button pressed. You should see the character 'C' printed on your serial console.
```
snagrecover -s am3358 -f src/snagrecover/templates/am335x-beaglebone-black.yaml
```
It will load U-Boot in volatile memory. Then in the u-boot shell
```
U-Boot# mmc dev 1
U-Boot# ums 0 mmc 1 // This operation might fail, your cursor should spin if successfull.
```
On your host computer you should see a new block device corresponding to the internal eMMC of the target.
```
HOST$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sdb           8:16   1   3.5G  0 disk 
├─sdb1        8:17   1    64M  0 part 
├─sdb2        8:18   1    16M  0 part 
├─sdb3        8:19   1   1.7G  0 part 
└─sdb4        8:20   1   1.7G  0 part

HOST$ sudo dd if=sdcard.img of=/dev/sdb status=progress
```