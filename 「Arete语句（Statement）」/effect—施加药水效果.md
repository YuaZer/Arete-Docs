# 🧪 effect — 施加药水效果

给一个或一组实体添加原版**药水效果（PotionEffect）**。

+ 类型名大小写不敏感（内部会 `uppercase()` 再匹配）。
+ 持续时间单位为 **tick**（20 tick = 1 秒）。
+ 目标解析优先级：`targetVar` > `target` > 默认 `self`。
+ 支持对**单个实体**或**实体列表变量**同时生效。

---

## 参数一览
| 参数 | 必填 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | ✔ | String | — | 药水类型键：如 `SPEED`<br/>, `REGENERATION`<br/>, `POISON`<br/>… |
| `amp` | ✖ | Int | `0` | 等级（`0`<br/>=I，`1`<br/>=II…） |
| `time` | ✖ | Int(tick) | `100` | 持续 tick（例：`200`<br/>=10s） |
| `target` | ✖ | String | `"self"` | 兼容字段：`self`<br/> / `target`<br/> / `pointer` |
| `targetVar` | ✖ | String | `null` | **推荐**：从 `ctx.vars[name]`<br/> 读目标；支持单体或集合 |


注：当 `targetVar` 存在时，`target` 将被忽略。

---

## 获取实体的常用前置（配合 `targetVar`）
### ① 射线选取（拿到“看向”的命中）
```plain
# 获取玩家看向的指定距离范围内的坐标和实体
raycast { range = 12 storePos = "wind_dashPos" storeEntity = "wind_dashTarget" }
```

### ② 范围选取（中心、半径、过滤条件）
```plain
# 选中中心为玩家自身, 范围 3.5 格, 活着的生物, 不含自己
# 最近实体存到 wind_primary, 全部实体列表存到 wind_victims
target-entity {
  center = "@self"
  radius = 3.5
  type = "living"
  includeSelf = false
  alive = true
  store = "wind_primary"
  storeList = "wind_victims"
}
```

---

## 快速上手示例
### A. 对自己加速 3 秒（等级 II）
```plain
effect { type = "SPEED" amp = 1 time = 60 }
```

### B. 对“当前目标”（`ctx.target`）施加虚弱 10 秒
```plain
effect { type = "WEAKNESS" amp = 0 time = 200 target = "target" }
```

### C. 对 `raycast` 命中的实体上毒
```plain
raycast { range = 12 storeEntity = "wind_hit" }
effect { type = "POISON" amp = 0 time = 120 targetVar = "wind_hit" }
```

### D. 对范围内**所有选中的实体**群体加抗性
```plain
target-entity {
  center = "@self"
  radius = 3.5
  type = "living"
  includeSelf = false
  alive = true
  storeList = "wind_victims"
}

# 群体生效（与 damage 的 targetVar 用法一致）
effect { type = "DAMAGE_RESISTANCE" amp = 1 time = 200 targetVar = "wind_victims" }
```

### E. 先伤害再给负面效果（与 `damage` 示例一致的变量）
```plain
# 造成 7.5 伤害，对 wind_victims 列表变量里的所有实体
damage { amount = 7.5 targetVar = "wind_victims" }

# 再给同一批实体添加缓慢 5 秒
effect { type = "SLOW" amp = 0 time = 100 targetVar = "wind_victims" }
```

### F. 单体优先目标（最近实体）+ 自己增益
```plain
target-entity {
  center = "@self" radius = 4 type = "living" includeSelf = false alive = true
  store = "wind_primary"
}

# 给最近目标施加虚弱
effect { type = "WEAKNESS" amp = 1 time = 80 targetVar = "wind_primary" }

# 给自己回血
effect { type = "REGENERATION" amp = 1 time = 100 target = "self" }
```



> 更新: 2025-10-17 14:13:15  
> 原文: <https://www.yuque.com/yuazer/blow95/fwplvopm3tx1ncv8>