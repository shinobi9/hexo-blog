---
title: steamdeck 挂载 NAS
date: 2023-03-29 19:21:18
tags: [nfs,steamdeck,nas]
---
### steamdeck 挂载 NAS 

先安装nfs依赖,需要先关闭steamos的读写保护,装完了再开启

```shell
sudo pacman -Syu
sudo steamos-readonly disable
sudo pacman -S sshfs
sudo pacman-key --init
sudo pacman-key --populate
sudo pacman -S sshfs
sudo pacman -S nfs-utils
sudo steamos-readonly enable
```

然后再修改`/etc/fstab`,添加路径映射

```shell
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# SteamOS partitions
#/dev/disk/by-partsets/self/rootfs /       ext4    defaults                0       1
#/dev/disk/by-partsets/self/var    /var    ext4    defaults                0       2
/dev/disk/by-partsets/self/efi    /efi                                  vfat    defaults,nofail,umask=0077,x-systemd.automount,x-systemd.idle-timeout=1min 0       2
/dev/disk/by-partsets/shared/esp  /esp                                  vfat    defaults,nofail,umask=0077,x-systemd.automount,x-systemd.idle-timeout=1min 0       2
/dev/disk/by-partsets/shared/home /home                                 ext4    defaults,nofail,x-systemd.growfs 0       2
<你的NAS IP>:<你的NAS 路径>         <映射到steamdeck的路径,我的是/run/media/nas>  nfs     defaults,soft,timeo=900,x-systemd.automount,vers=4.1,_netdev 0 0
```

最后
```shell
sudo mount -a
```

一般开机有网络会自动连上，没有网络再输入一遍`sudo mount -a` 即可。每次更新`steamos` 可能会失效，需要重新安装依赖，具体关注 [963](https://github.com/ValveSoftware/SteamOS/issues/963)