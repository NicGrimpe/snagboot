Une fois dans le namespace
```
snagrecover -s am3358 -f src/snagrecover/templates/am335x-beaglebone-black.yaml
```
Ca vas downloader une version de U-Boot sur le board.

## Flashing an image with ums
```
U-Boot# mmc dev 1
U-Boot# ums 0 mmc 1
```
```
HOST$ lsblk
HOST$ sudo dd if=sdcard.img of=/dev/sdb status=progress
```

# The rest is exploratory and not fully working yet

## Fastboot exploration 

Dans la console U-Boot roule
```
fastboot usb 0
```

Pour tester que ça a bien work depuis le namespace roule:
```
sudo snagflash -P fastboot -p 0451:d022 -f getvar:version-bootloader
```

```
sudo snagflash -P fastboot -p 0451:d022 -f download:/home/nic/ideo/outputs/rs485/images/boot.vfat
sudo snagflash -P fastboot -p 0451:d022 -f flash:0.0
```

## Reformat the disk from u-boot
```
U-Boot# mmc list
sdhci@fa10000: 0 ()
sdhci@fa00000: 1 (eMMC)
```
```
U-Boot# setenv partitions 'uuid_disk=${uuid_gpt_disk};name=boot,start=2MiB,size=64M,bootable,uuid=${uuid_gpt_boot};name=permanent,size=16M,uuid=${uuid_gpt_permanent};name=rootfsA,size=512M,uuid=${uuid_gpt_rootfsA};name=rootfsB,size=512M,uuid=${uuid_gpt_rootfsB}'
```
```
U-Boot# uuid
...first uuid...
U-Boot # setenv uuid_gpt_disk ...first uuid...
```
```
U-Boot# uuid
...first uuid...
U-Boot # setenv uuid_gpt_boot ...first uuid...
```
```
U-Boot# uuid
...first uuid...
U-Boot # setenv uuid_gpt_permanent ...first uuid...
```
```
U-Boot# uuid
...first uuid...
U-Boot # setenv uuid_gpt_rootfsA ...first uuid...
```
```
U-Boot# uuid
...first uuid...
U-Boot# setenv uuid_gpt_rootfsB ...first uuid...
```
```
U-Boot# gpt write mmc 1 ${partitions}
```

You then need to reset. And you should see the new partitions after.
```
U-Boot# reset
resetting ...

U-Boot# mmc dev 1
switch to partitions #0, OK
mmc1(part 0) is current device

U-Boot# mmc part

Partition Map for MMC device 1  --   Partition Type: EFI

Part    Start LBA       End LBA         Name
        Attributes
        Type GUID
        Partition GUID
  1     0x00001000      0x00020fff      "boot"
        attrs:  0x0000000000000004
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   63a87d4d-3b3f-47f6-8baa-aa2536ac856f
  2     0x00021000      0x00028fff      "permanent"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   2f7aada0-bb26-4bf2-8db8-ad533b212171
  3     0x00029000      0x00128fff      "rootfsA"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   0b5e6c9f-fdbf-4b81-a7f4-7828bdec75e8
  4     0x00129000      0x00228fff      "rootfsB"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   30a562bf-92ca-4e93-945d-990b296c4227
```

## Flashing bootloader via dfu (Very slow)

```
U-Boot# setenv dfu_alt_info 'boot part 1 1'
U-Boot# printenv dfu_alt_info
U-Boot# dfu 0 mmc 1
```

```
HOST$ sudo snagflash -P dfu -p 0451:d022 -D 0:/home/nic/ideo/outputs/rs485/images/boot.vfat
```