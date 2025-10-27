# 🎯 raycast — 获取视线命中点 / 命中实体指针

## 🧩 功能简介
`raycast` 从**玩家视线**发射一条射线，在指定 `range` 内采集：

+ **命中点位**（方块或实体表面坐标）→ 保存到 `storePos`
+ **命中实体**（可选）→ 保存到 `storeEntity`
+ 便捷指针 `pointer`（命中实体时自动写入 / 未命中时清空）

实现要点（与你的类一致）：

+ 仅当 `**ctx.source**`** 是玩家** 时才执行；
+ **主线程**执行 RayTrace；
+ `FluidCollisionMode.NEVER`（忽略液体）、`ignorePassableBlocks=true`（会无视可穿透方块如草、门等）；
+ 若没命中任何东西，命中点fallback为**视线尽头**（`eye + dir * range`）。

---

## ⚙️ 语法
```plain
raycast {
  range = 40                 # Double，射线长度（默认 40）
  storePos = "aimPos"        # String，命中点变量名（必填）
  storeEntity = "aimEnt"     # String，可选：命中实体变量名
}
```

---

## 📦 写入的变量
| 变量名 | 类型 | 说明 |
| --- | --- | --- |
| `<storePos>` | `Location` | 命中点（或视线末端）的位置对象 |
| `<storePos>Str` | `String` | 便于展示的字符串：`world,x,y,z`<br/>（坐标保留2位小数） |
| `<storeEntity>` | `Entity` | 命中实体（仅在命中时写入；未命中会 `remove`<br/>） |
| `<storeEntity>Str` | `String` | 命中实体名称（命中时写入；未命中会 `remove`<br/>） |
| `pointer` | `Entity` | 命中实体时自动写入；未命中则 `remove("pointer")` |


在 `ifchain` / 条件表达式中请使用 `var.<name>` 访问，如：`var.pointer != null`、`var.aimPos != null`。

---

## ⚡ 快速示例
### ① 获取命中点与实体
```plain
raycast { range = 12 storePos = "aimPos" storeEntity = "aimEnt" }
message { text = "&7命中点: ${aimPosStr}" }
ifchain {
  case { cond = "var.aimEnt != null" } { message { text = "&a命中实体: ${aimEntStr}" } }
  else { message { text = "&c没有命中实体" } }
}
```

### ② 突进到视线方向（轻跳+冲刺）
```plain
seq {
  raycast { range = 10 storePos = "dashPos" }
  velocity { x = 0 y = 0.35 z = 1.0 mul = 1.2 }   # 自身向前上冲刺
}
```

### ③ 命中实体则挑飞（使用 `pointer`）
```plain
seq {
  raycast { range = 10 storeEntity = "hit" }
  ifchain {
    case { cond = "var.pointer != null" } {
      velocity { x = 0 y = 0.9 z = 0 target = "pointer" }
      sound { key = "entity.player.attack.knockback" v = 1 p = 1 at = "@self" }
    }
    else { message { text = "&7未命中目标~" } }
  }
}
```

### ④ 在命中点释放范围伤害与特效（用 `storePos`）
```plain
seq {
  raycast { range = 16 storePos = "boomPos" }
  particle { type = "explosion" at = "${boomPos}" }
  target-entity {
    center = "${boomPos}"
    radius = 3.5
    type = "living"
    includeSelf = false
    storeList = "victims"
  }
  damage { amount = 8.0 targetVar = "victims" }
  knockback { targetVar = "victims" s = 0.9 y = 0.25 }
}
```

### ⑤ “视线锁定后多段连击”
```plain
seq {
  raycast { range = 12 storeEntity = "lock" }
  ifchain {
    case { cond = "var.lock != null" } {
      for { times = 3 } {
        damage { amount = 3.0 targetVar = "lock" }
        velocity { x = 0 y = 0.35 z = -0.6 targetVar = "lock" add = true } # 连段后坐
        delay { tick = 2 }
      }
    }
    else { message { text = "&7没锁到目标，取消连击。" } }
  }
}
```



> 更新: 2025-10-17 18:26:35  
> 原文: <https://www.yuque.com/yuazer/blow95/rnzz0121vxg5e9y9>