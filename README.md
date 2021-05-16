# Download Android source with local_manifests
 Refer to http://source.android.com/source/downloading.html

```bash
$ mkdir android4pi && cd android4pi #create folder and cd to it
$ repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r34
$ git clone https://github.com/nguyenanhgiau/local_manifests .repo/local_manifests -b rpi4-a11-telephony
$ repo sync -j8
 ```

# Build for Raspberry Pi 4
 https://github.com/android-rpi/device_arpi_rpi4/tree/arpi-11

Use -j[n] option on sync & build steps, if build host has a good number of CPU cores.<br>
For example:
```bash
$ make -j16
```

# Write image to SDCard

Suppose your drive is **/dev/sdb**, run command:
```bash
$ ./scripts/android_flash_rpi4.sh sdb
```
