---
title: Arch linux 安裝 - Module efivars not found
date: 2015-10-22 23:41:45
tags: [Arch]
categories: [Debug]
---
好幾次安裝 Arch linux 設定 EFI 開機時
按照教學先在隨身碟載入 efivars 的模組
```bash
modprobe efivars
```
<!--more-->
會出錯：
```bash
Module efivars not found
```

這會影響後面安裝 grub 的時候會失敗
多嘗試幾次之後發現了兩種情形會導致這樣：
1. 開機的時候沒有用 UEFI 模式進入 live usb
如果你發現外面沒有/sys/firmware/efi/efivars那可能就是這個原因了
只要重開機 在開機選單選擇是用UEFI進入隨身碟就可以了
2. arch-chroot裏面沒有/sys/firmware/efi/efivars
這有兩個解法：
一個是arch-chroot進去之後安裝efibootmgr
以掛載在/mnt為例：
```bash
arch-chroot /mnt
pacman -S efibootmgr
```
還有一個方法是在外面
```bash
mount --bind /sys/firmware/efi/efivars /mnt/sys/firmware/efi/efivars
```
兩種方法都可以試試看，我自己是比較喜歡直接安裝efibootmgr
解決了之後
安裝 grub 時應該就不會有問題了
