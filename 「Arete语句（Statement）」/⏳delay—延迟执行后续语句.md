# ⏳ delay — 延迟执行后续语句

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [← 🎲 random](random—按权重随机执行一个子块.md) | [🗨️ message →](️message—发送消息语句.md)

---

## 📖 简介

`delay` 语句用于**在指定的时间延迟后执行一段子语句块**。  

**常见用途：**
- 给技能添加前摇/后摇
- 为连段技能设置时间间隔
- 让多个效果分步触发（如连环爆炸、逐帧动画）

延迟单位为 **Minecraft Tick**（1 tick = 1/20 秒），  
即 `20 ticks = 1 秒`。

---

## ⚙️ 语法

```yaml
delay { ticks = N } {
    # 延迟 N ticks 后执行的语句块
    ...
}
```

**参数说明：**
- 参数 `ticks` 表示等待的时长（整数），默认 `20`
- `{ ... }` 内是延迟结束后要执行的内容
- 如果省略 `{ ... }`，语句只会简单地暂停指定时间，不执行任何后续动作

---

## 🔢 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `ticks` | Long (整数) | `20` | 延迟的时间长度，单位为 Tick（1 秒 = 20 Tick） |

---

## ⚡ 示例

### ① 基本用法 — 延迟执行

```yaml
delay { ticks = 40 } {
    message { text = "&a2 秒后发动技能！" }
}
```

说明：技能触发后等待 40 ticks（=2 秒），再向施法者发送提示。

### ② 连段技能示例

```yaml
seq {
    message { text = "&a第一击！" }
    damage { amount = 5.0; target = "target" }

    delay { ticks = 10 } {
        message { text = "&e第二击！" }
        damage { amount = 8.0; target = "target" }
    }

    delay { ticks = 20 } {
        message { text = "&c终结一击！" }
        damage { amount = 12.0; target = "target" }
        particle { type = "EXPLOSION"; at = "@target"; count = 10 }
    }
}
```

**效果时间轴：**
- 第 0 tick：执行第一击
- 第 10 tick：执行第二击
- 第 20 tick：执行终结一击

三段动作分时触发，表现为**连击动画**。

### ③ 无子块延迟（纯暂停）

```yaml
message { text = "&7准备中..." }
delay { ticks = 40 }
message { text = "&a准备完成！" }
```

效果：中间停顿 2 秒，然后输出第二条消息。

### ④ 与 `parallel` 组合：错时播放特效

```yaml
parallel {
    delay { ticks = 0  } { particle { type = "FIREWORK"; at = "@self" } }
    delay { ticks = 10 } { particle { type = "CLOUD"; at = "@self" } }
    delay { ticks = 20 } { particle { type = "CRIT"; at = "@self" } }
}
```

形成一种流畅的"时间特效序列"。

### ⑤ 倒计时效果

```yaml
for { from = 3; to = 1; step = -1; var = "sec" } {
    title { text = "&e${sec}" }
    delay { ticks = 20 }
}
title { text = "&aGO!" }
particle { type = "EXPLOSION"; count = 20; at = "@self" }
```

---

## 💡 使用建议

?> **与 `parallel` 组合**：实现不同特效的错时播放

?> **与 `seq` 组合**：制作连续动作，例如"蓄力 → 释放 → 冷却"

?> **与 `for` 组合**：创建重复的分帧动画效果

?> **嵌套使用**：`delay` 可以嵌套在 `if`、`random` 等语句内，无执行顺序冲突

---

## ⚠️ 常见问题（FAQ）

**Q1：`delay` 会阻塞主线程吗？**  
不会。内部使用协程与 Tick 计时逻辑，不影响服务器性能。

**Q2：能否设置小于 1 tick 的延迟？**  
不建议。Tick 是 Minecraft 的最小时间单位，`ticks < 1` 会被当作 0，即立即执行。

**Q3：能否同时执行多个 `delay`？**  
可以，彼此独立；若在 `parallel` 语句中使用，会同时开始计时。

---

## 🔗 相关语句

- [🔁 for - 循环语句](for—计次数_区间循环_临时变量.md) - 配合实现重复延时效果
- [🌌 particle - 粒子特效](particle—粒子特效语句.md) - 创建分帧粒子动画
- [🪶 title - 标题显示](title—显示标题与副标题.md) - 制作倒计时效果

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 10:55:03</p>
</div>
