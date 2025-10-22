# 🎯 target-entity — 范围选取实体 / 写入上下文变量

## 🧩 功能简介
`target-entity` 用于**以某个中心点和半径/长宽高**为条件，筛选附近实体，并把结果写入上下文变量（首个、列表、位置、距离、数量），还能按类型/名称/标签/视线等进行过滤，并支持最近/最远/随机排序与数量上限。可选把首个命中的实体写入全局指针 `pointer`，方便后续语句直接使用。

---

## 🎯 `target-entity` 参数总览表
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| **center** | `String` | 否 | `@self` | 选取中心位置，可为 `@self`<br/>、`@target`<br/>、`${posVar}`<br/>、`world:x,y,z`<br/> 等可解析位置 |
| **offset** | `String`<br/> (格式 `x,y,z`<br/>) | 否 | 无 | 相对中心点的偏移量，可用于微调检测中心位置 |
| **radius** | `Double` | 否 | `8.0` | 三个方向统一半径；若指定 `radiusX/Y/Z`<br/> 则会被覆盖 |
| **radiusX** | `Double` | 否 | `radius` | X 轴半径（宽度） |
| **radiusY** | `Double` | 否 | `radius` | Y 轴半径（高度） |
| **radiusZ** | `Double` | 否 | `radius` | Z 轴半径（深度） |
| **limit** | `Int` | 否 | `1` | 限制返回的实体数量；`0`<br/> 表示不限制（取全部） |
| **sort** | `String`<br/> (`nearest`<br/> / `farthest`<br/> / `random`<br/> / `unsorted`<br/>) | 否 | `nearest` | 排序方式：最近、最远、随机或不排序 |
| **type / types** | `String`<br/> (逗号分隔) | 否 | `*` | 实体类型过滤，可使用关键字：`player`<br/>、`living`<br/>、`monster`<br/>、`projectile`<br/>、`nonliving`<br/>、`*`<br/> 或具体类型名（如 `ARMOR_STAND`<br/>） |
| **nameContains** | `String` | 否 | 无 | 名称模糊匹配，忽略大小写（匹配 `displayName`<br/> 或 `customName`<br/>） |
| **nameEquals / name** | `String` | 否 | 无 | 名称精确匹配，忽略大小写 |
| **tags** | `String`<br/> (逗号分隔) | 否 | 无 | 过滤拥有指定计分板标签的实体 |
| **tagMode** | `String`<br/> (`any`<br/> / `all`<br/>) | 否 | `any` | 标签匹配方式：`any`<br/> 任意一个匹配即可；`all`<br/> 必须全部拥有 |
| **includeSelf** | `Boolean` | 否 | `false` | 是否包含自己 |
| **includeTarget** | `Boolean` | 否 | `true` | 是否包含上下文 `target`<br/> 实体 |
| **lineOfSight** | `Boolean` | 否 | `false` | 是否要求与执行者有视线可见（仅限 `LivingEntity`<br/> 执行者） |
| **alive** | `Boolean` | 否 | `false` | 是否仅筛选“活体且存活”的实体 |
| **store** / **as** / **id** | `String` | ✅ 必填 | 无 | 首个命中实体写入的变量名，例如 `wind_primary` |
| **storeList** / **storeAll** | `String` | 否 | 无 | 存储命中的实体集合（List<Entity>） |
| **storeLocation** / **storePos** | `String` | 否 | 无 | 存储首个命中者位置（Location） |
| **storeDistance** / **storeDist** | `String` | 否 | 无 | 存储首个命中者距离（Double） |
| **storeCount** / **count** | `String` | 否 | 无 | 存储命中的实体数量（Int） |
| **pointer** | `Boolean` | 否 | `true` | 是否将首个命中实体写入全局 `ctx.vars["pointer"]`<br/>，方便后续语句使用 |


---

## 📦 附加生成的变量
执行完后，语句会自动生成以下变量：

| 变量名 | 类型 | 内容 |
| --- | --- | --- |
| `${store}` | `Entity` | 首个命中的实体 |
| `${store}Str` | `String` | 实体的显示名称（优先 `customName`<br/>） |
| `${storeList}` | `List<Entity>` | 命中实体集合 |
| `${storeList}Str` | `String` | 以名称拼接的实体列表 |
| `${storeLocation}` | `Location` | 首个命中实体的位置 |
| `${storeLocation}Str` | `String` | `world,x,y,z`<br/> 格式字符串（坐标保留两位小数） |
| `${storeDistance}` | `Double` | 首个实体距离（米） |
| `${storeCount}` | `Int` | 命中实体总数 |
| `pointer` | `Entity` | 当 `pointer = true`<br/> 时，自动更新为首个命中实体 |


---

## 🧭 常见参数组合建议
| 目标 | 推荐参数组合 |
| --- | --- |
| 最近的活体 | `type=living sort=nearest limit=1 alive=true` |
| 随机抽取若干玩家 | `type=player sort=random limit=3` |
| 视线中的怪物 | `type=monster lineOfSight=true` |
| 自身为中心的圆形AOE | `center=@self radius=5 includeSelf=false storeList=targets` |
| 指定点爆炸 | `center=${aimPos} radius=6 type=living storeList=victims` |


## ⚙️ 语法
```plain
target-entity {
  # 选取范围
  center = "@self|@target|${posVar}|world:x,y,z"   # 中心（可省略，默认解析成 @self）
  offset = "dx,dy,dz"                               # 对 center 的偏移（可选）
  radius  = 4.0                                     # 统一半径（立方/长方体半径，见下）
  radiusX = 4.0                                     # 单独指定长宽高
  radiusY = 4.0
  radiusZ = 4.0

  # 数量与排序
  limit = 1                                         # 取前 N 个；=0 取全部
  sort  = "nearest|farthest|random|unsorted"        # 默认 nearest

  # 过滤条件
  type  = "living,player,monster,projectile,ARMOR_STAND,..."  # 多值逗号分隔
  nameContains = "关键字"                         # 名称包含
  nameEquals   = "完整名称"                       # 名称精确匹配（高优先级）
  tags  = "boss,elite"                              # 计分板标签（scoreboardTags）
  tagMode = "any|all"                                # 任一/全部标签满足（默认 any）
  includeSelf = false                                # 是否包含自己（默认 false）
  includeTarget = true                               # 是否包含 ctx.target（默认 true）
  lineOfSight = false                                # 需与 source 视线可见（默认 false）
  alive = false                                      # 仅活体且存活（默认 false）

  # 存储变量
  store = "wind_primary"                             # 首个命中实体（必填）
  storeList = "wind_victims"                         # 命中实体列表（可选）
  storeLocation = "wind_pos"                         # 首个命中者位置（可选）
  storeDistance = "wind_dist"                        # 首个命中者距离（米，double，可选）
  storeCount = "wind_count"                          # 命中数量（可选）
  pointer = true                                     # 是否把首个命中者写入 ctx.vars["pointer"]（默认 true）
}
```

半径说明：使用 `world.getNearbyEntities(center, radiusX, radiusY, radiusZ)`，即**长方体半径**（非球体）；`radius` 同时作用于 X/Y/Z 三个方向。

---

## 🧾 写入的变量（含友好字符串）
+ `store`（必填）：首个命中 `Entity`；若无命中则移除该键
    - 额外写入：`${store}Str` → 便于显示的名称（优先 `customName`，玩家用 `displayName`）
+ `storeList`：`List<Entity>`（命中集合）；为空则移除
    - 额外写入：`${storeList}Str` → 以名称拼接的简表
+ `storeLocation`：`Location`（首个命中者的位置）；为空则移除
    - 额外写入：`${storeLocation}Str` → `world,x,y,z`（坐标保留两位小数）
+ `storeDistance`：`Double`（首个命中者距离，米）
+ `storeCount`：`Int`（命中数量）
+ `pointer`（当 `pointer = true` 时）：首个命中者；无命中则移除

在 `ifchain` 表达式中，使用 `var.<name>` 访问，如 `var.wind_victims`、`var.wind_primary`、`var.pointer != null`。

---

## 🔢 过滤与匹配要点
+ **类型**（`type`/`types`，多值逗号分隔）：
    - 预置关键字：`player|players`、`living|livingentity`、`monster|hostile`、`projectile`、`nonliving`、`*|any|all`
    - 或使用 **Bukkit 枚举名**：如 `ARMOR_STAND`, `VILLAGER`, `CREEPER`…
+ **名称**：`nameEquals` 精确匹配（含 `customName`/显示名），`nameContains` 模糊包含
+ **标签**：匹配 `Entity.getScoreboardTags()`；`tagMode = all` 要求全部命中
+ **可视线**：`lineOfSight = true` 时由 `source as LivingEntity` 判定 `hasLineOfSight(entity)`
+ **仅活体**：`alive = true` 仅保留活体，且排除死亡个体；非活体直接排除

---

## ⚡ 快速示例
### ① 最近的活体敌人，写入指针并展示距离
```plain
target-entity {
  center = "@self"
  radius = 4
  type = "living"
  includeSelf = false
  store = "primary"
  storeDistance = "dist"
  pointer = true
}
ifchain {
  case { cond = "var.primary != null" } {
    message { text = "&a最近目标: ${primaryStr}，距离 ${dist} 格" }
  }
  else { message { text = "&7附近没有活体目标" } }
}
```

### ② 取一圈怪物并群体结算
```plain
target-entity {
  center = "@self"
  radius = 3.5
  type = "monster"
  includeSelf = false
  store = "first"
  storeList = "victims"
  storeCount = "cnt"
}
message { text = "&e命中怪物数: ${cnt}" }
damage  { amount = 7.5 targetVar = "victims" }
knockback { targetVar = "victims" s = 0.9 y = 0.25 }
```

### ③ 只选带标签的 Boss，且要求视线可见
```plain
target-entity {
  center = "@self"
  radius = 12
  tags = "boss,phase2"
  tagMode = "all"
  lineOfSight = true
  store = "boss"
  storeLocation = "bossPos"
}
ifchain {
  case { cond = "var.boss != null" } {
    particle { type = "villager_happy" at = "${bossPos}" }
  }
  else { message { text = "&7视线中未发现 Boss(phase2)" } }
}
```

### ④ 以“光标点”为中心，随机抽取 3 个玩家
```plain
raycast { range = 20 storePos = "aim" }
target-entity {
  center = "${aim}"
  radius = 6
  type = "player"
  sort = "random"
  limit = 3
  store = "p"
  storeList = "players"
}
message { text = "&b抽到的玩家: ${playersStr}" }
```

### ⑤ 仅拿到首个命中者位置与名称
```plain
target-entity {
  center = "@self"
  radius = 6
  type = "living"
  store = "hit"
  storeLocation = "hitPos"
}
message { text = "&a命中: ${hitStr} @ ${hitPosStr}" }
```



> 更新: 2025-10-17 15:17:28  
> 原文: <https://www.yuque.com/yuazer/blow95/cre7sl9inxv6xa67>