# Building Android for Rpi4
## General Notes
This is project experimental;<br>
It is not guaranteed to build on all possible systems and it is not guaranteed to run on all Raspberry Pi configurations.<br>
You need a fair amount of knowledge of Linux and Android to get this to work. Be warned!<br>
Prerequisites:<br>
- RAM is greater than 16GB
- CPU as much core as possible
- Free space is greater than 200GB

You can refer my build machine:
- Running on Ubuntu 18.04
- RAM 16GB
- CPU 8 cores and 16 threads

Of course, you need a [Rpi4ModelB4G_RAM](https://hshop.vn/products/may-tinh-raspberry-pi-4-model-b-made-in-uk).<br>
In addition, you also need a 7" HDMI Touch Screen 1024x600. There are many available. The one I use is from [Waveshare](https://hshop.vn/products/man-hinh-7-inch-hdmi-lcd-h-cam-ung-dien-dung-waveshare-co-vo-bao-ve).
## Download Android source with local_manifests
Refer this link for install git and repo tool: https://source.android.com/setup/develop<br>
Then, you can download the android source by the command:

```bash
$ mkdir android4pi && cd android4pi #create folder and cd to it
$ repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r34
$ git clone https://github.com/nguyenanhgiau/local_manifests .repo/local_manifests -b rpi4-a11-telephony
$ repo sync -j$(your core)
 ```
Reference download the android source: http://source.android.com/source/downloading.html<br>

## Build Kernel
```bash
$ sudo apt install gcc-aarch64-linux-gnu libssl-dev bc
$ cd android4pi/kernel/arpi
$ ARCH=arm64 scripts/kconfig/merge_config.sh arch/arm64/configs/bcm2711_defconfig kernel/configs/android-base.config kernel/configs/android-recommended.config
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make Image.gz
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS="-@" make broadcom/bcm2711-rpi-4-b.dtb
$ ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS="-@" make overlays/vc4-kms-v3d-pi4.dtbo
```
## Build Android source

Before building, you have to apply this patch https://github.com/android-rpi/device_arpi_rpi4/wiki/arpi-11-:-framework-patch
```bash
$ cd /path/to/android4pi
$ source build/envsetup.sh
$ lunch rpi4-eng
$ make ramdisk systemimage vendorimage -j$(your core)
```
Use -j[n] option with make, if build host has a good number of CPU cores.<br>
Reference: http://source.android.com/source/building.html
## Write image to sdcard
### Prepare sd card
Partitions of the card should be set-up like followings.<br>
p1  128MB for boot : Do fdisk, set W95 FAT32(LBA) & Bootable type, mkfs.vfat<br>
p2 1024MB for /system : Do fdisk, new primary partition<br>
p3  128MB for /vendor : Do fdisk, new primary partition<br>
p4 remainings for /data : Do fdisk, mkfs.ext4<br>
Set volume label of /data partition as userdata<br>
: use -L option for mkfs.ext4, and -n option for mkfs.vfat<br>
 
### Writing image
```bash
# Write system.img and vendor.img
$ cd out/target/product/rpi4
$ sudo dd if=system.img of=/dev/<p2> bs=1M
$ sudo dd if=vendor.img of=/dev/<p3> bs=1M
# Copy kernel & ramdisk to BOOT partition
$ cd /path/to/android4pi
$ sudo cp device/arpi/rpi4/boot/* to p1:/
$ sudo cp kernel/arpi/arch/arm64/boot/Image.gz to p1:/
$ sudo cp kernel/arpi/arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb to p1:/
$ sudo mkdir p1:/overlays
$ sudo cp kernel/arpi/arch/arm64/boot/dts/overlays/vc4-kms-v3d-pi4.dtbo to p1:/overlays/
$ sudo out/target/product/rpi4/ramdisk.img to p1:/
```
You can use the below script for both preparing and writing image.<br>
Suppose your drive is **/dev/sdb**, run command:
```bash
$ cd /path/to/android4pi
$ ./scripts/android_flash_rpi4.sh sdb
```
Now, you can unplug your sdcard and plug on your rpi4, setup and enjoy!<br>
**HAVE FUN!**
