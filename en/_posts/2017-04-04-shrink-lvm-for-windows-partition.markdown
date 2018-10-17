---
published: true
title: Shrink LVM volume for Windows partition
layout: post
---
In order to release space from an LVM volume to finally get a free physical partition requires the following steps:

1. Resize File Sytem in the LVM volume
2. Resize Physical Volume
3. Create a NTFS partition

Also, root access is necessary


## Resize File Sytem in the LVM volume
``` shell
# lvresize --resizefs --verbose -L -30G /dev/machine_vg/root
```

## Resize Physical Volume
``` shell
# pvresize --setphysicalvolumesize [the size of your volume - 30G] /dev/sdaX
```
If you get an error as:
/dev/sdaX: cannot resize to X extents as later ones are allocated.
It means there's another volume between the free space and the end. To see it:
``` shell
# pvs -v --segments /dev/sdaX
```
To move the free space towards the end, grab the last column from the output for the volume to move and use it in pvmove:
``` shell
# pvmove --alloc anywhere /dev/sdaX:xxxxx-xxxx
```


## Create a NTFS partition
``` shell
# fdisk
n
c
1-n [partition number, probably sdaX+1]
L
87
# mkfs.ntfs /dev/sdaX
```
