# 🔁 for — 计次数 / 区间循环 / 临时变量

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [← 🧠 ifchain](ifchain—多分支条件执行_case…else链.md) | [🎲 random →](random—按权重随机执行一个子块.md)

---

## 📖 功能简介

`for` 用来**重复执行子语句块**：既支持"执行 N 次"的简单循环，也支持 `[from…to]`（含端点）的**数值区间循环**，并可把当前循环计数/数值保存到临时变量里给子语句使用。

---

## ⚙️ 语法

### 方式一：按次数（`times`/`count`）

```yaml
for { times = 5; var = "i" } {
  # 循环体；i 依次为 0,1,2,3,4
}
```

等价写法：

```yaml
for { count = 5; as = "i" } { ... }
```

### 方式二：按区间（`from`/`to`/`step`）

```yaml
for { from = 0; to = 10; step = 2; var = "i" } {
  # i = 0,2,4,6,8,10
}
```

**特性说明：**
- **递增**：`from <= to` 时默认 `step = 1`
- **递减**：`from > to` 时默认 `step = -1`
- **含端点**：到达 `to` 会执行（浮点容差 1e-9）

!> **安全规则**：`step = 0` 会被忽略（不循环）；若 `step` 符号与方向不匹配则直接不执行。

---

## 🔢 参数说明

| 键 | 类型 | 说明 |
| --- | --- | --- |
| `times` / `count` | Int | 执行次数；当提供它们时，会忽略 `from/to/step`，内部按 `0..(n-1)` 迭代 |
| `from` | Double | 起始值（默认 `0`） |
| `to` | Double | 结束值（默认等于 `from`，即只执行一次） |
| `step` | Double | 步长（默认按方向取 `±1`；为 `0` 时会被纠正为 `±1`） |
| `var` / `as` | String | 将当前计数/数值写入 `ctx.vars[var]`，供子语句引用（如 `${i}`） |

### 变量行为

- 每次循环前写入 `ctx.vars[var]`；数值若接近整数会自动转为 **Int/Long**（否则保留 Double）
- 循环结束后会**还原/删除**这个变量：
  - 如果之前存在，会还原成旧值
  - 如果之前不存在，会从 `ctx.vars` 移除

---

## ⚡ 实用示例

### ① 执行 5 次，展示计数

```yaml
for { times = 5; var = "i" } {
  message { text = "&7第 ${i} 次释放粒子" }
  particle { type = "crit" at = "@self" }
}
```

### ② 递增位移（连贯冲刺感）

```yaml
for { from = 0; to = 0.8; step = 0.2; var = "t" } {
  # 每次轻微叠加前进速度
  velocity { x = 0 y = 0 z = 0.2 add = true }
  delay 1
}
```

### ③ 环形击退（用计数驱动多次 knockback）

```yaml
target-entity { center = "@self" radius = 4 type = "living" includeSelf = false storeList = "victims" }

for { times = 3; var = "round" } {
  knockback { targetVar = "victims" s = 0.6 y = 0.22 }
  sound { key = "entity.player.attack.knockback" v = 1 p = 1 at = "@self" }
  delay 2
}
```

### ④ 递减循环（倒计时）

```yaml
for { from = 3; to = 1; step = -1; var = "sec" } {
  title { text = "&e${sec}" }
  delay 20
}
message { text = "&aGo!" }
```

### ⑤ 结合 `ifchain` 做分段特效

```yaml
for { from = 1; to = 5; var = "i" } {
  ifchain {
    case { cond = "var.i <= 2" } { particle { type = "flame" at = "@self" } }
    case { cond = "var.i <= 4" } { particle { type = "spell" at = "@self" } }
    else { particle { type = "explosion" at = "@self" } }
  }
  delay 2
}
```

---

## 💡 使用建议

?> **配合 `delay` 可以创建连续的动画效果**

?> **循环变量自动清理，不会污染全局上下文**

?> **支持浮点数步长，可以实现平滑的数值变化**

---

## ⚠️ 注意事项

!> **避免过大的循环次数，可能影响服务器性能**  
建议单次循环不超过 100 次，配合 `delay` 分帧执行

!> **循环变量在循环结束后会被移除或还原**  
如需保留结果，请在循环内赋值给其他变量

---

## 🔗 相关语句

- [⏳ delay - 延迟执行](⏳delay—延迟执行后续语句.md) - 配合实现分帧动画
- [🧠 ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md) - 在循环中使用条件判断
- [🌌 particle - 粒子特效](particle—粒子特效语句.md) - 创建循环粒子效果

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 14:49:31</p>
</div>
