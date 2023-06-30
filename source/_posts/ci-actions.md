---
title: 从Drone 迁移到 Gitea Actions
date: 2023-06-15 14:59:34
tags: [ci,gitea,gitea actions,drone]
---

先来说说代码托管服务器的一般问题和DevOps问题：
- `Github`: 
  因为GFW的原因，常年访问速度较慢。
  虽然标榜自己是开源平台，但是仍做出了不通知直接删除某些国家和地区repo或是封禁某些国家和地区的用户的行为。
  因其有实际上垄断地位，说其是帝国主义在网络上的延伸不为过。

  不过它的CI工具 `Github Actions` 配合 `Github Pages` 服务可以很方便地自动化部署一些静态的web项目。
- `Gitee`: 
  也是因为GFW以及更早的支持私有仓库，在国内积累了很大的生态。
  但是某段时间为了应付审查将所有的仓库强行变更为私有，只能通过类似备案或是申请审查来变回开源仓库。
  后面甚至还出现了审查代码内容的行为，让人讲话，天不会塌下来，再就是码云下载代码还要登录。

  因为过多原因，在个人开发时完全放弃了，新出的流水线功能还未尝试。

---

在个人的代码的托管上，我最后选择了 `Gitea`，它是 `Gogs` 的分支。

在选型时，`GitLab` 因为和 `GitLab CI` 绑定，不好迁移。

最早在使用 `Gitea` 时，它还没有CI功能，但是胜在比 `GitLab` 和 `Jenkins` 轻量级。

当时在公司服务器部署 `GitLab` ，经常吃掉 4~5G 内存，而 `Gitea` 可能就 50M ，虽然现在加了很多功能，

内存也才到200 ~ 300M，完全可以接受。

因为 `Gitea` 没有 CI 功能，我又给它找了个伴侣————`Drone CI`，说实话看到它第一眼确实被惊艳到了，它使用容器作为每一道工序的工具，拿起整个`Docker`的生态作为武器库，使得它的透明性和 DIY 能力非常强。

```yaml
kind: pipeline
type: docker
name: hexo-blog-ci

trigger:
  branch:
    - main
  event:
    - push
    - custom

clone:
  depth: 1

steps:
  - name: build
    image: node:18.15-alpine3.16
    pull: if-not-exists
    commands:
      - yarn
      - yarn build

  - name: publish
    image: plugins/gh-pages:1.3.2
    pull: if-not-exists
    environment:
      DRONE_REMOTE_URL: "git@github.com:shinobi9/hexo-blog.git"
    settings:
      pages_directory: public
      ssh_key:
        from_secret: SSH_KEY
```

上面曾是我博客的部署方式，`.drone.yml`的配置文件的内容。

`Drone CI` 官方的容器 一般以 plugins 开头，有点抢注了厉害的id的感觉。

写 `.drone.yml` 的感觉非常享受，尤其还是盲写不报错的时候，有点像写 `Kotlin` 的感觉，它也是借助 `Java` 生态并且提供更简介的语法，从这方面来讲，确实两者很像。

`Drone` 官方也提供了云服务，通过 [https://cloud.drone.io/](https://cloud.drone.io/) 来访问，我（部署在`github`上的博客）当时也是部署在这上。

不知道什么时候，想起更新博客的时候发现，托管在云服务上的CI被挂起了，也没找到原因（因为我很少写博客，理应不会占用太多免费配额）。

如果是本地的 `Gitea` 配合 本地的 `Drone` 确实是很好的组合。所以瑕不掩瑜，我还是使用了很久的 `Drone`，即使 `Github` 一直发邮件给我骚扰我用 `github actions` 我也没去看。

直到某天我打开我本地的 `Gitea` 看到了版本号的巨大变化，我迫不及待更新了 1.19 ，顺便去看了一眼changelog，才发现早在几个月前，`Gitea` 推出了自己的CI ———— `Gitea Actions`。

虽然我不得不承认 `Gitea` 和 `Drone` 的组合很厉害，但是我还是忍不住去看了  `Gitea Actions` 的文档。

下面是 `Gitea Actions`的使用： 省流！

```shell 
# 在 gitea 中 开启 gitea actions 功能
cat >> app.ini << EOF
[actions]
ENABLED=true
EOF
```

关于 runner 搞一个 `config.yaml` 
```yaml
cache:
  enabled: true
  dir: <cache dir>
  # Use the LAN IP obtained in step 1
  host: <act_runner ip>
  # Use the port number obtained in step 2
  port: <act_runner port. eg 5500>
```

然后用 `docker-compose.yml`

```yaml
version: '3'

services:
  gitea_runners:
    image: gitea/act_runner:nightly
    ports:
      - 5500:5500
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
      - ./config.yaml:/config.yaml
    restart: unless-stopped
    environment:
      CONFIG_FILE: /config.yaml
      GITEA_INSTANCE_URL: <gitea url>
      GITEA_RUNNER_REGISTRATION_TOKEN: <token>
  #    GITEA_RUNNER_NAME:
  #   GITEA_RUNNER_LABELS:
```

`Gitea` 默认所有项目都不开启 `Gitea Actions` ，设置-高级设置 里打开即可。

`Github Actions` 和 `Drone CI` 一样借助于生态， 不同的是 `Github Actions`使用的生态就是它的本身，用户的仓库。

而`Gitea Actions` 的语法大部分兼容 `Github Actions` ，而原理是 借助于 [nektos/act](https://github.com/nektos/act) （它的分支），

它可以让用户可以在本地部署 `Github Actions` 的执行器。

下面给出它的例子 `.gitea/workflows/ci.yaml`

```yaml
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: checkout
      uses: actions/checkout@v2

    - name: cache node_modules
      uses: https://github.com/actions/cache@v2
      with:
        path: |
          node_modules
          ~/.npm
        key: <...key>
        restore-keys: <...restore key>

    - name: use nodejs
      uses: actions/setup-node@v1
      with:
        node-version: '18.16.0'

    - name: install dependencies and build
      run: |
        npm install
        npm run build
      
    - name: install rsync
      run: apt-get update && apt-get install rsync -y 

    - name: upload and restart
      uses: https://github.com/easingthemes/ssh-deploy@main
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SHINOBI9_DEPLOY_KEY }}
        ARGS: "-rlgoDzvc -i --delete"
        SOURCE: ./build
        REMOTE_HOST: ${{ secrets.DEPLOY_SERVER }}
        REMOTE_USER: ${{ secrets.DEPLOY_USER }}
        TARGET: ${{ secrets.DEPLOY_TARGET }}
        # EXCLUDE: "/dist/, /node_modules/"
        SCRIPT_AFTER: |
          cd ${{ secrets.DEPLOY_TARGET }}
          docker-compose restart web      
```

我隐去了部分无关内容，可以看的出虽然看上去比起`Drone CI`来说没有那么简洁，但是同为 Yaml，也说不上多复杂，

写过几个例子之后，感觉不如`Drone CI`。。。。简洁，但是一想到不仅可以从`github`上的 [market](https://github.com/marketplace?type=actions) 里~~~毛~~~别人的 actions。还可以直接拿repo来用。

不说了，去做移植工作了。顺便贴下这个博客移植之后的actions文件。

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18.16.0

      - name: Install Yarn
        run: npm i -g yarn
      
      - name: Install Dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: |
          yarn install --frozen-lockfile

      - name: Deploy
        id: deploy
        uses: sma11black/hexo-action@v1.0.4
        with:
          deploy_key: ${{ secrets.DEPLOY_KEY }}
          branch: gh-pages
      - name: Get the output
        run: |
          echo "${{ steps.deploy.outputs.notify }}"
```