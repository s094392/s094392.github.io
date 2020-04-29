---
title: anaconda 與 yaourt 衝突
date: 2015-10-22 23:33:30
tags: [Arch]
categories: [Debug]
---
在重灌這台電腦之前，以及重灌之後
使用 yaourt 安裝東西的時候每次都遇到
這種在 AUR 找不到 package 的問題
```bash
yaourt -S xxx
==> Downloading xxx PKGBUILD from AUR...
==> ERROR: xxx not found in AUR.
```
<!--more-->

所以就什麼都得自己 build @@
後來陰錯陽差發現原來是 anaconda 出了問題
因為我用 pyenv 將 global 設成 anaconda

只要設回成 system 或是 3.4.3
就可以正常使用 yaourt 了
