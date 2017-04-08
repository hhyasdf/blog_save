---
title: Linux/Windows多功能启动盘安装
date: 2017-04-08 23:29:43
categories: Linux
tags: 
	- Linux
	- OS
---

由于一些（不可描述的）原因，一直在使用 Linux/Windows 双系统，十足地体会到了每次连续装机都免不了将 Linux 启动盘刷成 Windows 启动盘再刷回来的苦逼。再加上希望启动和存储的功能隔离开。所以参考网上的一些资料和博客，做了一个不算很完美的多功能启动盘。

由于 Windows 不支持u盘多分区加载，默认只会加载u盘第一分区，所以将第一分区作为存储分区。另外两个分区分别是 Linux（arch）启动分区和 Windows 启动分区。

又因为网上对于直接引导 windows 原始镜像成功可能的怀疑和否定，以及各种不可描述的 winpe iso 无法下载，所以打算直接在 windows 下利用一键制作启动盘软件先行将 windows 启动分区做好。

机缘巧合之下了解到了 u启动 这个工具，好处在于它会自动将u盘分为存储和启动两个部分，于是乎果断用之。。。

<!-- more -->

制作完 windows 启动分区后切入 Linux 系统。

先用 fdisk 修改分区表，增加两个分区，并格式化为 fat32 filesystem：

```shell
username@host>> sudo fdisk /dev/sdc
username@host>> sudo mkfs.vfat /dev/sdc1
username@host>> sudo mkfs.vfat /dev/sdc2
```

如果有需要可以用 mtools 给分区加上卷标

而后我们需要通过 blkid 命令知道分区的 uuid（我们将 /dev/sdc2 作为 Linux 启动分区）

Linux 启动分区用 grub2 引导，我们使用 efi 模式安装grub （既支持 legacy 又支持 EFI 的方法貌似有点魔性。。。失败了一次之后没有再研究）：

```shell
sudo mount /dev/sdc2 /mnt
username@host>> sudo grub-install --target=x86_64-efi --efi-directory=/mnt --boot-directory=/mnt/boot --removable
```

/mnt 是 Linux 启动分区的挂载点

而后我们需要（创建）修改分区中的 /boot/grub/grub.cfg 配置文件：

```shell
menuentry "Archlinux-x86_64.iso" --class iso {
   set isofile="/archlinux-2017.01.01-dual.iso"
 #  loopback loop (hd0,msdos2)$isofile
   loopback loop $isofile
   linux (loop)/arch/boot/x86_64/vmlinuz archisolabel=archinstall img_dev=/dev/disk/by-uuid/1489-E7DE img_loop=$isofile earlymodules=loop
   initrd (loop)/arch/boot/x86_64/archiso.img
 }
 
 
menuentry "Archlinux-i686.iso" --class iso {
   set isofile="/archlinux-2017.01.01-dual.iso"
 #  loopback loop (hd0,msdos2)$isofile
   loopback loop $isofile
   linux (loop)/arch/boot/i686/vmlinuz archisolabel=archinstall img_dev=/dev/disk/by-uuid/1489-E7DE img_loop=$isofile earlymodules=loop
   initrd (loop)/arch/boot/i686/archiso.img
 }
```

  这里我们使用的是直接从 iso 镜像中读取启动文件的方式。1489-E7DE 是 /dev/sdc2 分区的 uuid （没有指定 root 变量的话，grub 默认根目录是当前分区根目录），archinstall 是分区的卷标（貌似这个值可以任意，只要不和已存在的分区重复）。而后将 arch 镜像拷入 Linux启动分区的根目录下就完成了。（需要刷 windows 只需要将镜像拷入存储分区就好）其他 Linux 发行版相差不大。

这样虽然在 BIOS 里切启动项时会有两个u盘启动项（看起来有点蹩脚），并且 Linux 启动项只支持 efi 启动（windows 是支持 efi 和 legacy 的），但好在简单，流程较快，并且也能够满足存储、多系统安装维护盘的要求。