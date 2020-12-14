---
title: hello
date: 2020-12-07 10:18:03
tags: [misc,hello world]
---

## hello!

<!-- more -->
这个博客由 github + drone 完成
使用了两个仓库
`github.com/shinobi9/hexo-blog(private)`  `github.com/shinobi9/shinobi9.github.io`

`hexo-blog` 推送会用webhook trigger `drone.io` , 
执行 `hexo deploy`,部署到 <https://shinobi9.github.io>, 
因为配置了 CNAME, 故会跳转到 最终的配置的域名 <https://blog.shinobi9.cyou>