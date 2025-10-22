# 🪶 title — 显示标题与副标题

> [🏠 首页](/) | [📚 语句总览](index.md) | **消息与界面** | [← 📢 nearby-message](nearby-message—向附近玩家发送消息.md) | [🌌 particle →](particle—粒子特效语句.md)

---

## 📖 简介

`title` 语句用于**在玩家屏幕中央显示标题与副标题文字**，  
这是 Minecraft 常见的屏幕提示方式之一，常用于：

**典型应用场景：**
- 技能释放的提示动画
- 战斗状态通知（如「暴击！」「无敌状态」）
- 任务开始、Boss 出现等场景
- 特效过场或剧情演出

---

## ⚙️ 语法

```yaml
title { 
    title = "主标题"
    subtitle = "副标题"
    in = 10
    stay = 40
    out = 10
}
```

这会在执行者的屏幕中央显示一条渐入的标题提示。

**时间参数说明：**
- `in`：淡入时间（tick）
- `stay`：停留时间（tick）
- `out`：淡出时间（tick）

默认值：`in = 10`, `stay = 40`, `out = 10`

---

## 🔢 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `title` | `String` | `null` | 主标题内容（可为空） |
| `subtitle` / `subTitle` | `String` | `null` | 副标题内容，支持别名键 |
| `in` | `Int` | `10` | 标题淡入时间（单位 tick） |
| `stay` | `Int` | `40` | 标题停留时间（单位 tick） |
| `out` | `Int` | `10` | 标题淡出时间（单位 tick） |

?> **别名兼容**：你可以写 `subtitle` 或 `subTitle`，两者效果相同

---

## ⚡ 示例

### ① 基本示例

```yaml
title {
    title = "&6神技释放！"
    subtitle = "&7天地为之动容"
    in = 10
    stay = 40
    out = 10
}
```

**显示：**
- 主标题 "神技释放！"（金色）
- 副标题 "天地为之动容"（灰色）
- 渐入 0.5 秒 → 停留 2 秒 → 渐出 0.5 秒

### ② 技能提示组合

```yaml
seq {
    message { text = "&a你引导了太阳之力！" }
    delay { ticks = 40 } {
        title {
            title = "&6日耀裁决"
            subtitle = "&7炽焰掠过的轨迹"
            in = 5
            stay = 30
            out = 5
        }
    }
}
```

**效果**：玩家先收到一条提示信息，2 秒后在屏幕中央显示"日耀裁决"标题。

### ③ 无副标题

```yaml
title {
    title = "&b阶段完成"
    in = 10
    stay = 60
    out = 10
}
```

只显示主标题，副标题为空。

### ④ 动态变量与 PAPI 支持

```yaml
title {
    title = "&a%player_name%"
    subtitle = "&e当前血量: %player_health%"
}
```

通过 PlaceholderAPI 动态替换玩家信息。

### ⑤ 倒计时效果

```yaml
for { from = 3; to = 1; step = -1; var = "sec" } {
    title { 
        title = "&c&l${sec}"
        subtitle = "&7技能即将释放..."
        in = 0
        stay = 20
        out = 0
    }
    delay { ticks = 20 }
}
title { 
    title = "&a&lGO!"
    subtitle = "&e技能已释放"
    in = 0
    stay = 10
    out = 10
}
```

### ⑥ 暴击提示

```yaml
ifchain {
    case { cond = "var.isCrit == true" } {
        title {
            title = "&c&l暴击！"
            subtitle = "&e造成 ${damage} 点伤害"
            in = 5
            stay = 20
            out = 5
        }
    }
}
```

---

## 💡 使用建议

?> **配合 `delay` 可以创建连续的标题动画**

?> **使用短时间参数（in/out）可以实现快速闪现效果**

?> **结合 `for` 循环可以制作倒计时**

?> **支持 Minecraft 颜色代码和格式代码（&l 加粗、&o 斜体等）**

---

## ⚠️ 注意事项

!> **频繁显示标题可能影响玩家视野，建议合理使用**

!> **标题会覆盖之前的标题，注意不要冲突**

!> **时间单位为 tick（1秒=20tick），注意换算**

---

## 🔗 相关语句

- [🗨️ message - 发送消息](️message—发送消息语句.md) - 聊天栏消息
- [📢 nearby-message - 范围消息](nearby-message—向附近玩家发送消息.md) - 范围广播
- [⏳ delay - 延迟执行](⏳delay—延迟执行后续语句.md) - 制作标题动画
- [🔁 for - 循环语句](for—计次数_区间循环_临时变量.md) - 倒计时效果

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 11:15:01</p>
</div>
