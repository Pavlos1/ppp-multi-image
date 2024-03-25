# PinePhone Pro multi-distribution SD card creator

A shell script to automatically create a bootable multi-distribution SD card
for the PinePhone Pro.

## Quick start

Ensure the following packages are installed:

* `wget`
* `rsync`
* `unzip`

Change directory to the project root:

```shell
cd /path/to/ppp-multi-image
```

Run the following command to download the images/tarballs for each distro (do
**not** run as root):

```shell
./download.sh
```

Any time you pull new updates from the repository, or make manual changes to
the distro parameters (i.e. in `distros/*/config`), you will need to re-run
`download.sh` to download the updated files.

Finally, run the following command to generate the image:

```shell
./mkimage.sh [-p <partition size>] /dev/[DEVICE]
```

Replace `/dev/[DEVICE]` with the device file of your SD card. Each distribution
will be allocated its own partition; use the `-p` argument to specify the size
of these partitions. If no `-p` argument is provided a default size of 10 GiB
per partition will be used.

The `-p` argument will be passed straight to `sfdisk`, so refer to `man sfdisk`
regarding how to specify sizes accurately (e.g. GiB vs GB).

Currently there are five distributions available to install, so if we have a
64 GB (~= 60 GiB) SD card on `/dev/mmcblk0` and we run the following command:

```shell
./mkimage.sh -p 11GiB /dev/mmcblk0
```

Then we will obtain the following partition structure:

* `/dev/mmcblk0p1`: Bootloader (raw partition, 16 MiB)
* `/dev/mmcblk0p2`: Distro 1 (11 GiB)
* `/dev/mmcblk0p3`: Distro 2 (11 GiB)
* `/dev/mmcblk0p4`: Distro 3 (11 GiB)
* `/dev/mmcblk0p5`: Distro 4 (11 GiB)
* `/dev/mmcblk0p6`: Distro 5 (11 GiB)
* `/dev/mmcblk0p7`: Shared partition for general data storage (4.6 GiB)


## Bootloaders

The `mkimage.sh` script will automatically install Megi's custom U-Boot image
([source](https://xff.cz/git/u-boot/tree/?h=ppp-2023.07),
[config+build](https://xff.cz/kernels/bootloaders-2024.04/ppp.tar.gz)) to
the SD card. This image **must** be executed by the PinePhone Pro during boot in
order to display the graphical distribution selector.

The easiest way to ensure this happens is to install
[rk2aw](https://xnux.eu/rk2aw/) to the phone's SPI flash; then it will boot the
correct U-Boot image regardless of what is installed on the eMMC.

Alternatively, you can hold the **RE** button while you power up your phone; this
will temporarily disable the eMMC and SPI flash in order force a boot from
the U-Boot image on the SD card.

## Install to eMMC

If you have access to the phone's eMMC as a `/dev/[DEVICE]` node, you can use
the same method (as described above) to install the multi-distribution image
to the eMMC.

You will still need to make sure that the correct bootloader is being executed.
You can either do this by installing rk2aw to the SPI flash and **not**
inserting a bootable SD card, or by zeroing out your SPI flash.

## Reinstall a single distro

This functionality exists mainly for development purposes, e.g. adding support
for new distributions. Make sure that:

1. You are still in the project's root directory, and
2. `/dev/[DEVICE]` has already been imaged via `mkimage.sh`.

Run the following command, replacing `<distro>` with the name of one of the
subdirectories in `distros/`.

```shell
util/installdistro.sh distros/<distro> /dev/[DEVICE]
```

## Building a release image

If you want to create an SD card image that can be written directly to an SD
card (without having to re-run this script), you can do so as follows:

```shell
fallocate -l 64000000000 sdcard.img # e.g. for a 64GB SD card
./download.sh                       # if you haven't downloaded the images already
./mkimage.sh -p 11GiB sdcard.img
```

It should be noted that this approach will take up 64GB of space on your hard
drive _and_ slow down the final copy to the SD card:

```shell
sudo dd if=sdcard.img of=/dev/[DEVICE] bs=1M conv=fsync
```

## See also

* [Megi's multi-distro image script for the PinePhone (non-Pro)](https://xff.cz/git/pinephone-multi-boot)
* [Manual installation guide](https://pine64.org/documentation/PinePhone_Pro/Software/Multi-distribution_image/)
