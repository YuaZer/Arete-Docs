# 🌌 particle — 粒子特效语句（最新版）

> 更新：2025-11-12 03:16:04
>
> 适配代码：`ParticleStatement`（支持 **路径快照** 与 **流式路径** 两种模式，新增完成标记与空闲自动停止）

## 简介
`particle` 语句用于在指定位置或沿着一条路径生成粒子效果。  
这是 **视觉特效类技能** 的核心组件之一，可用于：

- 展示技能轨迹、爆炸、冲击波、法阵等效果；
- 制作炫酷的技能动画（如“陨石坠落”“能量波动”等）；
- 增强玩家动作反馈与临场感。

你可以通过调整粒子类型、偏移量、数量和速度，**在单点** 或 **沿路径** 播放粒子，组合出丰富的视觉表现。

粒子种类请参考 Bukkit 文档：<https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Particle.html>

---

## 🆕 变更概览（相对旧文档）
- ✅ **支持两种路径模式**：
    - `SNAPSHOT`：一次性或等间隔回放**固定列表**的路径点。
    - `STREAM`：**动态跟随**可变列表（`MutableList<Location>`），适合“能量弹道/导弹”这类不断追加新点的场景。
- ✅ **新增节奏控制**：`ticks-per-point` / `point-interval`（兼容旧字段 `interval`）。
- ✅ **新增完成标记**：`pathDoneVar`（默认名：`<path>#done`）。
- ✅ **新增空闲停止**：`pathIdleStopTicks`（流式模式下，长时间无新增点自动停止）。
- ✅ **更强的参数兼容**：`mode`/`playback`/`stream`/`live`/`follow` 等别名自动识别。

---

## ⚙️ 语法

```plain
particle {
    type = "FLAME"
    count = 50
    ox = 0.2
    oy = 0.6
    oz = 0.2
    speed = 0.05
    at = "@self"
}
```
在玩家位置周围生成火焰粒子，模拟燃烧的效果。

---

## 参数说明

### A. 通用参数
| 参数名        | 类型     | 必填 |           默认值 | 说明                                                                  |
|------------|--------|---:|--------------:|---------------------------------------------------------------------|
| `type`     | String |  ✅ |             — | 粒子类型（`FLAME`、`END_ROD` 等）                                           |
| `count`    | Int    |  ❌ |          `20` | 粒子数量（Bukkit `count`）                                                |
| `ox/oy/oz` | Double |  ❌ | `0.2/0.2/0.2` | 偏移量（Bukkit offsetX/Y/Z）                                             |
| `speed`    | Double |  ❌ |         `0.0` | 速度（Bukkit `speed`）                                                  |
| `at`       | String |  ❌ |       `@self` | **单点模式** 的坐标；支持 `@self / @target / ${var} / world:x,y,z / ~x,~y,~z` |

> 行为优先级：当判定为**路径模式**（见下表）时优先沿路径播放；否则**回退单点模式**（读取 `at`）。

### B. 路径相关参数
| 参数名                               | 类型     | 必填 |           默认值 | 说明                                                                                                |
|-----------------------------------|--------|---:|--------------:|---------------------------------------------------------------------------------------------------|
| `path`                            | String |  ❌ |             — | **路径变量名**。要求：`List<Location>`（快照） 或 `MutableList<Location>`（流式更优）                                 |
| `step`                            | Int    |  ❌ |           `1` | **抽样步长**：每 `step` 个点播放 1 次（≥1）                                                                    |
| `ticks-per-point`                 | Int    |  ❌ |           `0` | **每点间隔**（tick）。`0`=同 tick 一次性刷出全部（快照模式）                                                           |
| `mode` / `path-mode` / `playback` | String |  ❌ |    `SNAPSHOT` | `SNAPSHOT` / `STREAM`（别名：`stream`/`live`/`follow`→`STREAM`；`snapshot`/`static`/`once`→`SNAPSHOT`） |
| `stream` / `live` / `follow`（布尔）  | Bool   |  ❌ |       `false` | 任意为 `true` 时强制 `STREAM`                                                                           |
| `pathDoneVar`                     | String |  ❌ | `<path>#done` | **完成标记变量名**（布尔）。仅 `STREAM` 模式有效                                                                   |
| `pathIdleStopTicks`               | Int    |  ❌ |          `20` | 流式模式下**空闲自动停止**阈值（tick）。`≤0` 表示不因空闲而停止                                                            |

### C. 兼容字段（自动识别）
- `ticks-per-point` ≡ `tick-per-point` ≡ `point-interval` ≡ **`interval`（旧）**
- `mode` ≡ `path-mode` ≡ `playback`
- `stream` ≡ `live` ≡ `follow`（任意为真 → `STREAM`）
- `pathDoneVar` ≡ `pathDone` ≡ `pathCompleteVar` ≡ `pathComplete`
- `pathIdleStopTicks` ≡ `pathIdleTicks` ≡ `idleTicks` ≡ `idle-ticks` ≡ `pathIdle`

---

## ⛓️ 模式判定与行为

### 1) 路径快照（`SNAPSHOT`）
- 读取 `path` 变量中的 **固定点集**（`List<Location>` / 数组）。
- `ticks-per-point = 0`：**同一 tick** 按步长 (`step`) 一次性刷出整条轨迹；末尾点会**尽量补一次**以保证渲染闭合感。
- `ticks-per-point > 0`：以固定节奏逐点播放（每点相隔 `ticks-per-point` tick）。

### 2) 流式路径（`STREAM`）
- 读取 `path` 变量中的 **可变列表**（`MutableList<Location>`），每个调度周期**抓取新增点**并按 `step` 输出。
- 如无 `MutableList`，会**回退**到 `SNAPSHOT`（若 `path` 为不可变列表）。
- 终止条件：
    - `pathDoneVar`（默认 `<path>#done`）为 `true` 且已播放至末尾，或
    - 连续空闲 `pathIdleStopTicks` tick 未发现新点（`≤0` 表示忽略此条件）。
- 终止前会尝试**补播最后一个点**，避免出现轨迹“断尾”。

---

## 🧭 `at` 参数速查
| 示例 | 说明 |
|---|---|
| `@self` | 施法者位置 |
| `@target` | 当前目标实体位置 |
| `${aimPos}` | 变量中的 `Location` |
| `100,65,-20` | 世界绝对坐标 |
| `~0,1,~0` | 相对施法者位置（上方 1 格） |

---

## 示例

### 🔥 基础：玩家周围的火焰
```plain
particle {
  type = "FLAME"
  count = 80
  ox = 0.3
  oy = 1.0
  oz = 0.3
  speed = 0.05
  at = "@self"
}
```

### 💥 命中点爆炸
```plain
particle {
  type = "EXPLOSION_LARGE"
  count = 5
  at = "${impactPos}"
}
```

### 🌈 闪光轨迹（配合 `seq`）
```plain
seq {
  particle { type = "END_ROD"; count = 10; at = "@self" }
  delay { ticks = 5 } {
    particle { type = "END_ROD"; count = 10; at = "@target" }
  }
}
```

### ⚡ 并行动画
```plain
parallel {
  particle { type = "CRIT"; count = 20; at = "@self" }
  particle { type = "SMOKE_NORMAL"; count = 10; at = "@self" }
}
```

---

## 🧩 路径用法

### 1) 快照：一次性“刷轨迹”（同 tick）
```plain
# 假设 curve 为 List<Location>
particle {
  type = "END_ROD"
  count = 1
  path = "curve"
  step = 1
  ticks-per-point = 0   # 同 tick 一次性播完
}
```

### 2) 快照：逐点“流动”（固定节奏）
```plain
particle {
  type = "FLAME"
  count = 1
  path = "curve"
  step = 1
  ticks-per-point = 1   # 每点间隔 1 tick
}
```

### 3) 抽样降载（每 3 个点取 1 个）
```plain
particle {
  type = "CRIT"
  count = 1
  path = "curve"
  step = 3
  ticks-per-point = 0
}
```

### 4) 流式：能量弹道（动态追加点）
```plain
# 要求：ctx.vars["beam"] 是 MutableList<Location>
# 当弹道计算线程结束时，设置 ctx.vars["beam#done"] = true
particle {
  type = "END_ROD"
  count = 1
  path = "beam"
  step = 1
  mode = "STREAM"
  ticks-per-point = 1
  # 可选：自定义完成标记名
  # pathDoneVar = "beam#done"
  # 可选：空闲自动停止
  pathIdleStopTicks = 40
}
```

### 5) 轨迹 + 末端爆点（混用）
```plain
seq {
  # 先沿路径播放
  particle { type = "END_ROD"; count = 1; path = "beam"; step = 1; ticks-per-point = 1 }

  # 路径末端再爆点（假设有 ${beam.last} 能取到末尾点）
  particle { type = "EXPLOSION_NORMAL"; count = 8; at = "${beam.last}" }
}
```

---

## 🔗 与 `nurbs` 组合

### A. 世界坐标曲线 + 流动粒子
```plain
seq {
  nurbs {
    store = "curve"
    coord = "world"
    at = "@self"
    points = "-4,4,2; -2,10,-2; 3,10,1; 8,4,4"
    degree = 3
    precision = 1.2
    equalSpacing = true
    spacing = 0.25
  }
  delay { ticks = 2 }    # nurbs 异步计算，等 1~2 tick
  particle {
    type = "FLAME"
    count = 2
    path = "curve"
    step = 1
    ticks-per-point = 1
  }
}
```

### B. 本地坐标曲线 + 一次性能量弧
```plain
seq {
  nurbs {
    store = "arc"
    coord = "local"
    at = "@self"
    points = "0,0,0; 2,0.6,0.5; 5,0.8,1.2; 8,0.2,1.8"
    precision = 1.5
    equalSpacing = true
    spacing = 0.2
  }
  delay { ticks = 1 }
  particle { type = "SPELL_WITCH"; count = 1; path = "arc"; ticks-per-point = 0 }
}
```

### C. 轨迹 + 末端爆点
```plain
seq {
  nurbs { store = "beam"; coord = "local"; at = "@self"; points = "0,0,0; 4,0.2,0; 8,0,0" }
  delay { ticks = 1 }

  particle { type = "END_ROD"; count = 1; path = "beam"; step = 1; ticks-per-point = 1 }
  particle { type = "EXPLOSION_NORMAL"; count = 8; at = "${beam.last}" }
}
```
## 示例：能量弹道
```yaml
# 发射能量模式的虚拟子弹，并把轨迹列表存入 energyPath
    projectile {
      bulletType = "energy"
      type = "snowball"
      shooter = "self"
      speed = 2.8
      spread = 0.0
      gravity = false
      store = "energyPath"
      damage = 6.0
      autoRemoveDelayTicks = 40
      energyPeriod = 1
    }
    # 使用粒子语句直接追踪 energyPath 的实时轨迹
    # path-mode = "stream" 会在路径列表新增点时持续渲染
    particle {
      type = "FLAME"
      count = 8
      ox = 0.06
      oy = 0.06
      oz = 0.06
      speed = 0.0
      path = "energyPath"
      path-mode = "stream"
      step = 1
      ticks-per-point = 1
      pathIdleTicks = 10
    }
```
---

## 📎 注意事项
- `ticks-per-point = 0` 时为**同 tick 刷轨迹**，适合瞬时“能量弧/轨迹残影”；若点数很多，建议提高 `step` 以避免单 tick 压力过大。
- `STREAM` 模式下优先使用 `MutableList<Location>`；若只能提供 `List<Location>`，会回退为 `SNAPSHOT`。
- `STREAM` 模式下请在弹道计算完成时**设置完成标记**（默认 `<path>#done`）以便尽快停止；否则依赖空闲超时停止。
- 所有粒子实际播放均在主线程完成（内部已调度到主线程）。

---

## 版本信息
- 语句：`particle`
- 支持模式：`SNAPSHOT` / `STREAM`
- 兼容字段：`interval` 等均已向前兼容
- 对应实现：`ParticleStatement`（Arete 内置）
