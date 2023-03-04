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

## Required Steps

1. download and unzip [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline/releases)
2. download [archlinux-bootstrap](https://archlinux.org/download/)
3. install in powershell:

   ```shell
   LxRunOffline i -n <distro-name> -f <archlinux-bootstrap.tar.gz> -d <install_DIR> -r root.x86_64
   ```

   maybe need to upgrade the new distro to wsl2:

   ```shell
   wsl.exe --set-version <distro name> 2
   ```

4. add an admin user

   ```shell
   useradd -m -G users,wheel <username>
   ```

5. enable systemd and set default user in `/etc/wsl.conf`

   ```ini
   [boot]
   systemd = true

   [user]
   default = username
   ```
   {: file="/etc/wsl.con"}

6. set pacman mirror-list in `/etc/pacman.d/mirrorlist`
7. pacman-key init

   ```shell
   pacman-key --init && pacman-key --populate
   ```

8. install dev packages if you need

   ```shell
   yes | pacman -S archlinux-keyring base-devel inetutils git wget
   ```

## Optional

1. add your admin user to sudoers, or add the wheel group if you want.

   ```shell
   %wheel ALL=(ALL:ALL) NOPASSWD: ALL
   ```
   {: file="/etc/sudoers.d/wheel-users"}

2. if you want to share the proxy with the Win, add config in to your profile file.

   ```shell
   export ALL_PROXY=xxx.xxx.xxx.xxx:xxxx
   ```

> Other details or cmd please check [WSL Doc](https://learn.microsoft.com/en-us/windows/wsl/)
{: .prompt-info }
