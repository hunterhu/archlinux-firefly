# Get the rootfs.tar.bz2

I put a tiny default rootfs tarball, but github's limit of 50MB doesn't allow
upload it, which is 70MB.

Either prepare your own rootfs.tar.bz2 file and put it
inside create-linux-sdcard folder, or download this version:

https://drive.google.com/open?id=1gqKaSuIgarLmIg3kC9fmFAG1xP4mP1yP

# Creaet a Firefly-rk3288 SD card

```bash
./setup all
```

For details of how every step is done see the setup script itself.


# Prepare RK3288 bootloader

In order to let RK3288 boot from SD card, you will need a later bootloader
version, I have include it in the repo:

```bash
RK3288Loader_uboot_V2.17.02.bin
```

Enter loader mode or MaskRom mode to flash the bootloader file.

## 32-bit machine

You will need a 32-bit machine to work with the Rockchip loader mode,
64 bit machine won't recognize the Rockchip usb device.

Consider yourself warned on this.

## loader mode
In Loader mode, bootloader will enter into upgrade state, waiting for commands
from host, which is used in firmware upgrading.

To enter Loader mode, make the bootloader aware that the RECOVERY key is pressed
and USB cable is connected:

1.Keep device power on.
2.Use micro USB OTG cable to connect host and device together.
3.Press and hold RECOVERY key.
4.Shortly press RESET key.
5.Release RECOVERY key.

## MaskRom Mode
MaskRom mode is used to fix system when bootloader is broken.

In most cases, there are no need to enter MaskRom mode. When the BootRom code
fails to verify bootloader (cannot read IDR block, or bootloader is broken), it
will boot into MaskRom mode, which waits for host to send bootloader code
through USB, then runs it, so that bootload can take control again.

To enforce device into MaskRom mode, please check:
http://en.t-firefly.com/doc/product/info/401.html

## flash the bootloader

I have include the v1.33 Linux upgrade_tool, after entering loader mode or
Maskrom mode, do the following on your board:

```bash
sudo ./upgrade_tool ul RK3288Loader_uboot_V2.17.02.bin
```

Insert the SD card into a firefly rk3288 board and happy developing.
