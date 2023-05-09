---
title: pve lxc容器的NFS挂载问题
date: 2023-05-09 09:48:46
tags: [docker,nfs,virtualization]
---

折腾 `PVE` 的时候，遇到docker容器挂载nfs卷失败，具体表现是 `operation not commit`，单纯 `mount -vvv`出来的东西拿去搜不好使，关键词带搜索 `Proxmox` 之后 解决方案就多了，一种是在 `pve` 上先挂载 `nfs` 然后再 `bind mount` 到容器里去，具体参考 [这里](https://unix.stackexchange.com/questions/715278/mount-nfs-operation-not-permitted-in-proxmox-container) , 另外一种简单的就是在创建容器的时候取消勾选 `非特权容器` ，这里会同时取消 `嵌套` ，再之后 用命令行打开 `嵌套` 即可。

这里折腾了老半天主要是因为我因为问题在`docker`上