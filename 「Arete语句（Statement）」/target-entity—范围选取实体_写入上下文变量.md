# 🎯 target-entity — 范围选取实体 / 写入上下文变量

## 🧩 功能简介
`target-entity` 用于**以某个中心点和半径/长宽高**为条件，筛选附近实体，并把结果写入上下文变量（首个、列表、位置、距离、数量），还能按类型/名称/标签/视线等进行过滤，并支持最近/最远/随机排序与数量上限。可选把首个命中的实体写入全局指针 `pointer`，方便后续语句直接使用。

---

## 🎯 `target-entity` 参数总览表

| 参数名                                          | 类型                                                        | 必填   | 默认值       | 说明                                                                                                                                                                                                                                                          |
|----------------------------------------------|-----------------------------------------------------------|------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **center** / **at** / **position** / **pos** | `String`                                                  | 否    | `null`    | 检测中心位置（AABB模式下使用）。可为 `@self`、`@target`、`${posVar}`、`world:x,y,z` 等能被 `ctx.resolveLocation(...)` 解析的任意位置。只有当 **未使用 `path`** 时才会生效。                                                                                                                           |
| **offset** / **shift** / **delta**           | `String` (`"x,y,z"`)                                      | 否    | 无         | 额外偏移，会在解析出 `center` 之后再平移这个向量（单位为方块）。只在 AABB 模式（非 `path`）下使用，用来微调盒子的中心。                                                                                                                                                                                     |
| **radius** / **range**                       | `Double`                                                  | 否    | `8.0`     | AABB 模式下的半径基准值。如果未显式指定 `radiusX` / `radiusY` / `radiusZ`，则三个轴都会回落到该值。                                                                                                                                                                                       |
| **radiusX** / **rangeX** / **width**         | `Double`                                                  | 否    | `radius`  | AABB 模式：X轴方向的一半长度（左右各扩多少格）。                                                                                                                                                                                                                                 |
| **radiusY** / **rangeY** / **height**        | `Double`                                                  | 否    | `radius`  | AABB 模式：Y轴方向的一半长度（上下各扩多少格）。                                                                                                                                                                                                                                 |
| **radiusZ** / **rangeZ** / **depth**         | `Double`                                                  | 否    | `radius`  | AABB 模式：Z轴方向的一半长度（前后各扩多少格）。                                                                                                                                                                                                                                 |
| **path**                                     | `String` (变量名)                                            | 否    | `null`    | 路径模式：指定一个变量名，比如 `curve`。此变量必须是 `List<Location>`（例如由 `nurbs` 语句生成的路径采样）。如果填写了 `path`，则本语句会**忽略 `center` / `radiusX` / `radiusY` / `radiusZ`**，转而沿整条路径逐点扫描实体（见 `pathRadius`）。                                                                                 |
| **pathRadius**                               | `Double`                                                  | 否    | `1.0`     | 路径模式下使用。表示对路径上的每个点进行一次 `getNearbyEntities(point, r, r, r)` 的半径 `r`。半径是立方体/球形近似的半径，不区分水平和垂直方向。路径上所有点的扫描结果会合并并去重。                                                                                                                                             |
| **limit** / **take**                         | `Int`                                                     | 否    | `1`       | 最终返回多少个实体。`>0` 时会在排序后 `take(limit)`；如果你想“所有命中都返回”，可传 `0` 或负数（即不截断）。                                                                                                                                                                                         |
| **sort**                                     | `String` (`nearest` / `farthest` / `random` / `unsorted`) | 否    | `nearest` | 决定排序方式。`nearest` 最近优先，`farthest` 最远优先，`random` 随机打乱，`unsorted` 保持内部顺序（路径模式下是路径扫描的加入顺序，AABB 模式下是 Bukkit 返回迭代顺序）。排序后会再应用 `limit`。                                                                                                                             |
| **type** / **types**                         | `String` (逗号分隔)                                           | 否    | `*`       | 类型过滤。支持关键字：<br/>- `*`, `any`, `all`：任意实体<br/>- `player` / `players`：玩家<br/>- `living` / `livingentity`：任意生物（LivingEntity）<br/>- `monster` / `hostile`：怪物系（Monster）<br/>- `projectile`：投射物<br/>- `nonliving`：非生物实体<br/>也支持具体原版类型名（如 `ARMOR_STAND`、`ZOMBIE`）。 |
| **nameContains** / **contains**              | `String`                                                  | 否    | 无         | 名称模糊匹配。忽略大小写。会同时检查实体的 `displayName`、`customName`（如果有）。若填写，则实体名字必须包含该子串。                                                                                                                                                                                     |
| **nameEquals** / **name**                    | `String`                                                  | 否    | 无         | 名称精确匹配。忽略大小写。会同时比对展示名和自定义名。<br/>注意：当你传了 `name=` 时，如果同时传了 `contains=`，`name` 只在 `contains` 为空时才会生效（与当前实现逻辑一致）。                                                                                                                                               |
| **tags**                                     | `String` (逗号分隔)                                           | 否    | 无         | 只保留带指定 scoreboard tag 的实体。例如 `tags=Boss,Phase2`。                                                                                                                                                                                                            |
| **tagMode**                                  | `String` (`any` / `all` / `every` / `and`)                | 否    | `any`     | 标签匹配逻辑：<br/>- `any`（默认）：实体至少有其中一个标签即可<br/>- `all` / `every` / `and`：实体必须同时包含全部标签                                                                                                                                                                            |
| **includeSelf** / **excludeSelf**            | `Boolean`                                                 | 否    | `false`   | 是否允许把施法者自己也当成目标。<br/>优先级规则：`includeSelf` 显式指定优先；否则如果传了 `excludeSelf=true` 则强制不包含；都没写则默认不包含。                                                                                                                                                                 |
| **includeTarget**                            | `Boolean`                                                 | 否    | `true`    | 是否允许把 `ctx.target`（上下文里的目标实体，例如技能锁定对象）一并纳入候选。                                                                                                                                                                                                               |
| **lineOfSight**                              | `Boolean`                                                 | 否    | `false`   | 是否要求“施法者(LivingEntity)必须能直视到该实体”（`hasLineOfSight` 判定）。为 `true` 时，如果执行者不是 `LivingEntity`，则该检查被跳过。                                                                                                                                                            |
| **alive**                                    | `Boolean`                                                 | 否    | `false`   | 若为 `true`：<br/>- 只保留 `LivingEntity`<br/>- 且 `!isDead`。也就是只打活着的生物，屏蔽掉盔甲架、投射物、已死亡的尸体等。                                                                                                                                                                        |
| **pointer**                                  | `Boolean`                                                 | 否    | `true`    | 是否将**最终列表的第一个实体**写入 `ctx.vars["pointer"]`。当没有命中实体时会清除这个变量。这让后续的 `damage target=pointer` 之类语句能直接打最近的那个。                                                                                                                                                      |
| **store** / **as** / **id**                  | `String`                                                  | ✅ 必填 | 无         | 把排序/截断之后列表里的第一个实体，写进 `ctx.vars[这个名字]`。同时写一个 `${name}Str` 方便显示（是可读名字）。例如 `store=firstHit` → `ctx.vars["firstHit"]` = 最近的怪，`ctx.vars["firstHitStr"]` = 这个怪的名字。                                                                                                |
| **storeList** / **storeAll**                 | `String`                                                  | 否    | 无         | 把最终命中的实体集合（`List<Entity>`，应用过滤、排序、limit 之后的结果）写入该变量名。例如 `storeList=hitAll` → `ctx.vars["hitAll"]`。同时会生成 `xxxStr`，是名字用逗号拼接的可读列表。                                                                                                                             |
| **storeLocation** / **storePos**             | `String`                                                  | 否    | 无         | 将第一个命中的实体的位置 (`Location.clone()`) 存到这个变量，并额外存 `${varName}Str`（如 `world,100.23,65.00,200.77`）。                                                                                                                                                               |
| **storeDistance** / **storeDist**            | `String`                                                  | 否    | 无         | 将第一个命中实体与参考点的距离（单位方块，Double）写入该变量。<br/>参考点含义：<br/>- AABB 模式：使用 `center+offset` 作为参考点<br/>- 路径模式：使用该实体被捕捉到的那一帧对应的路径点做距离平方计算（对你来说就是“离路径最近的那个抽样点”的距离）。                                                                                                         |
| **storeCount** / **count**                   | `String`                                                  | 否    | 无         | 将最终命中的实体数量（`Int`）写入该变量。例如 `storeCount=hitCount` → `ctx.vars["hitCount"]=3`。                                                                                                                                                                                 |

---

### 两种工作模式怎么选？

#### 1. AABB（盒子）模式
不写 `path`。按照技能执行者（`@self`）的坐标，以技能执行者为中心，以技能执行者的 AABB（实体的盒子）为范围。
#### 2. Path（路径）模式
写 `path`。技能执行者（`@self`）会以nurbs所画曲线为路径，以`路径点*路径体积`为范围。


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