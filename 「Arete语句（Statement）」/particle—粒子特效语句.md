# 🌌 particle — 粒子特效语句（最新版）

> 更新时间：2025-12-18  
> 适配代码：`ParticleStatement`
> - 支持 **路径快照（SNAPSHOT）** 与 **流式路径（STREAM）** 两种模式
> - 支持 **带 data 粒子**（如可调色 REDSTONE / DUST_COLOR_TRANSITION、SCULK_CHARGE、SHRIEK 等）

---

## 一、简介

`particle` 语句用于在指定位置或沿着一条路径生成粒子效果。  
这是 **视觉特效类技能** 的核心组件之一，可用于：

- 展示技能轨迹、爆炸、冲击波、法阵等效果；
- 制作炫酷的技能动画（如“陨石坠落”“能量波动”“能量弹道”）；
- 增强玩家动作反馈与临场感。

你可以通过调整粒子类型、偏移量、数量、速度以及 **颜色 / 特殊 data**，在**单点**或**沿路径**播放粒子，组合出丰富的视觉表现。

粒子种类请参考 Bukkit 文档：<https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Particle.html>

---

## 二、核心能力概览

### ✅ 基础能力

- 在任意坐标（`at`）生成粒子
- 支持粒子数量（`count`）、偏移量（`ox/oy/oz`）、速度（`speed`）等基础参数
- 支持 DSL 变量坐标（`${var}`）、选择器（`@self/@target`）等

### ✅ 路径能力

- **SNAPSHOT 模式**：一次性或按固定节奏回放 **固定路径点列表**（`List<Location>`）。
- **STREAM 模式**：实时跟随 **可变路径列表**（`MutableList<Location>`），适合“能量弹道/导弹”等动态轨迹。

### ✅ 粒子 data 能力

通过 `createParticleData` 自动解析 DSL，当前支持：

| 粒子类型                        | Data 类型                     | 用途说明                         |
|-----------------------------|------------------------------|----------------------------------|
| `REDSTONE`（命名空间 `dust`） | `Particle.DustOptions`       | 可调色红石粒子 / 能量粒子基础色         |
| `DUST_COLOR_TRANSITION`     | `Particle.DustTransition`    | 从一个颜色渐变到另一个颜色的能量轨迹      |
| `SCULK_CHARGE`              | `Float`                      | 诡异脉冲强度                      |
| `SHRIEK`                    | `Int`                        | 尖啸阶段 / 延迟参数                  |

---

## 三、语法概览

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

在玩家位置周围生成火焰粒子，模拟燃烧效果。

---

## 四、参数说明

### A. 通用参数

| 参数名        | 类型     | 必填 |           默认值 | 说明                                                                       |
|------------|--------|---:|--------------:|--------------------------------------------------------------------------|
| `type`     | String |  ✅ |             — | 粒子类型（`FLAME`、`END_ROD`、`REDSTONE` 等，等同于 `Particle.valueOf(...)`）         |
| `count`    | Int    |  ❌ |          `20` | 粒子数量（Bukkit `count`）                                                     |
| `ox/oy/oz` | Double |  ❌ | `0.2/0.2/0.2` | 偏移量（Bukkit offsetX/Y/Z）                                                  |
| `speed`    | Double |  ❌ |         `0.0` | 速度（Bukkit `speed`）                                                       |
| `at`       | String |  ❌ |       `@self` | **单点模式** 的坐标解析表达式；支持 `@self / @target / ${var} / world:x,y,z / ~x,~y,~z` |

> 如果配置了 `path` 且能解析为路径，将优先走 **路径模式**；否则退回到 **单点模式**，使用 `at` 计算坐标。
> `count/ox/oy/oz/speed` 支持 `${变量}` / `vars.xxx` / 表达式。

---

### B. 路径相关参数

| 参数名                               | 类型     | 必填 |           默认值 | 说明                                                                                                |
|-----------------------------------|--------|---:|--------------:|---------------------------------------------------------------------------------------------------|
| `path`                            | String |  ❌ |             — | **路径变量名**。要求：`List<Location>`（快照）或 `MutableList<Location>`（流式优先）                                  |
| `step`                            | Int    |  ❌ |           `1` | **抽样步长**：每 `step` 个点播放 1 次（≥1）                                                                    |
| `ticks-per-point`                 | Int    |  ❌ |           `0` | **每点间隔**（tick）。`0` = 同 tick 一次性刷出全部（快照模式）                                                         |
| `mode` / `path-mode` / `playback` | String |  ❌ |    `SNAPSHOT` | `SNAPSHOT` / `STREAM`（别名：`stream`/`live`/`follow`→`STREAM`；`snapshot`/`static`/`once`→`SNAPSHOT`） |
| `stream` / `live` / `follow`      | Bool   |  ❌ |       `false` | 任意为 `true` 时强制 `STREAM` 模式                                                                        |
| `pathDoneVar`                     | String |  ❌ | `<path>#done` | **完成标记变量名**（布尔）。仅 `STREAM` 模式有效                                                                   |
| `pathIdleStopTicks`               | Int    |  ❌ |          `20` | 流式模式下**空闲自动停止**阈值（tick）。`≤0` 表示不因空闲而停止                                                            |

#### 兼容字段（自动识别）

- 间隔：`ticks-per-point` ≡ `tick-per-point` ≡ `point-interval` ≡ **`interval`（旧字段）**
- 模式：`mode` ≡ `path-mode` ≡ `playback`
- 流式标记：`stream` ≡ `live` ≡ `follow`（任意为真 → `STREAM`）
- 完成标记：`pathDoneVar` ≡ `pathDone` ≡ `pathCompleteVar` ≡ `pathComplete`
- 空闲停止：`pathIdleStopTicks` ≡ `pathIdleTicks` ≡ `idleTicks` ≡ `idle-ticks` ≡ `pathIdle`
  
> `path/step/ticks-per-point/pathIdleStopTicks` 支持 `${变量}` / `vars.xxx` / 表达式。

---

### C. 带 data 粒子参数（颜色/强度/阶段）

`ParticleStatement` 会根据 `type` 自动决定是否需要 data，并从 DSL 中读取附加字段：

#### 1. 可调色粒子：`REDSTONE`（dust）

使用 Bukkit 的 `Particle.DustOptions`：

```kotlin
DustOptions(color: Color, size: Float)
```

DSL 支持三种写法（任选其一）：

- 独立 RGB：
    - `r` / `g` / `b`：0–255
- 颜色字符串：
    - `color: "#RRGGBB"`
    - `color: "RRGGBB"`
    - `color: "r,g,b"`

可选粒子大小：

- `size`: `Float`，默认 `1.0`（和 Bukkit 原生 DustOptions 一致）

**示例：红色能量球**

```plain
particle {
  type = "REDSTONE"
  r = 255
  g = 60
  b = 60
  size = 1.2
  count = 3
  ox = 0.03
  oy = 0.03
  oz = 0.03
  speed = 0.0
  at = "@self"
}
```

或：

```plain
particle {
  type = "REDSTONE"
  color = "#FF3C3C"
  size = 1.0
  at = "@self"
}
```

#### 2. 渐变色粒子：`DUST_COLOR_TRANSITION`

使用 Bukkit 的 `Particle.DustTransition`：

```kotlin
DustTransition(fromColor: Color, toColor: Color, size: Float)
```

- 起始颜色：复用 `parseColor` 规则（`color` / `r,g,b`）。
- 目标颜色：使用如下任意字段：
    - `toColor`
    - `toColour`
    - `targetColor`
    - `targetColour`
- 若目标颜色未指定或解析失败，则回退为与起始颜色相同（无视觉渐变，只是兼容写法）。

**示例：从红到蓝的能量轨迹**

```plain
particle {
  type = "DUST_COLOR_TRANSITION"
  # 起始色
  color = "255,0,0"
  # 目标色
  toColor = "0,0,255"
  size = 1.0
  path = "beam"
  step = 1
  ticks-per-point = 1
}
```

#### 3. 诡异脉冲：`SCULK_CHARGE`

内部 data 是一个 `Float` 值，表示脉冲强度：

- `charge`：优先使用
- `power`：备用字段

**示例：**

```plain
particle {
  type = "SCULK_CHARGE"
  charge = 1.0
  at = "@self"
}
```

#### 4. 尖啸：`SHRIEK`

内部 data 是一个 `Int` 值，表示阶段 / 延迟等参数：

- `stage`：优先使用
- `delay`：备用字段
- 若均未配置，默认 `0`

**示例：**

```plain
particle {
  type = "SHRIEK"
  stage = 2
  at = "@self"
}
```

---

## 五、模式行为详解

### 1) SNAPSHOT 模式

- 从 `path` 变量读取 **固定点集**（`List<Location>` / 数组）。
- 逻辑分支：

    - `ticks-per-point <= 0`：
        - 使用 `onMain { ... }` 同一 tick 内按 `step` 遍历所有点并播放粒子。
        - 如果最后一个点未被刚好采样到，会尝试**额外补一次最后一个点**，避免轨迹尾部缺口。

    - `ticks-per-point > 0`：
        - 使用 `submit(period = interval)` 在主线程定时任务中逐点播放。
        - 每次调用播放 1 个点（按 `step` 递增索引）。
        - 结束时同样尝试补播最后一个点后 `cancel()` 任务。

### 2) STREAM 模式

- 优先尝试将 `path` 变量解析为 `MutableList<Location>`：
    - 若成功：使用同一个 `MutableList`，每个 tick 抓取当前 `size`，按 `step` 播放**新增点**；
    - 若失败：退回尝试 `List<Location>` → `SNAPSHOT` 模式。
- 内部状态：

    - `nextIndex`：下一次要播放的点的下标；
    - `idleTicks`：连续“无新点”累计时长；
    - `lastSpawned`：最后一次实际播出的点。

- 停止条件：

    - `doneFlag`（默认为 `${path}#done` 或自定义的 `pathDoneVar`）为 `true` 且 `nextIndex >= size`；
    - 或 `idleTicks >= pathIdleStopTicks`（若配置 `≤0`，则忽略该条件）。

- 停止前会尝试补播最后一个点，并 `cancel()` 定时任务。

---

## 六、`at` 参数速查

| 示例          | 说明              |
|-------------|-----------------|
| `@self`     | 施法者位置           |
| `@target`   | 当前目标实体位置        |
| `${aimPos}` | 变量中的 `Location` |
| `100,65,-20`| 世界绝对坐标          |
| `~0,1,~0`   | 相对施法者位置（上方 1 格） |

> 实际解析逻辑由你的 `ExecutionContext.resolveLocation(at)` 决定，这里只说明常见约定。

---

## 七、基础示例

### 1) 玩家周围的火焰

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

### 2) 命中点爆炸

```plain
particle {
  type = "EXPLOSION_LARGE"
  count = 5
  at = "${impactPos}"
}
```

### 3) 闪光轨迹（配合 `seq`）

```plain
seq {
  particle { type = "END_ROD"; count = 10; at = "@self" }
  delay { ticks = 5 } {
    particle { type = "END_ROD"; count = 10; at = "@target" }
  }
}
```

### 4) 并行动画

```plain
parallel {
  particle { type = "CRIT"; count = 20; at = "@self" }
  particle { type = "SMOKE_NORMAL"; count = 10; at = "@self" }
}
```

---

## 八、路径用法示例

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

## 九、与 `nurbs` 组合

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

---

## 十、能量弹道完整示例

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

## 十一、注意事项

- `ticks-per-point = 0` 时为 **同 tick 刷轨迹**，适合瞬时“能量弧/轨迹残影”；若点数很多，建议提高 `step` 以减轻单 tick 压力。
- `STREAM` 模式下 **强烈建议** 使用 `MutableList<Location>` 存放路径点；否则只会回退为快照播放。
- `STREAM` 模式下，请在弹道计算结束时设置 **完成标记**（默认 `<path>#done` 或自定义 `pathDoneVar`），避免任务长时间挂起。
- 所有粒子实际播放均在主线程完成（内部已通过 `onMain` / `submit(async = false)` 跳转到主线程），请避免在极短间隔内生成大量复杂路径以免卡顿。
- 粒子 `type` 是通过 `Particle.valueOf(type.uppercase())` 解析的，写错会回退到默认值 `CLOUD`。

---

## 十二、版本信息

- 语句名：`particle`
- 支持模式：`SNAPSHOT` / `STREAM`
- 带 data 粒子：`REDSTONE`、`DUST_COLOR_TRANSITION`、`SCULK_CHARGE`、`SHRIEK`
- 兼容字段：`interval` / `tick-per-point` / `point-interval` / `idleTicks` 等
- 对应实现：`io.github.yuazer.arete.builtin.ParticleStatement`
