# 📢 nearby-message — 向附近玩家发送消息

> [🏠 首页](/) | [📚 语句总览](index.md) | **消息与界面** | [← 🗨️ message](️message—发送消息语句.md) | [🪶 title →](title—显示标题与副标题.md)

---

## 📖 简介

`nearby-message` 用于**向指定半径范围内的玩家发送一条消息**。  
它非常适合在技能释放时提示周围的玩家，例如：

**常见用途：**
- 通知附近玩家有技能爆发
- 实现 AOE 技能的"喊话"提示
- 在特定范围广播事件（如陷阱触发、boss 吼叫、区域提示）

**与 `message` 的区别：**  
👉 `message` 只会发送给执行者（或目标）  
👉 而 `nearby-message` 会广播给**执行者周围的所有玩家**（包括自己，除非你手动排除）

---

## ⚙️ 语法

```yaml
nearby-message { radius = R; text = "消息内容" }
```

---

## 🔢 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `radius` | `Double` | `8.0` | 消息广播半径（X/Y/Z 三个方向上各 ±radius） |
| `text` | `String` | `""` | 发送的消息文本，支持 `&` 颜色符号与变量插值 |

---

## ⚡ 示例

### ① 基本使用

```yaml
nearby-message { 
    radius = 12
    text = "&e%player_name% 释放了 &c火焰爆发！"
}
```

**效果**：在玩家周围 12 格内的所有玩家都会看到一条黄色广播：

```
玩家YuaZer 释放了 火焰爆发！
```

### ② 与技能特效结合

```yaml
seq {
    particle { type = "EXPLOSION"; at = "@self"; count = 10 }
    nearby-message { radius = 10; text = "&c%player_name% 爆发出强烈的能量！" }
    damage { amount = 10; targetVar = "victims" }
}
```

玩家释放技能时，先播放爆炸粒子，然后让周围 10 格的玩家看到一条提示。

### ③ 延迟广播（与 `delay` 搭配）

```yaml
seq {
    message { text = "&7你引导了一股能量..." }
    delay { ticks = 40 } {
        nearby-message { radius = 8; text = "&c一股能量从 %player_name% 身上爆发！" }
    }
}
```

**效果**：玩家先发送"引导"提示，2 秒后（40 ticks）周围玩家收到爆发提示。

### ④ 动态半径（变量插值）

```yaml
var { r = 20 }
nearby-message { radius = ${r}; text = "&b半径 ${r} 格的范围广播！" }
```

则广播范围会根据 `var` 定义的半径变量动态变化。

### ⑤ Boss 出现提示

```yaml
seq {
    nearby-message { 
        radius = 30
        text = "&4&l【警告】&r &c强大的 Boss 已经苏醒！"
    }
    sound { sound = "ENTITY_ENDER_DRAGON_GROWL"; v = 2.0; p = 0.8 }
    particle { type = "EXPLOSION_LARGE"; count = 5; at = "@self" }
}
```

---

## 💡 使用建议

?> **适合配合大型技能使用，增强游戏临场感**

?> **可以使用 PlaceholderAPI 实现动态消息**

?> **配合 `ifchain` 可以根据条件发送不同消息**

---

## ⚠️ 注意事项

!> **默认包含施法者自己，如需排除需要额外判断**

!> **范围过大可能影响体验，建议根据技能类型合理设置半径**

!> **高频触发可能导致消息刷屏，建议配合延迟使用**

---

## 🔗 相关语句

- [🗨️ message - 发送消息](️message—发送消息语句.md) - 向指定目标发送消息
- [🪶 title - 标题显示](title—显示标题与副标题.md) - 屏幕中央显示标题
- [🔊 sound - 音效播放](sound—播放音效.md) - 配合音效增强效果

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 11:06:56</p>
</div>
