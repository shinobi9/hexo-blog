---
title: garuda linux 尝鲜
date: 2026-02-05 01:04:28
tags:
---

受够了 windows 的隐性性能负担以及复杂性，各种盲盒玄学
我最终还是切换到了linux， 一番选型下来， 没有选择较热门的 `cachyos` 而是选了 `garuda linux`

在一番备份 (每天下班抽点时间往 NAS 迁移文件) 后，我直接用 `rufus` 烧写镜像 安装了
说实话是有点赌的成分，毕竟是主力机，以前也不怎么爱折腾，万一折腾坏了，查google还得用手机

前段时间在备用手机上装的豆包确实帮到了我，拍照直接问，当然对于答案要审计一下。

安装完成后为了摆脱之前那种因为win本身闭源导致滑坡到安全性无所谓的情况，这次从头开始设置我的各种东西

首先是 文件挂载，因为 `smb` 授权麻烦又不是很安全
这台主力机直接使用了 sshfs 加配置一些参数配置私钥以及一些缓存之类的性能优化
办公软件通通丢进 NAS 中的 虚拟机win10，用 webdav + 独立用户挂载几个共享目录 用于多设备间传输文件
然后在 NAS 上关闭了 `smb` ，`nfs` 之类的协议

然后是连接到设备的方式，原先是一对 SSH key 拥有连接内网全部设备能力，到 切到了 `SSH agent`，现在 我使用了 [ttyd](https://github.com/tsl0922/ttyd) 这个项目来解决问题 在 NAS 上部署多个这个镜像重写Dockerfile后 分别特化成 SSH跳板机 以及 打包环境 以及 工具环境， 分别用于
- 存储一堆 key，使用 `SSH agent` 连接到内网各个设备
- 自带 go/rustc/gcc/g++ 环境方便自己编译一些库，暂时还负责了回家的通道，一个仅公司公钥登录的 SSHD 服务
- 一些 cli/tui 的实用工具，比如 [ytb-dlg](https://github.com/yt-dlg/yt-dlg) ，`ffmpeg` 之类的

然后把一些计算密集以及吃内存的较重的服务比如游戏服务器之类的迁移到了另一台`PVE LXC`上，甚至包括现在在写文章用的 [openvscode-server](https://github.com/gitpod-io/openvscode-server)，因为偶尔用 `opencode` 会有内存泄漏把我 NAS 中的虚拟机崩掉

暂时就想到这点..