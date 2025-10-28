# 🍞 hunger-cost — 扣除饱食度

扣除玩家的饱食度（Food Level）。仅对 `Player` 生效，通过直接修改 `foodLevel` 来实现。

---

## 参数一览

| 参数       | 必填 | 类型     | 默认       | 说明                          |
|----------|----|--------|----------|-----------------------------|
| `amount` | ✔  | Int    | —        | 要扣除的饱食度（1 = 半格鸡腿，20 = 满饱食度） |
| `target` | ✖  | String | `"self"` | 作用对象：`self`（默认）/ `target`   |
| `min`    | ✖  | Int    | `0`      | 保底值，扣完后不能低于这个饱食度            |

**核心逻辑：**
- 只对 `Player` 生效
- 直接修改 `foodLevel`
- 如果扣除后结果 `< min`，则设置为 `min`

---

## 语法示例

### 基础用法：扣除自己的饱食度

```plain
hunger-cost { amount = 6 }
```

扣除 6 点饱食度（3 格鸡腿），作用于施法者自身。

---

### 指定保底值

```plain
hunger-cost {
  amount = 10
  min = 4
}
```

扣除 10 点饱食度，但最终饱食度不会低于 4（2 格鸡腿）。

**示例：**
- 当前饱食度为 12 → 扣除后为 2 → 拉回到 4
- 当前饱食度为 20 → 扣除后为 10
- 当前饱食度为 5 → 扣除后为 -5 → 拉回到 4

---

### 扣除目标的饱食度

```plain
hunger-cost {
  amount = 8
  target = "target"
  min = 0
}
```

对技能目标（`ctx.target`）扣除 8 点饱食度。

---

### 使用饱食度代价的技能示例

```plain
# 施放火球需要消耗饥饿值
seq {
  hunger-cost { amount = 3 target = "self" }
  message { text = "&e消耗了 1.5 格饱食度！" }
  particle {
    type = "FLAME"
    center = "@self"
    spread = 0.5
    count = 30
  }
  # ... 其他技能效果
}
```

---

### 搭配 `if` 判断饱食度是否充足

```plain
# 先判断饱食度是否足够
if {
  cond = "papi.player_food_level >= 6"
} {
  # 消耗 3 格饱食度施放技能
  seq {
    hunger-cost { amount = 6 target = "self" }
    message { text = "&a施展了强力技能！" }
    damage { amount = 8 target = "target" }
  }
}
else {
  message { text = "&c饥饿度不足，无法施放此技能！" }
}
```

---

### 群体技能：对范围内的玩家扣除饱食度

```plain
# 先选取范围内的玩家
target-entity {
  center = "@self"
  radius = 5
  type = "player"
  includeSelf = false
  storeList = "affected_players"
}

# 对选中的每个玩家扣除饱食度（需要在支持集合变量的版本中实现）
# 注：当前版本仅支持 target = "self" / "target"，集合支持需后续扩展
hunger-cost { amount = 4 target = "target" }
```

---

### 连击技能：分阶段扣除

```plain
seq {
  # 第一阶段：消耗 2 格饱食度
  hunger-cost { amount = 4 target = "self" }
  damage { amount = 5 target = "target" }
  
  delay { ticks = 10 } {
    # 第二阶段：再消耗 1 格饱食度
    hunger-cost { amount = 2 target = "self" }
    damage { amount = 7 target = "target" }
  }
  
  delay { ticks = 20 } {
    # 第三阶段：再消耗 1.5 格饱食度
    hunger-cost { amount = 3 target = "self" }
    damage { amount = 10 target = "target" }
  }
}
```

---

### 随机效果：根据饱食度改变技能效果

```plain
var { food = papi.player_food_level }

random {
  weights = "1,2,1"
} {
  # 饱食度为 0-6（饥饿）→ 效果较弱
  if {
    cond = "vars.food <= 6"
  } {
    hunger-cost { amount = 2 target = "self" }
    damage { amount = 3 target = "target" }
    message { text = "&7你处于饥饿状态，技能威力减弱！" }
  }
  
  # 饱食度为 7-12（正常）→ 标准效果
  if {
    cond = "vars.food >= 7 && vars.food <= 12"
  } {
    hunger-cost { amount = 4 target = "self" }
    damage { amount = 6 target = "target" }
    message { text = "&a标准威力！" }
  }
  
  # 饱食度为 13-20（饱食）→ 强化效果
  if {
    cond = "vars.food >= 13"
  } {
    hunger-cost { amount = 6 target = "self" }
    damage { amount = 9 target = "target" }
    message { text = "&6饱食状态加成！技能威力提升！" }
  }
}
```

---

### 与 `effect` 配合：饥饿状态下获得负面效果

```plain
seq {
  hunger-cost { 
    amount = 8
    target = "self"
    min = 0
  }
  
  # 如果饱食度降到极低，施加负面效果
  if {
    cond = "papi.player_food_level <= 4"
  } {
    effect {
      type = "HUNGER"
      amp = 0
      time = 100
      target = "self"
    }
    message { text = "&c饥饿正在侵蚀你的身体！" }
  }
}
```

---

## 💡 设计说明

### 饱食度数值对照

| 数值 | 视觉表现 | 说明 |
| --- | --- | --- |
| 1 | 0.5 格鸡腿 | 半格 |
| 2 | 1 格鸡腿 | 完整一格 |
| 4 | 2 格鸡腿 | 两格 |
| 8 | 4 格鸡腿 | 一半饱食度 |
| 20 | 10 格鸡腿 | 满饱食度 |

### 使用场景

- **资源消耗**：技能需要消耗饥饿值作为代价
- **平衡机制**：通过饥饿度限制强力技能的频繁使用
- **战略决策**：玩家需要管理饥饿度来使用技能
- **状态联动**：低饥饿度触发特定效果

---

## ⚠️ 注意事项

!> **仅对玩家生效**：如果目标不是 `Player`，语句会被跳过

!> **保底值限制**：`min` 参数确保玩家不会因为使用技能而饿死

!> **直接修改值**：不触发 MC 原版饥饿机制的事件，需配合 `effect { type = "HUNGER" }` 来模拟饥饿效果

---

## 🔗 相关语句

- [🗡️ damage - 造成伤害](️damage—造成伤害.md) - 配合饥饿代价造成伤害
- [🧪 effect - 药水效果](effect—施加药水效果.md) - 配合饥饿施加负面状态
- [⚖️ if - 条件判断](⚖️if—条件判断语句.md) - 根据饱食度判断技能效果
- [🗨️ message - 发送消息](️message—发送消息语句.md) - 提示玩家饱食度消耗

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-01-XX XX:XX:XX</p>
</div>
