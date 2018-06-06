# Creaet a Firefly-rk3288 SD card

```bash
./setup all
```

For details of how every step is done see the setup script itself.


# Prepare RK3288 bootloader

In order to let RK3288 boot from SD card, you will need a later bootloader
version; try boot the sdcard you created, if not booting, flash this one that I
have included in the repo:

```bash
RK3288Loader_uboot_V2.17.02.bin
```

Enter loader mode or MaskRom mode to flash the bootloader file.

## 32-bit machine

You will need a 32-bit machine to work with the Rockchip loader mode,
64 bit machine won't recognize the Rockchip usb device.

## Loader mode
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

## Flash the bootloader

I have include the v1.33 Linux upgrade_tool, after entering loader mode or
Maskrom mode, do the following on your board:

```bash
sudo ./upgrade_tool ul RK3288Loader_uboot_V2.17.02.bin
```

Insert the SD card into a firefly rk3288 board and happy developing.
