# 🎲 random — 按权重随机执行一个子块

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [← 🔁 for](for—计次数_区间循环_临时变量.md) | [⏳ delay →](⏳delay—延迟执行后续语句.md)

---

## 📖 简介

`random` 会在其子块列表中**按权重**随机选择**一个**执行。

**核心特性：**
- 权重通过参数 `weights = "w1,w2,w3,..."` 指定
- 如果未指定或数量不足，对应子块权重默认 **1.0**
- 每次只执行**一个**子块；不执行并行或多个分支

**内部实现要点：**
- 先计算权重总和 `sum`，生成 `r ∈ [0, sum)`
- 依次减去每个子块权重，首次 `r <= weight` 的子块被执行
- 为防浮点误差，若循环仍未命中，则回退执行**最后一个**子块

---

## ⚙️ 语法

```yaml
random { weights = "w1,w2,w3" } {
    # 这里放多个"子块候选项"
    # 每个候选项可以是单条语句，或一个 seq { ... } 组合块

    # 候选 1
    seq {
        message { text = "&a发动：绿焰形态" }
        particle { type = "GREEN_FLAME"; count = 30; at = "@self" }
    }

    # 候选 2
    seq {
        message { text = "&b发动：雷霆形态" }
        particle { type = "ELECTRIC_SPARK"; count = 60; at = "@self" }
    }

    # 候选 3
    seq {
        message { text = "&c发动：爆裂形态" }
        particle { type = "EXPLOSION"; count = 1; at = "@self" }
        damage { amount = 6.0; target = "target" }
    }
}
```

?> **提示**：若某个候选项需要多条语句，请用 `seq { ... }` 包起来。

---

## 🔢 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `weights` | `String` | _(可选)_ | 逗号分隔的权重串，如 `"3,7,1"`。缺失或数量不足时，缺口权重按 **1.0** 处理；多出的权重会被忽略。 |

**权重规则：**
- **允许小数**（Double），如 `0.2, 0.3, 0.5`
- **全 0 权重不推荐**：实现上仍能选中第一个（因为 `sum=0` 时 `r=0`，首个 `r <= weight(=0)` 命中）
- **避免负数权重**：行为未定义

---

## ⚡ 快速示例

### ① 等概率（不写权重）

```yaml
random {
    message { text = "&7等概率：A" }
    message { text = "&7等概率：B" }
    message { text = "&7等概率：C" }
}
```

三个分支的权重都默认为 1.0 → **1/3 概率**各自执行。

### ② 非等概率（写权重）

```yaml
random { weights = "3,7" } {
    message { text = "&a30%：治疗 +2" }
    message { text = "&e70%：治疗 +5" }
}
```

- 第一条权重 3，第二条权重 7 → 概率约 30%/70%

### ③ 权重列数不足时的默认值

```yaml
random { weights = "5" } {
    message { text = "&a第一分支（权重=5）" }
    message { text = "&b第二分支（权重=1，自动补齐）" }
}
```

### ④ 复杂候选（使用 `seq` 把多条语句打包）

```yaml
random { weights = "2,1" } {
    seq {
        message { text = "&a轻度冰封" }
        effect  { type = "SLOW"; time = 40; amp = 0; target = "target" }
    }
    seq {
        message { text = "&b强力冰封" }
        effect  { type = "SLOW"; time = 80; amp = 1; target = "target" }
        damage  { amount = 4.0; target = "target" }
    }
}
```

### ⑤ 技能形态随机切换

```yaml
random { weights = "5,3,2" } {
    seq {
        message { text = "&e火焰形态（50%）" }
        particle { type = "FLAME"; count = 50; at = "@self" }
        damage { amount = 10; targetVar = "enemies" }
    }
    seq {
        message { text = "&b冰霜形态（30%）" }
        particle { type = "SNOWFLAKE"; count = 40; at = "@self" }
        effect { type = "SLOW"; amp = 2; time = 60; targetVar = "enemies" }
    }
    seq {
        message { text = "&d雷电形态（20%）" }
        particle { type = "ELECTRIC_SPARK"; count = 30; at = "@self" }
        damage { amount = 15; targetVar = "enemies" }
        effect { type = "GLOWING"; amp = 0; time = 40; targetVar = "enemies" }
    }
}
```

---

## 💡 使用建议

?> **用于创建随机性技能效果，增加游戏趣味性**

?> **配合权重可以精确控制各个分支的触发概率**

?> **适合实现"暴击系统"、"随机buff"、"多形态技能"等玩法**

---

## ⚠️ 注意事项

!> **每次只会执行一个分支，不会并行执行多个**

!> **权重总和决定概率分布，单个权重除以总和即为该分支概率**

---

## 🔗 相关语句

- [⚖️ if - 条件判断](⚖️if—条件判断语句.md) - 基于条件的确定性分支
- [🧠 ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md) - 复杂条件逻辑
- [🔁 for - 循环语句](for—计次数_区间循环_临时变量.md) - 重复执行

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 10:52:12</p>
</div>
