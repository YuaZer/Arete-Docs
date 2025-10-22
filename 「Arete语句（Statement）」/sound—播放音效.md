# 🔊 sound — 播放音效

## 简介
`sound` 用于在指定位置播放 Bukkit 内置枚举 `org.bukkit.Sound` 的音效。  
常用于：技能释放声、命中声、蓄力/落地/爆炸提示音、环境营造等。

当前实现特点：

+ 使用 `Sound.valueOf(key.uppercase())` 解析枚举名；
+ 只支持 **原版/服务端内置枚举** 的声音键；
+ 支持设置音量与音调；支持 `at` 指定播放位置。

---

## 语法
```plain
sound {
    sound = "ENTITY_PLAYER_LEVELUP"  # 或 key = "ENTITY_PLAYER_LEVELUP"
    v = 1.0                          # 音量（float）
    p = 1.0                          # 音调/音高（float）
    at = "@self"                     # 播放位置（可选）
}
```

`sound` 与 `key` 二选一；内部都会作为 `Sound.valueOf(...)` 来解析。

---

## 参数说明
| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `sound`<br/>/`key` | String | `"ENTITY_PLAYER_LEVELUP"` | Bukkit `Sound`<br/> 枚举名（大小写不敏感，内部会转大写） |
| `v` | Float | `1.0` | 音量（一般 0.0 ~ 1.0，也可更大以增幅范围） |
| `p` | Float | `1.0` | 音调（`0.5`<br/> 低沉、`1.0`<br/> 正常、`2.0`<br/> 尖锐） |
| `at` | String? | `null` | 播放位置；为空时用语句默认位置解析为 `ctx.resolveLocation(at)`<br/>（通常为执行者） |


注意：你的实现里若 `at` 为空会回退到 `resolveLocation(null)` 的逻辑，一般等价于 `@self`。

---

## 基础示例
### 1) 释放提示音
```plain
sound {
    sound = "ENTITY_BLAZE_SHOOT"
    v = 1.0
    p = 1.3
    at = "@self"
}
```

### 2) 命中位置播放爆炸音
```plain
raycast { range = 20; storePos = "hit" }

sound {
    sound = "ENTITY_GENERIC_EXPLODE"
    v = 0.9
    p = 1.0
    at = "${hit}"
}
```

### 3) 低沉的蓄力音效
```plain
sound {
    sound = "BLOCK_BEACON_AMBIENT"
    v = 0.8
    p = 0.6
    at = "@self"
}
```

---

## 组合示例
### ⚔️ 与 `particle`/`damage` 联动
```plain
seq {
    sound   { sound = "ENTITY_WITHER_SPAWN"; v = 1.0; p = 0.7; at = "@self" }
    particle{ type = "SMOKE_LARGE"; count = 30; at = "@self" }
    damage  { amount = 8.0; target = "target" }
}
```

### 🎲 与 `random` 做多变音色
```plain
random { weights = "2,1" } {
    seq { sound { sound = "ENTITY_ZOMBIE_ATTACK_WOODEN_DOOR"; v = 1; p = 1.0 } }
    seq { sound { sound = "ENTITY_ZOMBIE_ATTACK_WOODEN_DOOR"; v = 1; p = 1.4 } }
}
```

### ⏳ 与 `delay` 做连段音效
```plain
seq {
    sound { sound = "BLOCK_NOTE_BLOCK_PLING"; v = 1; p = 1.0 }
    delay { ticks = 5 } { sound { sound = "BLOCK_NOTE_BLOCK_PLING"; v = 1; p = 1.2 } }
    delay { ticks = 10 }{ sound { sound = "BLOCK_NOTE_BLOCK_PLING"; v = 1; p = 1.5 } }
}
```



> 更新: 2025-10-17 14:05:28  
> 原文: <https://www.yuque.com/yuazer/blow95/ytr67f5lgq8q71gz>