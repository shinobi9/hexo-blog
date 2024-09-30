---
title: 使用traefik管理内网服务
date: 2024-09-30 16:02:22
tags: [docker,traefik]
---


在自己内网部署了几十个服务之后，我才后知后觉的发现管理这么多web页面是非常困难的事

使用 [homepage](https://github.com/gethomepage/homepage) 可以很好的归纳整理内网的服务到一个统一的页面。但是每个服务一个端口，还得保证他们不乱撞（脑子里维护一个端口数据库），某些内网服务可能还要暴露到外网。

`traefik` 是专为容器而生，使用服务发现来实现网关，非常舒服，比起 `nginx` 不太需要写太多的配置文件。

首先 `docker network create traefik-proxy --attachable` 创建一个网络可以让 `traefik`的流量到达别的容器

`traefik` 服务：
```yaml
version: '3'

services:
  reverse-proxy: 
    image: traefik:v3.1.4
    command: 
      - --api.dashboard=true
      - --api.insecure=true
      - --providers.docker
      - --providers.docker.exposedByDefault=false
      - --providers.docker.network=traefik
      - --providers.file.directory=/traefik/providers
      - --entryPoints.web.address=:80
      - --entryPoints.websecure.address=:443
    volumes:
    # 定义 tls 文件
      - ./providers:/traefik/providers
    # 存放证书
      - ./certs:/certs
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "80:80"
      - "443:443"
    networks:
      - traefik-proxy
    #   下面是让traefik 自身的页面被反代的配置
    # labels:
    #   - traefik.enable=true
    #   - traefik.http.routers.traefik.rule=Host(`traefik.shinobi9.server`)
    #   - traefik.http.services.reverse-proxy.loadbalancer.server.port=8080
    #   - traefik.http.routers.traefik.tls=true
    #   - traefik.http.routers.traefik.entrypoints=websecure
networks:
  traefik-proxy:
    external: true
```

`./providers/providers` 文件里主要定义证书：
```yaml
tls:
  certificates:
    - certFile: /certs/shinobi9.server.cer
      keyFile: /certs/shinobi9.server.key
```

要被反向代理的服务，一个例子：
```yaml
services:
  kuma:
    image: louislam/uptime-kuma:1.23.13
    volumes:
      - ./data:/app/data
    # 因为通过 `traefik-proxy` 网络来通信，故不需要在暴露给0.0.0.0
    # ports:
    #   - 9901:3001
    labels:
      - traefik.enable=true
    #   配置域名
      - traefik.http.routers.kuma.rule=Host(`uptime.shinobi9.server`)
    #   如果容器内服务的端口是80，可以省略下面这条
      - traefik.http.services.kuma.loadbalancer.server.port=3001
      - traefik.http.routers.kuma.tls=true
      - traefik.http.routers.kuma.entrypoints=websecure
    networks:
      - traefik-proxy
networks:
  traefik-proxy:
    external: true
```

内网域名也是一个问题，有些部署的web应用可能使用 `webrtc` 之类的API强制要求https，所以内网可能还要自签证书。

关于这部分，网上查了很多资料，最后看了一篇写的非常详尽的 [博客](https://www.tangyuecan.com/2021/12/17/%E5%B1%80%E5%9F%9F%E7%BD%91%E5%86%85%E6%90%AD%E5%BB%BA%E6%B5%8F%E8%A7%88%E5%99%A8%E5%8F%AF%E4%BF%A1%E4%BB%BB%E7%9A%84ssl%E8%AF%81%E4%B9%A6/) 。

再有就是如何让 `DNS` 解析自己的域名，比如内网部署一个 `DNS` 服务器，考虑到炸了之后网络容易瘫痪，所以选择了更轻量的两种替代。

本机Host文件修改，有个非常好用的工具就是 [SwitchHosts](https://github.com/oldj/SwitchHosts) 。

还有就是路由器Host文件修改了，比起本机修改，手机等别的设备就也能享受域名服务了，这个根据路由器品牌和操作系统八仙过海了。