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

1. download and unzip LxRunOffline (https://github.com/DDoSolitary/LxRunOffline/releases)
2. download archlinux-bootstrap (https://archlinux.org/download/)
3. powershell cmd: ```LxRunOffline i -n <distro-name> -f <archlinux-bootstrap.tar.gz> -d <install_DIR> -r root.x86_64``` (maybe need to upgrade wsl to wsl2: ```wsl --set-version <distro name> 2```)
4. useradd -m -G users,wheel <username>
5. set nameserver ```nameserver xxx.xxx.xxx.x``` in ```/etc/resolv.conf```
6. enable systemd ```[boot] systemd=true``` ```/etc/wsl.conf```
7. set default user ```[user] default=username``` in ```/etc/wsl.conf```
8. set mirror-list ```Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch``` in /etc/pacman.d/mirrorlist
9. set proxy ```export ALL_PROXY="http://xxx.xxx.xxx.xxx:xxxx"
10. ```pacman-key --init && pacman-key --populate``` update && install archlinux-keyring base-devel inetutils git wget
