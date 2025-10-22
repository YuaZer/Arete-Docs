# 如何创建一个简单的技能模板

在文件夹skill_template内新建文件夹或者直接新建文件 moba.yml

示例配置:

```yaml
统统肘开: |
  seq {
    raycast { range = 14 storePos = "hitPos" storeEntity = "hitTargets" }
    if { cond = "vars.hitPos != null" } {
      parallel {
        title { title = "&6统统肘开!"; subtitle = "&eMamba Out!"}
        sound { key = "ENTITY_ENDERMAN_TELEPORT" v = 2 p = 1.2 at = "@self" }
        particle { type = "SMOKE_NORMAL" count = 40 speed = 0.1 at = "@self" }
      }
      move-accel { mode = "self" to = "${hitPos}" accel = 0.6 max = 2.8 duration = 8 gravity = false }
      delay { ticks = 2 } {
        sound { key = "ENTITY_PLAYER_ATTACK_SWEEP" v = 1.5 p = 1.0 at = "${hitPos}" }
        particle { type = "SWEEP_ATTACK" count = 10 speed = 0.0 at = "${hitPos}" }
        damage { amount = 10 target = "nearby"; rx = "4.0"; ry = "4.0";  rz = "4.0" }
        effect { type = "WEAKNESS" time = 60 amp = 1 target = "pointer" }
        knockback { target = "pointer" s = 1.4 y = 0.3 }
      }
    }
  }
踏前斩: |
  #按顺序执行语句
  seq {
    message { text = "&b[疾风连斩]&f ${player}化作疾风突进！" }
    title { title = "&b踏前斩" subtitle = "&7切裂前方敌人" in = 5 stay = 25 out = 5 }
    sound { sound = "ENTITY_PLAYER_ATTACK_SWEEP" v = 1.2 p = 1.4 at = "@self" }
    #获取玩家看向的指定距离范围内的坐标和实体
    raycast { range = 12 storePos = "wind_dashPos" storeEntity = "wind_dashTarget" }
    #平滑移动自身,到刚才看到的坐标,加速度为0.5,设置速度最大为2.8,持续12ticks,速度停止阈值为0.3,重力设置为false
    move-accel { mode = "self"; to = "${wind_dashPos}"; accel = 0.5; max = 2.8; duration = 12; stopAt = 0.3; gravity = false }
    #播放粒子在自身位置
    particle { type = "SWEEP_ATTACK"; count = 24; at = "@self" }
    #播放粒子在玩家看向的坐标位置 偏移量:x=玩家坐标+0.45,y=玩家坐标-0.1,z=玩家坐标+0.45,
    particle { type = "CLOUD" count = 35 ox = 0.45 oy = 0.1 oz = 0.45 speed = 0.01 at = "${wind_dashPos}" }
    #选中中心为玩家自身,范围3.5格,所有实体,不包括自己的,活着的,选中的最近实体存储到wind_primary变量,选中的所有实体存储到wind_victims这个列表变量里
    target-entity { center = "@self" radius = 3.5 type = "living" includeSelf = false alive = true store = "wind_primary" storeList = "wind_victims" }
    #造成7.5的伤害,对wind_victims这个列表变量里的所有实体
    damage { amount = 7.5 targetVar = "wind_victims" }
    #药水效果,类型是速度,倍数是1倍,持续60ticks,目标是自己
    effect { type = "SPEED" amp = 1 time = 60 target = "self" }
    #播放音乐,音调,音量,播放给自己
    sound { sound = "ENTITY_IRON_GOLEM_ATTACK" v = 1.0 p = 1.2 at = "@self" }
  }
```



> 更新: 2025-10-17 01:08:03  
> 原文: <https://www.yuque.com/yuazer/blow95/qdbimy4zyxvdksts>