# 🗨️ message — 发送消息语句

> [🏠 首页](/) | [📚 语句总览](index.md) | **消息与界面** | [← ⏳ delay](⏳delay—延迟执行后续语句.md) | [📢 nearby-message →](nearby-message—向附近玩家发送消息.md)

---

## 📖 简介

`message` 语句用于向玩家或全服发送一条带有变量插值与 PlaceholderAPI 支持的消息。  
它可嵌入任意技能序列（`seq` / `parallel`）中，用于技能提示、系统广播、战斗反馈等。

---

## ⚙️ 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `text` | String | _(必填)_ | 要发送的消息文本，支持变量插值 `${varName}`、`${player}`、`${target}` 等 |
| `to` | String | `"self"` | 消息接收者。支持：<br/>• `"self"`：施法者<br/>• `"target"`：技能目标<br/>• `"server"` 或 `"broadcast"`：全服广播 |

---

## 🔧 插值系统（变量替换）

在 `text` 中可使用以下占位符形式，所有占位符都由 `ExecutionContext.vars` 提供：

| 插值形式 | 含义 | 示例 |
| --- | --- | --- |
| `${player}` | 当前施法者玩家名 | `你释放了技能！` → `YuaZer 释放了技能！` |
| `${target}` | 当前技能目标实体名 | `攻击了 ${target}` |
| `${varName}` | 从上下文变量中读取值 | 例如 `${aimPos}` 或 `${damage}` |
| `${vars.xxx}` | 等价于 `${xxx}`，两种写法均可 | `${vars.pointer}` |
| **自动类型格式化** | 位置（`Location`）与实体（`Entity`）会被友好显示 | `${aimPos}` → `123.0,64.0,-256.0` |

?> **提示**：未命中变量时，会保留原样（不会替换为空字符串）

---

## 🧰 PlaceholderAPI 支持

如果执行者是 `Player`，则消息内容在替换变量后会自动通过 `PlaceholderAPI` 进行解析。  
这意味着你可以在消息中写：

```yaml
message { text = "&a你当前血量: %player_health%" }
```

---

## ⚡ 示例

### ① 基本消息

```yaml
message { text = "&a技能释放成功！" }
```

### ② 使用变量插值

```yaml
var { skillName = "火焰风暴" }
message { text = "&e${player} 释放了 &6${skillName}&e！" }
```

### ③ 发送给目标

```yaml
message { 
    text = "&c你受到了来自 ${player} 的攻击！" 
    to = "target" 
}
```

### ④ 全服广播

```yaml
message { 
    text = "&b&l【世界公告】&r&f ${player} 完成了史诗任务！" 
    to = "broadcast" 
}
```

### ⑤ 结合 PlaceholderAPI

```yaml
message { 
    text = "&a玩家: %player_name% | 等级: %player_level% | 血量: %player_health%" 
}
```

### ⑥ 多变量组合

```yaml
target-entity { center = "@self" radius = 5 type = "living" store = "enemy" storeCount = "cnt" }
message { text = "&e发现 &c${cnt} &e个目标，锁定: &b${enemyStr}" }
```

---

## 💡 使用建议

?> **配合 `var` 语句定义变量，使消息更加动态**

?> **使用颜色代码让消息更醒目：** `&a`绿色、`&c`红色、`&e`黄色、`&b`青色等

?> **结合条件判断实现智能提示**

---

## ⚠️ 注意事项

!> **变量名必须在 `ctx.vars` 中存在，否则会保留原始占位符**

!> **全服广播慎用，避免刷屏影响玩家体验**

---

## 🔗 相关语句

- [📢 nearby-message - 范围消息](nearby-message—向附近玩家发送消息.md) - 向周围玩家发送消息
- [🪶 title - 标题显示](title—显示标题与副标题.md) - 屏幕中央显示标题
- [🧰 var - 变量定义](var—定义_覆盖上下文变量.md) - 定义消息中使用的变量

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 09:57:46</p>
</div>
