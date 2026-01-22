# 🧠 ifchain — 多分支条件执行 / case … else 链

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [← ⚖️ if](⚖️if—条件判断语句.md) | [🔁 for →](for—计次数_区间循环_临时变量.md)

---

## 📖 功能简介

`ifchain` 用于按照**从上到下、命中即止**的方式执行条件分支：

- 支持多个 `case { cond = "..." } { … }`
- 可选 `else { … }` 兜底
- 条件表达式由 `JSConditionEvaluator` 以 **JavaScript 表达式**求值
- 内置可用变量：`var`（上下文变量表）、`player`、`x`、`y`、`z`

命中第一个 `case` 就会**立即执行并返回**，不会继续匹配后续 `case`。若全部未命中，则执行 `else`（存在时）。

---

## ⚙️ 语法格式

```yaml
ifchain {
  case { cond = "<JS表达式>" } {
    # 命中该条件时执行的子语句块
  }

  case { cond = "<JS表达式>" } {
    # 另一个候选
  }

  else {
    # 所有 case 都未命中时执行
  }
}
```

### 条件表达式可用的变量（由实现注入）

| 变量 | 说明 |
|------|------|
| `var` | `ctx.vars` 的只读快照（Map）<br/>例如：`var.solar_victims`、`var.pointer`、`var.hitPos` |
| `player` | 当前玩家名（若无玩家则为 `ctx.source.name`） |
| `x` / `y` / `z` | 触发源（`ctx.source`）的坐标 |

?> **注意**：条件中可使用 `var.` / `vars.` / `${}`（三种写法都支持）

---

## 🧮 条件表达式速查（Nashorn 风格）

| 表达式 | 说明 |
|--------|------|
| `var.solar_victims == null` | 判空 |
| `var.solar_victims != null && var.solar_victims.size() > 0` | 非空且有元素（Java 集合） |
| `var.damage >= 7.5` | 数值比较 |
| `var.stage === "CASTING"` | 字符串相等 |
| `y > 100` | 坐标判断 |
| `var.pointer != null` | 存在性判定 |
| `(var.hit != null && var.hit.size() > 2) \|\| x < 0` | 逻辑组合 |

?> **提示**：`ctx.vars` 里的 Java `Collection` 在 JS 里一般用 `size()` 获取数量；数组可用 `length`。

---

## ⚡ 快速示例

### ① 基础示例：检测目标列表

```yaml
ifchain {
  # 如果实体列表为空, 发送信息
  case { cond = "var.solar_victims == null" } {
    message { text = "&c你没命中任何目标!" }
  }

  # 否则：展示列表并结算
  else {
    message { text = "目前实体列表: ${solar_victims}" }
    damage  { amount = 9.5; targetVar = "solar_victims" }
    effect  { type = "GLOWING"; amp = 0; time = 60; targetVar = "solar_victims" }
    sound   { sound = "ENTITY_FIREWORK_ROCKET_BLAST"; v = 0.9; p = 1.6; at = "${solar_mark}" }
  }
}
```

### ② 非空并且数量大于 3，给予更高伤害

```yaml
ifchain {
  case { cond = "var.solar_victims != null && var.solar_victims.size() > 3" } {
    damage { amount = 12.0; targetVar = "solar_victims" }
    message { text = "&e群体打击！命中超过 3 个目标，额外伤害！" }
  }
  else {
    damage { amount = 7.0; targetVar = "solar_victims" }
  }
}
```

### ③ 射线命中判定：命中则击退，未命中则提示

```yaml
seq {
  raycast { range = 12 storeEntity = "pointer" }

  ifchain {
    case { cond = "var.pointer != null" } {
      knockback { targetVar = "pointer" s = 1.0 y = 0.25 }
      sound { key = "entity.player.attack.knockback" v = 1 p = 1 at = "@self" }
    }
    else {
      message { text = "&7未命中目标~" }
    }
  }
}
```

### ④ 按高度段分三档处理（从上到下命中即止）

```yaml
ifchain {
  case { cond = "y >= 150" } {
    message { text = "&b高空领域·增益强化" }
    effect  { type = "SPEED"; amp = 2; time = 80; target = "self" }
  }
  case { cond = "y >= 70" } {
    message { text = "&a高地优势·轻微增益" }
    effect  { type = "SPEED"; amp = 1; time = 60; target = "self" }
  }
  else {
    message { text = "&7低地作战·无增益" }
  }
}
```

### ⑤ 与 `velocity/knockback` 联动（命中分支后即返回）

```yaml
ifchain {
  case { cond = "var.hit != null && var.hit.size() > 0" } {
    velocity  { x = 0 y = 0.35 z = 1.1 mul = 1.1 }  # 自己突进
    knockback { targetVar = "hit" s = 1.0 y = 0.3 }  # 目标后退
  }
  else {
    command { cmd = "tell ${player} &7没有命中目标，已取消突进" }
  }
}
```

---

## 💡 使用建议

?> **多分支判断优先使用 `ifchain`，而不是嵌套多个 `if`**

?> **将最可能命中的条件放在前面，可以提高执行效率**

?> **配合 `raycast`、`target-entity` 等语句，可以实现复杂的技能判定逻辑**

---

## ✅ 小结

- `ifchain` 让**条件逻辑结构化**，把"判断 → 子动作块"写得一目了然
- 采用 **JS 表达式**，灵活判定 `var` 中的任意上下文变量
- 与 `raycast`、`target-entity`、`velocity/knockback`、`damage` 等语句组合，可覆盖 90% 技能判定需求

---

## 🔗 相关语句

- [⚖️ if - 简单条件判断](⚖️if—条件判断语句.md) - 简单的二分支判断
- [🧰 var - 变量定义](var—定义_覆盖上下文变量.md) - 定义条件中使用的变量
- [🎯 target-entity - 选取实体](target-entity—范围选取实体_写入上下文变量.md) - 获取实体列表用于判断

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-12-18 12:28:00</p>
</div>
