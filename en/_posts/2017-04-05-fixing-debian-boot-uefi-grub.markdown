---
published: true
title: Fixing Debian UEFI boot manager with Debian Live
layout: post
description: The goal of this note is to fix the UEFI Boot Manager (located in the NVRAM) for a Debian installation, by using a Debian Live image to mount a broken system via chroot and then reinstall grub-efi. This will recreate the boot loader for grub2-efi in the EFI System Partition (as /boot/efi) and add an entry for it in the boot manager. 
---

**Important note:** If you are new to UEFI please read this 30min article to understand what's happening: [UEFI boot: how does that actually work, then?](https://www.happyassassin.net/2014/01/25/uefi-boot-how-does-that-actually-work-then/)

The goal of this note is to fix the UEFI Boot Manager (located in the NVRAM) for a Debian installation, by using a Debian Live image to mount a broken system via chroot and then reinstall grub-efi. This will recreate the boot loader for grub2-efi in the EFI System Partition (as /boot/efi) and add an entry for it in the boot manager. 
A possible scenario to execute this workaround is when your current Linux installation dissapeared from the UEFI Boot Manager. The instructions here were executed on a Debian Jessie box.

Steps summary:

1. Download Debian Live 
2. Burn it to an USB drive
3. Fix UEFI on the USB drive
4. Boot Debian Live
5. Verify Debian Live was loaded with UEFI 
6. Review devices location and current configuration
7. Mount broken system (via chroot)
8. Reinstall grub-efi
9. Verify configuration
10. Logout from chroot and reboot

## Download Debian Live ##
Download it from [Debian Live](https://www.debian.org/CD/live/).

***Important:*** At this point, UEFI support exists only in Debian's installation images. The Live images do not have support for UEFI boot.

## Burn it to an USB drive ##  
As stated in the initial recommended reading, UEFI works best with FAT32 and GPT partition table. However, Debian Live does not support it yet
so any method could be chosen. However, be sure to format the USB drive as FAT32. Recommended tools: 

* For Debian Linux, [Ether](https://github.com/resin-io/etcher). Note that in the instructions use the retired site pgp.mit.edu, therefore use hkp://pool.sks-keyservers.net instead.
* For Windows, [Rufus](https://rufus.akeo.ie). It has the advantage of clearly allow to select the Partition scheme and system type.

## Fix UEFI on the USB drive ##
1. Dowload Shell.efi from [UEFI Shell](https://github.com/tianocore/edk2/tree/master/ShellBinPkg/UefiShell/X64).
2. On the USB drive create a EFI folder and a BOOT sub folder, then copy Shell.efi to EFI\BOOT\ and rename it to Bootx64.efi.
3. Using a text editor, create a file called liveboot.nsh on the root of the USB drive and paste the following into the file: 

``` shell
live\vmlinuz initrd=live\initrd.img append boot=live components
```

## Boot Debian Live ##
Reboot the computer and hit F8 or F12 to bring the UEFI Boot Menu, and select your USB Drive. If it is shown more than one time, ensure to select the one that mentions UEFI.
After UEFI Shell loads, type liveboot.nsh and press Enter. 

## Verify Debian Live was loaded with UEFI
When Debian Live finishes loading, verify it it was loaded with UEFI with two basic methods:

1. Checking dmesg output you should see many lines related to EFI or UEFI.

``` shell
$ dmesg | grep -i efi 
```
2. Check the EFI vars are correctly loaded

``` shell
$ ls -l /sys/firmware/efi | grep vars
```

## Review devices location and current configuration ##
1. Identify the root, boot and EFI partitions from the broken system.
Use fdisk -l to survey the current partition. EFI normally goes into /dev/sdX1. Mount one by one trying to identify which is the correct one. 
As a reference, /boot would normally look like:

``` shell
$ ls -l /boot
total 21581
-rw-r--r-- 1 root root   157721 Oct 19 03:45 config-3.16.0-4-amd64
drwxr-xr-x 5 root root     1024 Apr  5 15:26 grub
-rw-r--r-- 1 root root 16023698 Dec  5 09:50 initrd.img-3.16.0-4-amd64
-rw-r--r-- 1 root root  2679264 Oct 19 03:45 System.map-3.16.0-4-amd64
-rw-r--r-- 1 root root  3126448 Oct 19 03:41 vmlinuz-3.16.0-4-amd64
```
**Note:** In the case of lvm volumes, you might need to download the lvm2 package and expose the volumes before mounting:

``` shell
# apt-get install lvm2
# vgscan
# vgchange -a -y
# mount /dev/volumegroup/logicalvolume /mnt
```
2. Download efibootmgr and review current UEFI Boot Manager configuration

``` shell
# apt-get install efibootmgr
# efibootmgr -v
```

## Mount broken system (via chroot) ##
Mounting another system via chroot is the usual procedure to recover broken system's. Once the chroot comand is issues, Debian Live will treat the broken system's "/" (root) as its own. Commands run in a chroot environment will affect the broken systems filesystems and not those of the Debian Live.
1. Mount root partition (for example an lvm)

``` shell
# mount /dev/volumegroup/logicalvolume /mnt
```
2. Mount boot partition (F.e. in drive sda)

``` shell
# mount /dev/sda2 /mnt/boot
```
3. Mount the EFI System Partition (usually in /dev/sda1)

``` shell
# mount /dev/sda1 /mnt/boot/efi
```
4. Mount the critical virtual filesystems with the following single command:

``` shell
# for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt$i; done
```
5. Chroot to your normal (and broken) system device

``` shell
# chroot /mnt
```
**Important:** From now on your changes will affect your normal system.

## Reinstall grub-efi ##
Download and Install grub-efi package, set the debian bootloader and create a grub config file based on your disk partitioning schema
1. Install grub-efi package

``` shell
# apt-get install --reinstall grub-efi
```
2. Put the debian bootloader in /boot/efi and create an appropiate entry in the computer's UEFI Boot Manager

``` shell
# grub-install /dev/sda
```
3. Re create a grub config file based on your disk partitioning schema

``` shell
# update-grub
```

## Verify configuration ##
1. Bootloader exists

``` shell
# file /boot/efi/EFI/debian/grubx64.efi 
/boot/efi/EFI/debian/grubx64.efi: PE32+ executable (EFI application) x86-64 
(stripped to external PDB), for MS Windows
```
2. NVRAM entry was properly created 

``` shell
# efibootmgr -v | grep debian
```

## Logout from chroot and reboot ##
Type ctrl+d or exit to quit the chroot session, and reboot.
