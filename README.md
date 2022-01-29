# Building Android for Rpi4
## General Notes
This is project experimental;<br>
It is not guaranteed to build on all possible systems and it is not guaranteed to run on all Raspberry Pi configurations.<br>
You need a fair amount of knowledge of Linux and Android to get this to work. Be warned!<br>
Prerequisites:<br>
- RAM is greater than 16GB
- CPU as much core as possible
- Free space is greater than 200GB

Of course, you need a [Rpi4ModelB4G_RAM](https://hshop.vn/products/may-tinh-raspberry-pi-4-model-b-made-in-uk).<br>
In addition, you also need a 7" HDMI Touch Screen 1024x600. There are many available. The one I use is from [Waveshare](https://hshop.vn/products/man-hinh-7-inch-hdmi-lcd-h-cam-ung-dien-dung-waveshare-co-vo-bao-ve).
## Download Android source with local_manifests
Refer this link for install git and repo tool: https://source.android.com/setup/develop<br>
Then, you can download the android source by the command:

```bash
mkdir android4pi && cd android4pi #create folder and cd to it
# remove --depth=1 if you want to care about the git history but the size of the download will increase
repo init --depth=1 -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r37
git clone https://github.com/nguyenanhgiau/local_manifests .repo/local_manifests -b rpi4-a11-telephony
repo sync -j14 --force-sync
 ```
Reference download the android source: http://source.android.com/source/downloading.html<br>

## Building Kernel

Because the kernel project haven't integrated into AOSP building, so we have to build it separately.
```bash
sudo apt install gcc-aarch64-linux-gnu libssl-dev bc
cd android4pi/kernel/arpi
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make lineageos_rpi4_defconfig
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make Image.gz dtbs -j16
```
## Building AOSP

Before building, you have to apply this patch https://github.com/android-rpi/device_arpi_rpi4/wiki/arpi-11-:-framework-patch
```bash
croot
source build/envsetup.sh
lunch rpi4-eng
make ramdisk systemimage vendorimage -j14
```
Use -j[n] option with make, if build host has a good number of CPU cores.<br>
Reference: http://source.android.com/source/building.html
## Writing the image to sdcard

```bash
croot
./scripts/android_flash_rpi4.sh sdb #suppose your sd card is sdb
```

Now, you can unplug your sdcard and plug on your rpi4, setup and enjoy!<br>

# ADB Connection
Because the Raspberry Pis do not have a USB OTG port, the only way to connect ADB is via Internet.<br>
Plug your Raspberry Pi into your Ethernet LAN or wifi, then wait for it to be given an IP address.<br>
After that, just run command:<br>
```bash
$ adb connect <IP_ADDRESS>:5555 #192.168.1.10:5555
```

**HAVE FUN!**
