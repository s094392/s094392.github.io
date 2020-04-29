---
title: Arch Linux 安裝小筆記
date: 2017-11-28 03:14:32
tags: [Arch]
categories: [Notes]
---
最近幫一個朋友裝了 Arch 想說順便紀錄一下安裝步驟，希望沒有寫錯或遺漏的 XD
他的電腦是 Asus N580VD，跟 Linux 相容的很好，推推
## 事前準備
安裝之前當然要下載安裝媒體啦，可以去[官網](https://www.archlinux.org/download/)下載然後燒到 usb 裡面，記得選 Taiwan 的 mirror 下載比較快
然後用 usb 開機就可以開始安裝了
然後建議安裝的時候準備另一台電腦或是手機可以邊裝邊查
<!--more-->

## 連上網路
因為 Arch Linux 是要連上網路才能安裝的，所以要連一下
建議是插上網路線，或是用手機連上去再用 usb 分享給電腦，或是有些比較幸運的話可以用 `wifi-menu`
總之，插上之後要先抓一下 ip
```
$ dhcpcd
```
如果它已經在執行的話就要先打 `dhcpcd -x` 關掉它
這樣就可以連上網路了
可以 ping 一下 8.8.8.8 試試看
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=59 time=2.45 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=59 time=2.47 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=59 time=2.42 ms
```
出現類似的東西應該就是連上了
## 切割硬碟
如果有電腦裡面已經有 Windows 的話，要先去磁碟管理割出一些空間給 Linux 使用
通常使用 Linux 會割出
* EFI （開機用通常已經有了，假設他在 /dev/sda1）
* Swap (dev/sda2)
* Root (dev/sda3)
* Home (dev/sda4)

把 Home 獨立出來的用意是重灌的時候比較方便的樣子，大小的話 EFI 2~300M，Root 大概 3, 40G 就可以了
使用 gdisk：
這裡 `/dev/sdx` 的 x 要看你要灌在哪個硬碟 可以先用 `fdisk -l` 看硬碟的狀況
```
$ gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):
```
然後打個 ? 看看他要怎麼用

```
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu
```
所以新增磁區是用 n
接著它會問你 First sector、Last sector 跟 Hex code
First sector 按 Enter 就好
Last sector 輸入你要的磁區大小
Hex code 要看你割什麼 按 L 可以看表
```
0700 Microsoft basic data  0c01 Microsoft reserved    2700 Windows RE
3000 ONIE boot             3001 ONIE config           3900 Plan 9
4100 PowerPC PReP boot     4200 Windows LDM data      4201 Windows LDM metadata
4202 Windows Storage Spac  7501 IBM GPFS              7f00 ChromeOS kernel
7f01 ChromeOS root         7f02 ChromeOS reserved     8200 Linux swap
8300 Linux filesystem      8301 Linux reserved        8302 Linux /home
8303 Linux x86 root (/)    8304 Linux x86-64 root (/  8305 Linux ARM64 root (/)
```
要注意的是
```
ef00 EFI System  8300 Linux filesystem 8200 Linux swap
```
這裡示範其中一個：
```
Command (? for help): n
Partition number (3-128, default 3):
First sector (34-1953525134, default = 878848000) or {+-}size{KMGTP}:
Last sector (878848000-1953525134, default = 1953525134) or {+-}size{KMGTP}: +16G
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8200
Changed type of partition to 'Linux filesystem'
```
這樣就會割好 16G 在第 3 磁區
然後依序割完 EFI Swap Root Home 就好了
割完可能會長這樣：
```
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          206847   100.0 MiB   EF00  EFI System
   2          206848        33761279   16.0 GiB    8200  Linux swap
   3       794961920       878847999   40.0 GiB    8300  Linux filesystem
   4       117647360       180561919   30.0 GiB    8300  Linux filesystem
```
## 格式化
Root Home 要格式化成 ext4
```
$ mkfs.ext4 /dev/sda3
$ mkfs.ext4 /dev/sda4
```
EFI 要格式化成 FAT32
```
$ mkfs.vfat -F32 /dev/sda1
```

## 掛載
然後把剛剛割完的磁區 mount 上去
```
$ mount /dev/sda3 /mnt
$ mkdir -p /mnt/boot/efi
$ mount /dev/sda1 /mnt/boot/efi
$ mkdir -p /mnt/home
$ mount /dev/sda4 /mnt/home
```
## 安裝
這裡建議先去改一下 mirrorlist 不然速度很慢
```
$　vim /etc/pacman.d/mirrorlist
```
然後把這個放到最前面
```
## Taiwan
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
```
接下來就安裝：
```
$ pacstrap /mnt base base-devel
```

然後把剛剛掛載的東西加到 `/etc/fstab` 裡面
```
$ genfstab -U -p /mnt >> /mnt/etc/fstab
```
chroot 進去然後建立 ramdisk
```
$ arch-chroot /mnt
$ mkinitcpio -p linux
```
## 安裝 grub
安裝 grub

```
$ pacman -S grub-efi-x86_64
$ pacman -S efibootmgr
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck
```
把 grub 設定存進去
```
$ grub-mkconfig -o /boot/grub/grub.cfg
```
設定 root 密碼
```
$ passwd
```
離開 arch-chroot 回到 usb 上
```
$ exit
```
然後 umount
```
$ umount /mnt/boot/efi
$ umount /mnt/home
$ umount /mnt
```
然後就可以重開機了
```
$ reboot
```
開機之後就可以安裝桌面了，我自己是用 gnome 下一篇在寫一下怎麼裝
