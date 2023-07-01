---
title: headscale 与 tailscale 
date: 2023-07-01 21:04:40
tags: [headscale,tailscale,vpn]
---

上班想访问家中的设备，因为设备比较多，暴露端口比较麻烦，动态ip还得弄ddns，查了一下 `zerotier`，`tailscale` 在这个领域比较热门。

刚开始用了很长一段时间的zerotier，配置比较简单，所以感觉还可以。

不知道什么时候开始，经常出现无法握手的情况，添加了自己搭的 `moon` 服务器也是好一阵坏一阵，所以试着换到 `tailscale`。

`tailscale` 是一个客户端开源，服务端不开源的软件，而 `headscale` 是一个欧洲航天局的开发者开发的一款开源的 `tailscale` 服务端。

下面是具体的配置

本地：

`headscale-ui/docker-compose.yml`
```
version: '3'
services:
  ui:
    image: "ghcr.io/gurucomputing/headscale-ui:latest"
    restart: unless-stopped
    container_name: headscale-ui
  nginx:
    image: "nginx:1.25.0"
    restart: always
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 3000:80
```     
`nginx/default.conf` 

```
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location /web/ {
        proxy_pass http://ui:80/web/;
    }
    location /api/ {
        proxy_pass http://<服务器ip>:5488/api/;
    }
}
```

---
服务器：

`headscale/docker-compose.yml`
```
version: '3'
services:
  headscale:
    image: headscale/headscale:0.22.3-debug
    container_name: headscale
    volumes:
      - ./headscale/config:/etc/headscale
      - ./headscale/data/data:/var/lib/headscale
    ports:
      - 5488:5488
    command: headscale serve
    restart: unless-stopped
```
`headscale/config.yaml`

```
server_url: http://<服务器ip>:5488
listen_addr: 0.0.0.0:5488
private_key_path: /etc/headscale/private.key
noise:
  private_key_path: /etc/headscale/noise_private.key
db_type: sqlite3
db_path: /etc/headscale/db.sqlite
ip_prefixes:
  - 100.64.0.0/10
```

我的`headscale-ui`部署在本地，所以配置没有写在一起。

进入 `headscale-ui` 之后，设置里 `Headscale URL` 填入 `headscale-ui` 自己本身的路径即可，这里为了避免跨域，因为 `headscale-ui` 页面都有 `/web/` 前缀，且 `headscale` 的接口不支持跨域，所以在 `nginx` 中的配置文件写了那样的规则。

在 `Headscale API Key` 填入服务器上执行 

```
docker-compose exec headscale headscale apikeys create
```
返回的结果，默认两个月会过期。

至于客户端的配置，windows的客户端需要修改注册表，具体什么缓存问题可以直接看官方给出的[文档](https://github.com/juanfont/headscale/blob/main/docs/windows-client.md)。文档里也给出了别的平台的客户端如何**不去连接** `tailscale` 的服务器。

要实现在异地使用 `192.168.x.y` 访问家里设备该如何做？

软路由/路由上一般也会给出 `tailscale` 插件，我用的固件只提供了连接官方服务器的插件，所以我使用下面的命令行。

```
tailscale up --accept-routes \
 --accept-dns=false --snat-subnet-routes=false \
 --advertise-routes=192.168.x.0/24 \
 --login-server=http://<服务器>:5488
```

`192.168.x.0/24` 是局域网的网段，我的是 `192.168.0.0`。

执行完需要去 `headscale-ui` 里启用一下 `Device Routes` ，客户端中打印路由表有额外的关于 `192.168.x.0` 就证明下发成功了。