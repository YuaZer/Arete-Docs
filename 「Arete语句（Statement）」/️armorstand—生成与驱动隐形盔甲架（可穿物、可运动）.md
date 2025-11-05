# 🛡️ armorstand — 生成与驱动隐形盔甲架（可穿物、可运动）

## 简介
`armorstand` 用于**生成一个不可见/无碰撞的盔甲架**，可为其穿上物品并进行**轨迹运动**（内置 `orbit` 预设）或**直线补间**，到时自动移除。非常适合做：

+ 环绕的“武器/符文/光翼/光球”；
+ 直线/跟随弹道（配合 `to` + `move`）；
+ 环绕后冲刺类“先演后打”的演出；
+ 需要“可挂装备”的飘浮视觉骨骼。

---

## 语法
```plain
armorstand {
    id = "as1"               # 可选：把生成的实体存到 vars["as1"]
    at = "@self"             # 生成位置
    slot = "head"            # 穿戴槽位：head/chest/legs/feet/hand/offhand
    item = "..."             # 穿戴物品（遵循物品解析规范）
    life = 100               # 生命周期（tick），到时自动移除
    name = "&e&l黄金守卫"     # 实体名称
    nameVisible = true      # 是否显示名称(不填则默认为true)
    small = false            # 小型化
    marker = true            # 标记模式（无碰撞、命中箱极小，推荐开）
    
    # 直线补间（可选）
    to = "${targetPos}"      # 终点坐标
    move = 20                # 运动时长（tick）
    easing = "linear"        # linear/easeIn/easeOut/easeInOut

    # 轨迹预设：环绕（可选）
    preset = "orbit"
    center = "@self"         # 围绕中心
    radius = 1.5             # 半径
    duration = 40            # 环绕时长（tick）
    revolutions = 1.0        # 绕圈数
    index = 0                # 分组索引（用于多件环绕：第几件）
    total = 1                # 分组总数（用于多件环绕：共几件）
}
```

---

## 参数说明
| 参数            | 类型      | 默认       | 说明                                                                        |
|---------------|---------|----------|---------------------------------------------------------------------------|
| `id`          | String? | `null`   | 若填写，将生成的 `ArmorStand`<br/> 存入 `ctx.vars[id]`<br/>，便于后续引用/移除               |
| `at`          | String? | `@self`  | 生成位置；支持 `@self/@target/${var}/x,y,z/~相对`                                  |
| `slot`        | String  | `head`   | 穿戴槽位：`head/chest/legs/feet/hand/offhand`<br/>（内部映射到 `EquipmentSlot`<br/>） |
| `item`        | String? | `null`   | 穿戴物品（你的 `parseItemStack(itemSpec)`<br/> 规则）                               |
| `name`        | String? | `null`   | 实体名称                                                                      |
| `nameVisible` | Boolean | `true`   | 是否显示名称                                                                    |
| `life`        | Long    | `100`    | 生存时长（tick），到时自动 `remove()`                                                |
| `small`       | Boolean | `false`  | 是否小型化（更迷你）                                                                |
| `marker`      | Boolean | `true`   | 标记模式（无碰撞，命中箱极小，演出推荐开启）                                                    |
| `to`          | String? | `null`   | 直线运动的终点坐标                                                                 |
| `move`        | Int     | `0`      | 直线补间时长（tick），`0`<br/> 表示不做直线补间                                            |
| `easing`      | String  | `linear` | 补间曲线：`linear/easeIn/easeOut/easeInOut`                                    |
| `preset`      | String? | `null`   | 轨迹预设，当前支持 `orbit`<br/>（环绕）                                                |
| `center`      | String? | `@self`  | `orbit`<br/> 的环绕中心                                                        |
| `radius`      | Double  | `1.5`    | `orbit`<br/> 的半径                                                          |
| `duration`    | Int     | `40`     | `orbit`<br/> 的持续时长（tick）                                                  |
| `revolutions` | Double  | `1.0`    | `orbit`<br/> 的圈数（1.0 即 360°）                                              |
| `index`       | Int     | `0`      | 多件环绕时的第几个（用于错相位）                                                          |
| `total`       | Int     | `1`      | 多件环绕的总数                                                                   |


---

## 基础示例
```plain
# 带名称/描述/附魔/自定义模型
armorstand {
  slot = "head"
  item = "minecraft:diamond_sword name:\"&6日耀之刃\" lore:\"&7恒星之火|&7穿透黑夜\" ench:\"sharpness=5,unbreaking=3\" cmd=123"
}

# 皮革盔甲染色 + 不可破坏 + 隐藏标志
var { colorHex = "#FFAA00" }
armorstand {
  slot = "chest"
  item = "leather_chestplate color:${colorHex} unbreakable=true flags:\"HIDE_ENCHANTS,HIDE_ATTRIBUTES\""
}

# 玩家头（支持名字或 UUID）
armorstand {
  slot = "head"
  item = "player_head owner:\"YuaZer\""
}
# 自定义名称
armor-stand {
  at = "@self"
  slot = "head"
  item = "minecraft:carved_pumpkin"
  name = "&e&l黄金守卫"
  # nameVisible 默认为 true（只要写了 name 就会显示）
}
```

### 1) 头顶漂浮图标，2 秒后消失
```plain
armorstand {
    id = "halo"
    at = "@self"
    slot = "head"
    item = "minecraft:gold_block"
    life = 40
    marker = true
}
```

### 2) 从自己飞向目标（直线补间 1 秒）
```plain
raycast { range = 20; storePos = "impact" }

armorstand {
    at = "@self"
    slot = "head"
    item = "minecraft:trident"
    to = "${impact}"
    move = 20
    easing = "easeOut"
    life = 25
}
```

---

## 轨迹预设：环绕（`preset = "orbit"`）
### 3) 单件环绕 2 秒，后冲刺至目标
```plain
raycast { range = 20; storePos = "hit" }

armorstand {
    slot = "head"
    item = "minecraft:iron_sword"
    preset = "orbit"
    center = "@self"
    radius = 1.2
    duration = 40
    revolutions = 1.5

    to = "${hit}"
    move = 12
    easing = "easeIn"
    life = 60
}
```

### 4) 四件武器均分环绕（index/total）
```plain
parallel {
    armorstand { slot = "head"; item = "minecraft:iron_sword"; preset = "orbit"; radius = 1.5; duration = 40; revolutions = 1.0; index = 0; total = 4 }
    armorstand { slot = "head"; item = "minecraft:iron_sword"; preset = "orbit"; radius = 1.5; duration = 40; revolutions = 1.0; index = 1; total = 4 }
    armorstand { slot = "head"; item = "minecraft:iron_sword"; preset = "orbit"; radius = 1.5; duration = 40; revolutions = 1.0; index = 2; total = 4 }
    armorstand { slot = "head"; item = "minecraft:iron_sword"; preset = "orbit"; radius = 1.5; duration = 40; revolutions = 1.0; index = 3; total = 4 }
}
```

---

## 与其它语句联动
### 5) 生成后记录 ID，稍后手动移除
```plain
seq {
    armorstand {
        id = "dagger"
        at = "@self"
        slot = "head"
        item = "minecraft:iron_sword"
        life = 0        # 不自动清理
    }
    delay { ticks = 60 } {
        # 你可以实现一个 remove-entity { targetVar = "dagger" } 语句来移除
        message { text = "&7匕首解除。" }
    }
}
```

### 6) 生成环绕 → 结束后爆炸特效
```plain
seq {
    armorstand {
        at = "@self"
        slot = "head"
        item = "minecraft:nether_star"
        preset = "orbit"
        radius = 1.2
        duration = 30
        life = 30
    }
    particle { type = "EXPLOSION_LARGE"; count = 2; at = "@self" }
}
```
### 7) 与预设轨迹联动（围绕玩家旋转，且显示编号）
```plain
# total=8 个盔甲架围绕玩家旋转一圈，每个都有编号名
for { from = 0; to = 7; step = 1; var = "i" } {
  armor-stand {
    at = "@self"
    preset = "orbit"
    center = "@self"
    radius = 2.2
    duration = 40
    revolutions = 1.0
    index = ${i}
    total = 8
    name = "&b哨兵 #${i}"
    nameVisible = true
    life = 60
  }
}
```


> 更新: 2025-10-17 18:33:22  
> 原文: <https://www.yuque.com/yuazer/blow95/shar3rs2qvcm64r8>