---
title: kubejs 
date: 2022-02-15 20:13:31 
tags: [game,minecraft,kubejs]
---

## 整合包利器: kubejs

<!-- more -->
最近拿家里的闲置电脑当服务器开了个minecraft服,在机械飞升整合包里发现了这个模组。

```javascript
// world/kubejs/server_scripts/scripts.js
// 修改配方需要 游戏内的 /reload 会卡ui线程

//简化代码
let MOD = (domain, id, x) => (x ? `${x}x ` : "") + (id.startsWith('#') ? '#' : "") + domain + ":" + id.replace('#', '')
let AE2 = (id, x) => MOD("appliedenergistics2", id, x)
let TE = (id, x) => MOD("thermal", id, x)
let AP = (id, x) => MOD("architects_palette", id, x)
let FD = (id, x) => MOD("farmersdelight", id, x)
let MC = (id, x) => MOD("minecraft", id, x)
let CH = (id, x) => MOD("chisel", id, x)

//移除匠魂 的弹簧鞋
onEvent('recipes', event => {
    log.push('Registering Recipes')
    event.remove({output: 'tconstruct:earth_slime_sling'})
    event.remove({output: 'tconstruct:sky_slime_sling'})
    event.remove({output: 'tconstruct:ichor_slime_sling'})
    event.remove({output: 'tconstruct:ender_slime_sling'})
})

// oak_log          橡木
// stripped_oak_log 去皮橡木
//  CR => create:   删除机械动力提供的橡木=>木板的配方,并且提供一个 获取农夫乐事的树皮的 合成(橡木 => 去皮橡木 + 树皮)
// 需要 kubejs create 支持( kubejs 扩展
onEvent('recipes', event => {
    event.remove({id: CR("cutting/oak_log")})
    event.recipes.createCutting([MC("stripped_oak_log"), FD("tree_bark")], MC("oak_log")).processingTime(50)
})

onEvent('recipes', event => {
// 移除凿子的两个合成
    event.remove({id: CH("iron_chisel")})
    event.remove({id: CH("diamond_chisel")})

// 为其添加不一样的合成（有序），这一步是因为他和农夫乐事的小刀的合成冲突
    event.shaped(CH("iron_chisel"), [
        'R ',
        ' M',
    ], {
        R: F('#rods'),
        M: MC('iron_ingot')
    })

    event.shaped(CH("diamond_chisel"), [
        'R ',
        ' M',
    ], {
        R: F('#rods'),
        M: MC('diamond')
    })
})


// 修改聊天事件 不需要/reload  使用 /kubejs reload server_scripts... 就可，不会卡ui线程
// 指令 
onEvent('player.chat', event => {
    // Check if message equals creeper, ignoring case
    if (event.message.startsWith('查询太行山积雪深度')) {
        event.player.tell('66.57cm')
        event.cancel() //隐藏指令
    }
})

```

查看游戏内的配方：

可以先在游戏内 `kubejs export ...` 然后去 `world/kubejs/exported` 查看导出文件

