# 🔵 cycle — 圆形路径采样语句

## 🧩 功能简介
`cycle` 用于在脚本中**生成圆形/扇形路径**，并将结果存入变量（`ctx.vars[store]`），供粒子、投掷物、伤害、位移等语句沿路径使用。  
支持 **world/local 坐标系**、不同平面（`xz/xy/yz`）、自定义角度与起始角度，还可选**收集路径附近实体**写入变量。

## ⚙️ 语法（不想看的可以直接往下拉，看示例即可）
| 参数名             | 类型      | 必填 | 默认值     | 说明                                               |
|-----------------|---------|----|---------|--------------------------------------------------|
| `store`         | String  | ✅  | —       | 结果变量名，存入 `ctx.vars[store]`（类型为 `List<Location>`） |
| `at`            | String  | ❌  | `@self` | 圆心位置，支持 `@self` / `${var}` / `world:x,y,z`       |
| `radius`        | Double  | ❌  | `1.0`   | 圆半径                                              |
| `points`        | Int     | ❌  | `32`    | 采样点数量                                            |
| `startAngle`    | Double  | ❌  | `0`     | 起始角度（度）                                          |
| `angle`         | Double  | ❌  | `360`   | 绘制角度（度，非 360 即扇形）                                |
| `coord`         | String  | ❌  | `world` | 坐标系：`world` 或 `local`                            |
| `plane`         | String  | ❌  | `xz`    | 平面：`xz` / `xy` / `yz`                            |
| `facing`        | String  | ❌  | `self`  | `coord=local` 时朝向参照：`self` / `target`            |
| `storeEntities` | String  | ❌  | —       | 把路径附近实体集合写入变量（`List<Entity>`）                    |
| `entityRadius`  | Double  | ❌  | `0.6`   | 收集实体的半径                                          |
| `includeSelf`   | Boolean | ❌  | `false` | 是否包含施法者自己                                        |

---

## ⚡ 快速示例

### ① 基础圆形路径 + 粒子播放
```plain
cycle {
  store = "circlePath"
  at = "@self"
  radius = 2.5
  points = 48
  startAngle = 0
  angle = 360
  coord = "world"
  plane = "xz"
}

particle {
  type = "FLAME"
  count = 1
  path = "circlePath"
  ticks-per-point = 1
}
```

### ② 面向朝向的扇形路径（local + arc）
```plain
cycle {
  store = "fanPath"
  at = "@self"
  radius = 3.0
  points = 36
  startAngle = -60
  angle = 120
  coord = "local"
  plane = "xz"
  facing = "self"
}
```

### ③ 记录路径经过的实体集合
```plain
cycle {
  store = "scanPath"
  at = "@self"
  radius = 2.0
  points = 24
  angle = 360
  storeEntities = "cycleHits"
  entityRadius = 0.8
  includeSelf = false
}

# cycleHits 为 List<Entity>
damage { targetVar = "cycleHits"; amount = 6.0 }
```

---

> 更新: 2025-12-18 12:20:00  
> 原文: —
