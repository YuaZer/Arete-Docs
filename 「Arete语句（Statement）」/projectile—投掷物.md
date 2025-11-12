# 🎯 projectile — 发射投掷物 / 弹道实体（最新版）

> 更新：2025-11-12 03:23:56
>
> 适配实现：`ProjectileStatement`（支持 **原版投掷物** 与 **能量子弹（虚拟弹道）** 两种模式，新增穿透、命中延迟、能量弹路径存储与完成标记、无动画伤害等）

从指定位置发射一个原版投掷物或弹道实体（如雪球、箭、三叉戟、火球等），可自定义射手、速度/方向、重力、随机散布、角度、穿透与移除策略；并可选择**能量子弹**模式以虚拟化弹道（不生成 Bukkit 实体，用路径点驱动视觉/命中）。

---

## 参数一览

### 通用
| 参数名               | 类型          | 必填 |          默认值 | 说明                                                                        |
|-------------------|-------------|---:|-------------:|---------------------------------------------------------------------------|
| `type`            | String      |  否 |   `snowball` | 投掷物/弹道实体类型，见下表                                                            |
| `bulletType`      | String      |  否 | `projectile` | `projectile` 原版投掷物；`energy` 能量子弹（虚拟弹道，不生成实体）                              |
| `at`              | LocationStr |  否 |      `@self` | 发射位置（解析遵循 `ctx.resolveLocation`），未写或 `@self` 且射手为玩家时，默认用**眼睛位置前方**一点，避免自撞 |
| `shooter`         | String      |  否 |       `self` | 射手：`self` / `target` / `none` / `var.xxx`                                 |
| `shooterVar`      | String      |  否 |            — | 从变量中直接读取射手对象（`Entity` 或 `ProjectileSource`）                               |
| `towards`         | LocationStr |  否 |            — | 朝该位置飞行；若未提供，使用射手/来源朝向                                                     |
| `vx`/`vy`/`vz`    | Double      |  否 |            — | 直接指定速度向量（优先级最高，覆盖 `towards`/朝向）                                           |
| `speed`           | Double      |  否 |        `1.0` | 方向单位向量的倍乘                                                                 |
| `spread`          | Double      |  否 |        `0.0` | 高斯散布（越大越发散）                                                               |
| `inheritVelocity` | Boolean     |  否 |      `false` | 叠加射手当前速度向量                                                                |
| `gravity`         | Boolean     |  否 |            — | 是否受重力（仅某些实体可控）                                                            |
| `name`            | String      |  否 |            — | 自定义显示名（`&` 颜色），显示在头顶                                                      |
| `angleX/Y/Z`      | Double      |  否 |      `0/0/0` | 角度微调（度）                                                                   |
| `angleMode`       | String      |  否 |   `relative` | `relative` 相对射手朝向；`absolute` 世界坐标轴旋转                                      |
| `store`           | String      |  否 |            — | 存储变量名：`projectile` 模式下存实体；`energy` 模式下存**路径 List<Location>**              |

> 速度解析优先级：`vx/vy/vz` ＞ `towards` ＞ 射手/来源朝向 → 再应用角度 → 乘 `speed` → 应用 `spread` / `inheritVelocity`。

### 命中/移除与伤害
| 参数名                    | 类型      | 必填 |           默认值 | 说明                                                          |
|------------------------|---------|---:|--------------:|-------------------------------------------------------------|
| `damage`               | Double  |  否 |             — | 自定义命中伤害；不填则遵循原版/事件侧逻辑                                       |
| `damageType`           | String  |  否 |      `normal` | `normal` 走 `Entity#damage`（有动画/无敌帧）；其他值走**无动画纯扣血**（仍派发伤害事件） |
| `hitStoreEntity`       | String  |  否 | `pjHitEntity` | 命中实体写入变量（仅非空写入）                                             |
| `hitStorePos`          | String  |  否 |    `pjHitPos` | 命中瞬间/移除前的位置写入变量（仅命中实体时写入）                                   |
| `hitRemoveDelayTicks`  | Int     |  否 |           `0` | 命中后延迟移除（tick）                                               |
| `autoRemoveDelayTicks` | Int     |  否 |          `40` | 兜底自动移除（tick）；`energy` 模式用于推算最大飞行时长/距离                       |
| `removeAtBlock`        | Boolean |  否 |        `true` | 命中方块是否移除（`projectile` 模式下）                                  |

### 穿透（箭类）
| 参数名      | 类型  | 必填 | 默认值 | 说明                                                   |
|----------|-----|---:|----:|------------------------------------------------------|
| `pierce` | Int |  否 | `0` | 仅对 `AbstractArrow` 生效；>0 时**命中实体不再自动移除**（命中方块或兜底才移除） |

### 能量子弹（虚拟弹道）
| 参数名            | 类型     | 必填 |            默认值 | 说明                                                              |
|----------------|--------|---:|---------------:|-----------------------------------------------------------------|
| `energyPeriod` | Int    |  否 |            `1` | 位置推进的调度周期（tick）                                                 |
| `pathDoneVar`  | String |  否 | `<store>#done` | 完成标记变量名；若未显式提供且 `store` 存在，默认使用 `store + "#done"`；流程结束写入 `true` |

---

## 支持的 `type`（常用）
`snowball`, `egg`, `ender_pearl`, `experience_bottle`/`exp_bottle`/`expbottle`,  
`potion`/`splash_potion`/`lingering_potion`, `arrow`/`spectral_arrow`/`tipped_arrow`,  
`trident`, `fireball`/`large_fireball`/`small_fireball`/`dragon_fireball`,  
`wither_skull`, `shulker_bullet`, `llama_spit`

> 名称大小写不敏感；支持下划线写法。

---

## 快速示例

### 1) 雪球朝目标飞行
```plain
projectile {
  type = "snowball"
  at = "@self"
  shooter = "self"
  towards = "@target"
  speed = 1.8
  spread = 0.05
  inheritVelocity = false
  gravity = true
  store = "lastProjectile"
  name = "&b冰弹"
  angleY = 10
  angleMode = "relative"
}
```

### 2) 直接向量 + 绝对角度
```plain
projectile { type = "fireball" vx = 0 vy = 0.1 vz = 1 speed = 1.0 angleY = 45 angleMode = "absolute" }
```

### 3) 穿透箭（命中实体不移除，仅命中方块或兜底移除）
```plain
projectile { type = "arrow" speed = 2.2 pierce = 2 autoRemoveDelayTicks = 80 }
```

### 4) 能量子弹（虚拟弹道）+ 路径粒子
```plain
parallel {
  # A. 能量弹：不生成实体，按速度推进位置，记录路径点，命中第一个 LivingEntity 后结束
  projectile {
    bulletType = "energy"
    type = "small_fireball"   # 仅作语义，不生成实体
    at = "@self"
    towards = "@target"
    speed = 0.6
    spread = 0.0
    store = "beam"            # -> ctx.vars["beam"] = MutableList<Location>
    pathDoneVar = "beam#done" # 可不写，默认也是 beam#done
    autoRemoveDelayTicks = 60 # 最大飞行步数推导用
    energyPeriod = 1
    damage = 6.0
    damageType = "pure"       # 无动画纯扣血（仍派发伤害事件）
  }

  # B. 粒子沿路径播放（示例选用 SNAPSHOT 或 STREAM 均可）
  particle {
    type = "END_ROD"
    count = 1
    path = "beam"
    step = 1
    ticks-per-point = 1
    # STREAM 模式可在粒子语句里按需设置
    # mode = "STREAM"
  }
}
```

### 5) 存变量方便后续操作
```plain
projectile { type = "snowball" store = "lastProjectile" }
message { text = "&7已发射: ${lastProjectile}" }
```

---

## 事件钩子（供脚本/插件扩展）

- **`AreteProjectileLaunchEvent`**
    - 字段：`projectile`、`shooter`、`projectileSource`、`velocity`、`damage`、`autoRemoveOnHit`
    - 可取消；可修改 `velocity` / `damage` / `autoRemoveOnHit`
- **`AreteProjectileHitEvent`**
    - 字段：`projectile`、`shooter`、`projectileSource`、`hitEntity`、`hitBlock`、`damage`、`removeProjectile`、`bukkitEvent`
    - 可取消；可修改 `damage`（命中时实际生效）与 `removeProjectile`

> `energy` 模式命中采用区域检测，仅处理 `LivingEntity`（“穿墙”命中，不检测方块）。

---

## 用法细节与优先级

1. **速度来源优先级**：`vx/vy/vz` ＞ `towards` ＞ 射手/来源朝向。
2. **角度应用**：
    - `relative`：在用 yaw/pitch 重建方向时应用 `angleX/angleY`；`angleZ` 不参与。
    - `absolute`：对世界向量逐轴旋转 `angleX/angleY/angleZ`。
3. **继承速度**：`inheritVelocity = true` 时叠加射手当前速度（更自然的奔跑射击）。
4. **重力**：并非所有投掷物都可控，实际取决于实体实现与服务端。
5. **穿透**：`pierce > 0` 时命中实体不移除，仅命中方块或到兜底计时移除。
6. **命中写变量**：仅在**命中实体**时才写入 `hitStoreEntity` / `hitStorePos`。
7. **能量子弹**：
    - 不生成实体；每 `energyPeriod` tick 用速度推进位置。
    - 轨迹写入 `store`（`MutableList<Location>`）。
    - 完成时写 `pathDoneVar = true`（默认 `<store>#done`）。
    - `autoRemoveDelayTicks` 用作最大步数/距离估计；命中第一个 `LivingEntity` 时提前结束。

---

## 相关语句
- `target-entity` — 选取实体
- `sound` — 播放音效
- `particle` — 粒子特效（可沿能量弹路径播放）
- `effect` — 施加药水效果

---

## 注意事项
- 当 `at` 无法解析或世界为空时，语句跳过执行。
- 若最终速度向量为零或 NaN，将不会设置速度/推进位置。
- `vx/vy/vz` 与 `towards` 同时存在时，以向量为准。
- 事件可能取消或修改参数；注意监听器生命周期与并发。
- 在 `projectile` 模式中，仍会有原版的碰撞箱/物理与无敌帧影响。

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-11-12 03:23:56</p>
</div>
