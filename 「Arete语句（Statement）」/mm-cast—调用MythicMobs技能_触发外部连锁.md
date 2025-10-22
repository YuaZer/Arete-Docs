# 🐉 mm-cast — 调用 MythicMobs 技能 / 触发外部连锁

## 🧩 功能简介
`mm-cast` 用于在 Arete 脚本中**直接调用 MythicMobs 技能**。

+ 触发者来源：`self | target | pointer | triggerVar`
+ 释放位置：`@self / 位置变量 / 绝对坐标`
+ 目标集合：实体 `et` / 位置 `lt`
+ 强度：`power`

内部会解析：施法者 `caster = ctx.source`、触发者（优先 `triggerVar`）、释放点 `origin`（默认 `@self`），并把 `et/lt` 变量解析成集合传给 MM。

---

## ⚙️ 语法
```plain
mm-cast {
  skill = "技能名"                      # 必填
  trigger = "self|target|pointer|none"  # 触发者模式（可选，默认 self）
  triggerVar = "变量名"                  # 从 ctx.vars 取触发者（优先级高于 trigger）
  origin = "@self|${posVar}|world:x,y,z" # 释放位置（可选，默认 @self）
  et = "实体集合变量"                    # 实体目标(可单体或集合)
  lt = "位置集合变量"                    # 位置目标(可单体或集合)
  power = 1.0                            # 技能强度(浮点)
}
```

### 参数一览表
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `skill`<br/> / `name` | `String` | ✅ | — | MythicMobs 技能名称 |
| `trigger` | `String`<br/> (`self/target/pointer/none`<br/>) | 否 | `self` | 触发者来源（当 `triggerVar`<br/> 为空时生效） |
| `triggerVar` | `String` | 否 | — | 从 `ctx.vars`<br/> 读取触发者（`Entity`<br/> 或集合首个），**优先于 **`**trigger**` |
| `origin` | `String` | 否 | `@self` | 释放位置，走 `ctx.resolveLocation`<br/>（支持 `@self`<br/> / `${posVar}`<br/> / 绝对坐标） |
| `et`<br/> / `entities` | `String` | 否 | — | 实体目标变量；支持 `Entity`<br/>、`Collection<Entity>`<br/>、`Array<Entity>` |
| `lt`<br/> / `locations` | `String` | 否 | — | 位置目标变量；支持 `Location`<br/>、`Collection<Location>`<br/>、`Array<Location>` |
| `power` | `Float` | 否 | `1.0` | 技能强度 |


线程安全：最终 `castSkill(...)` 在 `onMain {}` 内执行。  
触发者选择：`triggerVar`（如 `lock`）> `trigger`（self/target/pointer/none）。

---

## ⚡ 快速示例
### ① 基础释放：以自己为触发者，在自身位置释放
```plain
mm-cast { skill = "Fireball" power = 1.2 }
```

### ② 射线锁定为触发者 + 目标
```plain
raycast { range = 16 storeEntity = "lock" }
mm-cast { skill = "ChainLightning" triggerVar = "lock" et = "lock" power = 1.0 }
```

### ③ AOE：以光标点为释放位置，对范围内实体释放
```plain
raycast { range = 20 storePos = "aim" }
target-entity {
  center = "${aim}"
  radius = 6
  type = "living"
  includeSelf = false
  storeList = "victims"
}
mm-cast { skill = "ShoutFear" origin = "${aim}" et = "victims" power = 1.0 }
```

### ④ 与 ifchain 联动：有锁定则追击，无锁定则在末端点爆
```plain
raycast { range = 18 storeEntity = "hit" storePos = "hitPos" }
ifchain {
  case { cond = "var.hit != null" } {
    mm-cast { skill = "HomingStrike" triggerVar = "hit" et = "hit" power = 1.0 }
  }
  else {
    mm-cast { skill = "GroundBurst" origin = "${hitPos}" power = 1.0 }
  }
}
```

### ⑤ 群体链：随机抽 3 个玩家作为 et 目标
```plain
target-entity { center = "@self" radius = 12 type = "player" sort = "random" limit = 3 storeList = "picked" }
mm-cast { skill = "ThunderChain" et = "picked" power = 1.0 }
```

---

## 🧠 实战建议
+ **指针协同**：搭配 `target-entity { pointer = true }` 或 `raycast` 的 `pointer`，可直接 `trigger=pointer` 或 `triggerVar="pointer"`。
+ **变量容错**：`et/lt` 支持**单体**或**集合**。单体变量也能直接作为集合传入（内部已兼容）。
+ **位置解析**：`origin` 走 `ctx.resolveLocation`，因此 `${posVar}`、`@self`、`world:x,y,z` 都可用。
+ **线程安全**：无需自己处理线程切换，语句内部已 `onMain {}` 调用 MythicMobs API。
+ **强度与表现**：`power` 由具体 MM 技能定义；并非所有技能都会对 `power` 敏感，依据你的 MM 配置决定。

---

## 🧱 常见坑位
1. **忘记技能名**：`skill` 必填，缺失将抛出 `mm-cast: skill is required`。
2. **触发者空值**：当 `trigger = target/pointer`，但 `ctx.target/pointer` 为空时，触发者会是 `null`；如果技能强依赖触发者，建议在外层先用 `ifchain` 判空。
3. **变量类型不匹配**：`et/lt` 若为混合类型集合，会在内部过滤，非目标类型会被忽略。
4. **高频调用**：连续帧调用外部技能可能昂贵，建议在 `for`/循环里加 `delay` 或把多段逻辑封装成单个 MM 技能。

---

## 🧾 综合示例：风刃锁敌 · 三段落
```plain
seq {
  # 1) 锁定与提示
  raycast { range = 16 storeEntity = "lock" storePos = "aim" }
  ifchain {
    case { cond = "var.lock != null" } { message { text = "&a锁定: ${lockStr}" } }
    else { message { text = "&7未命中目标，释放落点爆裂" } }
  }

  # 2) 追击或点爆（MM 技能内含弹道/表现）
  ifchain {
    case { cond = "var.lock != null" } {
      mm-cast { skill = "WindChase" triggerVar = "lock" et = "lock" power = 1.0 }
    }
    else {
      mm-cast { skill = "WindBurst" origin = "${aim}" power = 1.0 }
    }
  }

  # 3) 余波（范围二次结算）
  target-entity { center = "${aim}" radius = 4 type = "living" includeSelf = false storeList = "victims" }
  mm-cast { skill = "WindShock" et = "victims" power = 0.8 }
}
```



> 更新: 2025-10-17 16:53:45  
> 原文: <https://www.yuque.com/yuazer/blow95/yug8pthzefvzclpw>