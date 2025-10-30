# 🎯 projectile — 发射投掷物/弹道实体

从指定位置发射一个原版投掷物或弹道实体（如雪球、箭、三叉戟、火球等），可自定义射手、速度/方向、重力、随机散布、以及把生成的实体存入变量。

---

## 参数一览

| 参数名               | 类型          | 必填 | 默认值        | 说明                                             |
|-------------------|-------------|----|------------|------------------------------------------------|
| `type`            | String      | 否  | `snowball` | 投掷物/弹道实体类型，见下方支持列表                             |
| `at`              | LocationStr | 否  | `@self`    | 发射位置，遵循 `ctx.resolveLocation(...)` 解析规则        |
| `shooter`         | String      | 否  | `self`     | 射手选择：`self` / `target` / `none` / `var:xxx`    |
| `shooterVar`      | String      | 否  | —          | 直接从变量读取射手（变量里需为 `Entity` 或 `ProjectileSource`） |
| `towards`         | LocationStr | 否  | —          | 朝该位置飞行；若未提供，将使用射手朝向（或 `source` 朝向）             |
| `vx`/`vy`/`vz`    | Double      | 否  | —          | 直接指定速度向量，若提供则覆盖 `towards` 方向计算                 |
| `speed`           | Double      | 否  | `1.0`      | 方向单位向量的倍率，最终速度通常为 `dir.normalize() * speed`    |
| `spread`          | Double      | 否  | `0.0`      | 高斯随机散布（越大越发散）                                  |
| `inheritVelocity` | Boolean     | 否  | `false`    | 是否叠加射手当前速度向量到弹道上                               |
| `gravity`         | Boolean     | 否  | —          | 是否受重力（部分投掷物可修改）                                |
| `store`           | String      | 否  | —          | 将创建的投掷物存入 `ctx.vars[store]`                    |
| `name`            | String      | 否  | —          | 自定义显示名（支持 `&` 颜色符），并显示在头顶                      |
| `angleX`          | Double      | 否  | `0.0`      | 绕 X 轴旋转角（度），抬头/俯仰方向微调                           |
| `angleY`          | Double      | 否  | `0.0`      | 绕 Y 轴旋转角（度），左右偏航方向微调                           |
| `angleZ`          | Double      | 否  | `0.0`      | 绕 Z 轴旋转角（度），滚转/平面内旋转微调                         |
| `angleMode`       | String      | 否  | `relative` | 角度模式：`relative` 相对朝向；`absolute` 世界坐标                 |

> 解析顺序提示：速度优先级为 手动向量(`vx/vy/vz`) > `towards` > 射手/施法者朝向；随后应用角度(`angle*`)，再乘 `speed` 并叠加 `spread` 与 `inheritVelocity`。

---

## 快速示例

```plain
projectile {
  type = "snowball"     # 发射雪球
  at = "@self"          # 从自己位置发射
  shooter = "self"      # 射手是自己
  towards = "@target"   # 朝目标位置飞
  speed = 1.8            # 速度
  spread = 0.05          # 微小随机偏移
  inheritVelocity = false
  gravity = true
  store = "lastProjectile"
  name = "&b冰弹"
  angleY = 10            # 轻微向右偏转 10°（相对模式）
  angleMode = "relative"
}
```

---

## 用法详解

### 1) 由向量直接控制轨迹（覆盖 towards）
```plain
projectile { type = "arrow" vx = 0 vy = 0.3 vz = 2 speed = 1.0 }
```

### 2) 使用射手朝向开火
```plain
projectile { type = "trident" shooter = "self" speed = 2.2 }
```

### 3) 指定位置 + 指定目标点（炮台/陷阱）
```plain
projectile { type = "fireball" at = "${towerPos}" towards = "${enemyPos}" speed = 0.8 }
```

### 4) 从变量里取射手（如盔甲架或召唤物）
```plain
# 先把某个实体存入 vars.shooter
projectile { type = "egg" shooterVar = "shooter" speed = 1.2 }
```

### 5) 继承射手速度 + 轻微散布
```plain
projectile { type = "arrow" inheritVelocity = true spread = 0.02 speed = 2.0 }
```

### 6) 角度旋转：相对/绝对
```plain
# 相对射手朝向（默认）：在当前朝向基础上再旋转
projectile { type = "trident" angleY = -15 angleX = 5 angleMode = "relative" speed = 2.0 }

# 绝对世界坐标：绕世界 X/Y/Z 轴旋转，忽略射手朝向
projectile { type = "fireball" vx = 0 vy = 0 vz = 1 angleY = 45 angleMode = "absolute" speed = 1.0 }
```

### 7) 存储投掷物并后续操作
```plain
projectile { type = "snowball" store = "lastProjectile" }
message { text = "&7已发射: ${lastProjectile}" }
```

---

## 支持的 `type` 列表（常用）

- `snowball`
- `egg`
- `ender_pearl`
- `experience_bottle` / `exp_bottle` / `expbottle`
- `potion` / `splash_potion` / `lingering_potion`
- `arrow` / `spectral_arrow` / `tipped_arrow`
- `trident`
- `fireball` / `large_fireball` / `small_fireball` / `dragon_fireball`
- `wither_skull`
- `shulker_bullet`
- `llama_spit`

> 名称大小写不敏感；支持下划线形式（如 `spectral_arrow`）。

---

## 进阶技巧

- 使用 `spread` 制作霰弹/散射效果；配合 `random` 可以做随机变体。
- 用 `inheritVelocity` 实现“奔跑射击”带速度叠加的更自然弹道。
- 通过 `store` 把实例写入变量，后续可结合自定义监听/别的语句进行跟踪或回收。
- `gravity` 并非对所有投掷物都可控，依赖服务端与实体实现。

---

## ⚠️ 注意事项

- 若无法解析 `at`，或世界为空，语句会被跳过。
- `vx/vy/vz` 与 `towards` 同时存在时，以 `vx/vy/vz` 为准。
- 若最终计算出的速度向量无效（零向量或 NaN），将不会设置速度。
- `angleX/angleY/angleZ` 会对最终方向应用旋转：无论该方向来源于 `vx/vy/vz`、`towards` 或射手朝向。
- `angleMode = relative` 时以射手/来源朝向为基准旋转；`absolute` 时以世界坐标轴为基准旋转。

---

## 🔗 相关语句

- [🎯 target-entity - 选取实体](target-entity—范围选取实体_写入上下文变量.md)
- [🔊 sound - 音效播放](sound—播放音效.md)
- [🌌 particle - 粒子特效](particle—粒子特效语句.md)
- [🧪 effect - 药水效果](effect—施加药水效果.md)

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-30 12:00:00</p>
</div>


