# ArchLinux ARM on Firefly-rk3288
This is a tutorial teaching how to install ArchLinux ARM on the Firefly-rk3288 over a USB stick.

This repository also contains some scripts to help on the installing process.

After following this tutorial, you should have a fresh working ArchLinux ARM install with wifi, vpu and mali gpu working thanks to libhybris

## Requirements
### Necessary Requirements
For installing ArchLinux ARM, we will need the following hardware

- SD Card (where we will put the linux kernel and bootloader)
- USB stick (that will be used to install the root filesystem)
- Keyboard
- Monitor for enjoying your ArchLinux
- Another computer running linux

### Extra requirements for building from scratch
- Ethernet cable
- Another computer with a running linux with Ethernet connection

## Introduction
In a higher level, what we will do in this tutorial is:
- Compile Linux Kernel and generate images
- Create the Arch Linux root filesytem
- Install libhybris on Arch Linux and enjoy

The expert linux reader can realize that the steps are pretty general and they can be used to install any linux distribution, not only ArchLinux, with minor tweaks

## Download software 
In this repository you can find a script called setup. This script will help you to download and set the necessary tools

Go to the repository head and type `. ./setup tools` or keep with the manual installation bellowyou can keep with the manual installation

- Fetch and install mkbootimg
```bash
git clone https://github.com/neo-technologies/rockchip-mkbootimg.git
cd rockchip-mkbootimg
make
export PATH=$PATH:`pwd`
cd ..
```
- Fetch rk2918 tools
```bash
git clone https://github.com/dayongxie/rk2918_tools.git
export PATH=$PATH:`pwd`/rk2918_tools
```
- Dowload ARM eabi cross compilling toolchain and export paths
```bash
wget https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6/+archive/refs/tags/android-4.4.2_r1.tar.gz
mkdir android-4.4.2_r1
tar -zxf android-4.4.2_r1.tar.gz -C android-4.4.2_r1
export ARCH=arm
export CROSS_COMPILE=./android-4.4.2_r1/bin/arm-eabi-
export LD_LIBRARY_PATH=./android-4.4.2_r1/lib:./android-4.4.2_r1/libexec/gcc/arm-eabi/4.6.x-google
```
After downloading all necessary tools, please export the path variables with `. ./setup source`.

## Build kernel

For bulding the kernel, you can run `./setup kernel` or you can build it manually with
```bash
git clone https://bitbucket.org/T-Firefly/firefly-rk3288-kernel.git
cd firefly-rk3288-kernel
make firefly-rk3288-linux_defconfig
make -j10 firefly-rk3288.img
make -j10 modules
[ -d modules_install ] && rm -rf modules_install
mkdir modules_install
make INSTALL_MOD_PATH=./modules_install modules_install
cd ..
```

## Build ramdisk
After building the kernel, you can run `./setup ramdisk` or you can manually type the commands
```bash
git clone https://github.com/TeeFirefly/initrd.git
make -C initrd
```

## Build boot image
After that the tools, kernel and ramdisk are setup and ready, lets make a boot image. 

For generating the boot image, please run the commands bellow or `setup boot_img`:
```bash
mkbootimg --kernel path/to/kernel.img --ramdisk path/to/initrd.img -o linux-boot.img
```

## Create SD Card
For flashing the new kernel,boot and resource images to the SD card, we need to copy linux-boot.img, kernel.img and resource.img to the folder. You can do it simply by typing `./setup create_sdcard_usb` or manually by typing. 

```bash
cp path/to/linux-boot.img path/to/create-linux-sdcard-usb/linux-boot.img
cp path/to/firefly-rk3288-kernel/kernel.img path/to/create-linux-sdcard-usb/kernel-linux.img 
cp path/to/firefly-rk3288-kernel/resource.img path/to/create-linux-sdcard-usb/resource-linux.img
```

If you are doing the steps manually, go to the create-linux-sdcard-usb folder and type `sudo ./create-linux-sdcard-usb` and follow the instructions, otherwise you should have a working kernel now

## Build Archlinux RFS (Root Filesystem)
In this point, if you have an already working root filesystem, just flash it to the USB stick and you are ready to go. If not, just keep following the tutorial

We will base our build on the arch-linux veyron rfs. 

### Get ArchLinux ARM veyron
Just download archlinux-veyron rfs with:
```bash
wget http://archlinuxarm.org/os/ArchLinuxARM-veyron-latest.tar.gz
```

### Format your USB stick
For the booting process to complete, we need that the first partition on the driver needs to be labeled as 'linuxroot'.

Please generate a partition table that pleases you with fdisk with a partition for the root filesystem larger than 700MB.

After that, lets format the USB stick with the following command
```bash
# Supposing the USB stick is under /dev/sdb
sudo mkfs.ext4 -F -L linuxroot /dev/sdb1
```
### Copy ArchLinux root filesystem to the USB stick
For doing that, we will first mount the USB stick and than untar the archlinux-veyron tarball to the mounted pen drive
```bash
sudo mount /dev/sdb1 /mnt
sudo tar -zxf ArchLinuxARM-veyron-latest.tar.gz -C /mnt
sync
```

Congratulations ! Now you have a working system ! 

If you dont need wifi or the vpu working, you can stop now, insert the usb stick and SD card into the firefly and enjoy your new ArchLinux install
Else, continue following the tutorial

## Install libhybris
Thanks to the great contribution of mac-l1, there is currently a .deb package that can be found [here](http://mac-l1.com/). Since a .deb package is not an ArchLinux dedicated package, it's necessary to generate a tarball from the .deb package. There are many programs that do this job, like [Alien](https://joeyh.name/code/alien/)

After the conversion is done, just use the provided script `setup_machybris` or unpack and copy all machybris files to the arch linux root filesystem. This can be done with:
```bash
mkdir machybris
tar zxf path/to/tgz/machybris -C machybris
rsync --exclude=etc/init -abviuzP machybris/* path/to/arch/linux/rfs
sync
umount path/to/arch/linux/rfs
```

For enabling the wifi module, we need to create an special unit file for systemd. Create a file called `wifi.service` and place it at `path/to/arch/linux/rfs/etc/systemd/system/`. Now, copy the following information under this file
```bash
[Unit]
Description=wifi
After=libhybris.service

[Service]
ExecStart=/usr/local/bin/wifi-on.sh
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Now, we need to create the script that will actually load the driver. Create a file called `wifi-on.sh` and place it at `path/to/arch/linux/rfs/usr/local/bin/` containing the following code:
```bash
# turn on wifi
while [ ! -f /sys/class/rkwifi/power ]
do
    sleep 1
done
    
echo 1 > /sys/class/rkwifi/power

# load module
while [ ! -f /sys/class/rkwifi/driver ]
do
    sleep 1
done

echo 1 > /sys/class/rkwifi/driver
```

Now, just unplug the USB stick from your computer, plug it in your firefly with you SD card and turn it on. 

You should now see the console. Loggin with user and password "root".

After beeing logged in, we need to enable libhybris with systemd
```bash
systemctl enable libhybris
systemctl enable android-logd
systemctl enable android-mediaserver
systemctl enable android-servicemanager
systemctl enable fbset
systemctl enable wifi
```

reboot so the changes take place with `reboot`.

Your firefly is configured and running ArchLinux ! \o/

Any suggestion and/or improvement, please fork me ! :)
