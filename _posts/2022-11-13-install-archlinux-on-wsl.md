---
layout: post
title: Install Archlinux on WSL
date: 2022-11-13 17:16 +0800
author: aold619
description:
image:
category: [Tutorial]
tags: [tutorial, wsl, linux]
published: true
sitemap: false
---

A brief guide to install Arch linux or any other distros linux on Windows WSL.

1. download and unzip [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline/releases)
2. download [archlinux-bootstrap](https://archlinux.org/download/)
3. powershell cmd: `LxRunOffline i -n <distro-name> -f <archlinux-bootstrap.tar.gz> -d <install_DIR> -r root.x86_64` (maybe need to upgrade wsl to wsl2: `wsl --set-version <distro name> 2`)
4. add admin user `useradd -m -G users,wheel <username>`
5. check vm router `wsl -- ifconfig eth0` and check nameserver config `/etc/resolv.conf`
6. enable systemd `[boot] systemd = true` `/etc/wsl.conf`
7. set default user `[user] default = username` in `/etc/wsl.conf`
8. set mirror-list `Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch` in `/etc/pacman.d/mirrorlist`
9. set proxy `export ALL_PROXY="http://xxx.xxx.xxx.xxx:xxxx"`
10. pacman-key init `pacman-key --init && pacman-key --populate`
11. install dev packages if you need `yes | pacman -S archlinux-keyring base-devel inetutils git wget`

> Other details or cmd please check [WSL Doc](https://learn.microsoft.com/en-us/windows/wsl/)
{: .prompt-info }
