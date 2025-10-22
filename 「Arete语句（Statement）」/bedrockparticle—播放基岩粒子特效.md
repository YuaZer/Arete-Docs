# 🌈 bedrockparticle — 播放基岩粒子特效

## 🧩 功能简介
`bedrockparticle` 语句用于 **在玩家客户端上播放基岩粒子效果**。

注意: 该语句需要前置  
服务端: ArcartX [https://arcartx.com/](https://arcartx.com/)  
客户端: BedrockParticle [https://github.com/17Artist/BedrockParticle](https://github.com/17Artist/BedrockParticle)

---

## ⚙️ 语法
```plain
bedrockparticle {
  id = "minecraft:campfire_smoke_particle"   # 必填：粒子资源路径
  at = "@self"                               # 可选：位置变量、@self/@target/坐标等
  data = "r=0,g=255,b=200,size=1.0"          # 可选：附加参数 (键=值, 以逗号分隔)
}
```

### 参数一览表
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `id`<br/> / `name` | `String` | ✅ | — | 粒子资源路径，如 `minecraft:flame_particle`<br/>、`minecraft:campfire_smoke_particle` |
| `at` | `String` | 否 | `@self` | 粒子生成位置，走 `ctx.resolveLocation`<br/>，支持：`@self`<br/>、`@target`<br/>、`${posVar}`<br/>、`world:x,y,z` |
| `data` | `String` | 否 | `{}` | 粒子附加参数（以 `key=value`<br/> 形式，用逗号分隔） |


⚙️ 示例：`data = "r=255,g=120,b=80,size=1.5,speed=0.1"`

---

## 💥 示例用法
### 🎇 ① 自身位置播放火焰粒子
```plain
bedrockparticle { id = "minecraft:flame_particle" at = "@self" }
```

在释放者位置播放火焰粒子（仅 Bedrock 玩家可见）

---

### 🌀 ② 光标点生成能量漩涡
```plain
raycast { range = 20 storePos = "aim" }
bedrockparticle {
  id = "minecraft:dragon_breath_particle"
  at = "${aim}"
  data = "r=120,g=0,b=255,size=2.0"
}
```

使用 `raycast` 获取玩家瞄准点，然后在该点生成粒子。

---

### ✨ ③ 群体环绕演出
```plain
target-entity { center = "@self" radius = 5 type = "player" storeList = "near" }
for { var = "p" times = 5 } {
  bedrockparticle {
    id = "minecraft:happy_villager_particle"
    at = "@self"
    data = "r=50,g=255,b=50,size=1.2"
  }
}
```

在自己周围多次生成绿色粒子，营造祝福特效。

---

### 💫 ④ 配合 move / ifchain 制作冲刺轨迹
```plain
move { target = "self" to = "${aimPos}" move = 10 }
bedrockparticle {
  id = "minecraft:cloud_particle"
  at = "@self"
  data = "r=255,g=255,b=255,size=0.5"
}
```

玩家移动时，身后生成云雾轨迹，仅 Bedrock 玩家可见。

---

## 🧠 使用建议
+ **粒子 ID 来源**：使用 Bedrock 官方粒子 ID，例如：
    - `minecraft:flame_particle`
    - `minecraft:campfire_smoke_particle`
    - `minecraft:explosion_particle`
    - `minecraft:enchanting_table_particle`
    - `minecraft:happy_villager_particle`
+ **参数格式**：所有附加参数为字符串对 (`key=value`)，用 `,` 分隔。  
例如：

```plain
data = "r=255,g=0,b=0,size=1.5,speed=0.2"
```

会生成红色、稍大的粒子。

+ **桥接支持**：确保你的环境中存在 `Bridges.bedrockParticle` 桥接模块，否则语句不会生效（静默跳过）。
+ **线程安全**：无需手动 `onMain`，语句内部已自动解析位置与调用。
+ **演出协同**：推荐与 `seq` / `parallel` / `delay` 组合，形成多层粒子特效。

---

## 📄 综合示例：技能释放动画
```plain
seq {
  # 锁定目标
  raycast { range = 16 storePos = "aim" }
  message { text = "&a聚气中..." }

  # 聚气阶段
  for { times = 10 } {
    bedrockparticle {
      id = "minecraft:enchanting_table_particle"
      at = "@self"
      data = "r=120,g=200,b=255,size=0.5"
    }
    delay 2
  }

  # 释放阶段
  bedrockparticle {
    id = "minecraft:explosion_particle"
    at = "${aim}"
    data = "size=2.0"
  }
  sound { sound = "ENTITY_GENERIC_EXPLODE" at = "${aim}" v = 1.2 p = 1.0 }
}
```

🎬 效果：玩家周身聚气 → 指向位置爆炸 → 同时播放音效。



> 更新: 2025-10-17 16:58:05  
> 原文: <https://www.yuque.com/yuazer/blow95/dy4ytuc5fghxwkgd>