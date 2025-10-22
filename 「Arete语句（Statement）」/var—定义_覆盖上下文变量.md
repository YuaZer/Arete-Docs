# 🧰 var — 定义/覆盖上下文变量

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [⚖️ if →](⚖️if—条件判断语句.md)

---

## 📖 简介

`var` 用来向当前技能的执行上下文 `ctx.vars` 写入键值对。  
它支持**字面量自动类型识别**、**PlaceholderAPI 快捷写法**，并允许覆盖已有变量。

**核心特性：**
- **作用域**：仅本次技能执行（ExecutionContext）
- **写入位置**：`ctx.vars[key] = value`
- **约束**：变量名不能包含 `.`（表达式/插值通过 `vars.xxx` 访问）

---

## ⚙️ 写法与参数

### 基本写法

```yaml
var {
    hp = 30
    name = "哈基米"
    isBoss = true
}
```

### 占位符（PAPI）快捷写法

以 `papi.` 开头将被解析为 PlaceholderAPI，并写入两份值：

- `ctx.vars["level"] = <PAPI解析结果>`
- 额外：`ctx.vars["papi.player_level"] = <同样的结果>`

```yaml
var {
    level = papi.player_level
    money = papi.vault_eco_balance
}
```

### 类型自动识别（字面量）

| 输入 | 解析类型 |
|------|---------|
| `true/false` | `Boolean` |
| `123` | `Int` |
| `12.34` | `Double` |
| `"abc"` 或 `'abc'` | 去引号后的字符串 |
| 其他 | 原样字符串 |

```yaml
var {
    a = true
    b = 42
    c = 3.1415
    d = "hello"
    e = papi.player_name   # 解析为玩家名字符串
}
```

---

## 📝 变量名规则

!> **禁止含 `.`**：`var { "player.name" = "x" }` 会被跳过并打印警告

**访问语法统一为：**
- `vars.key` 或 `${key}` 在 ifchain 和 if 中，用 `vars.key`
- 在其他语句中，则用 `${key}`

```yaml
# 条件判断中
ifchain { case { cond = "vars.hp > 20" } ... }

# 消息中
message { text = "&a当前血量: ${hp}" }
```

---

## 🔗 与其他语句的协作

### 配合 `message`

```yaml
var { title = "圣光斩" }
message { text = "&e${player} 施放了 &6${title}&e！" }
```

### 配合 `ifchain`

```yaml
var { critChance = 35 }

ifchain {
    case { cond = "vars.critChance >= 50" } {
        message { text = "&c暴击几率过高，已被平衡！" }
    }
    else {
        message { text = "&a当前暴击率: ${critChance}%" }
    }
}
```

---

## 💡 常见技巧

### 想要字面"papi.xxx"而不是解析？

用**引号**包住：

```yaml
var {
    raw = "papi.player_name"   # 不解析，原样字符串
}
```

### 运行期覆盖

后续 `var` 可覆盖先前值：

```yaml
var { mode = "normal" }
# ……中间流程
var { mode = "burst" }  # 覆盖
```

---

## ⚠️ 注意事项

!> **键名含 `.` 会被跳过**：日志会输出  
`[Arete] Var key 'a.b' contains '.', skipped.`

!> **无玩家时 PAPI 解析为空**：会输出告警，并返回 `""`  

如果该键必须有值，请在 `ifchain` 里兜底：

```yaml
ifchain {
    case { cond = "vars.playerName == null || vars.playerName == \"\"" } {
        message { text = "&c未找到玩家上下文，跳过 PAPI。" }
    }
}
```

---

## 🔗 相关语句

- [⚖️ if - 条件判断](⚖️if—条件判断语句.md) - 使用变量进行条件判断
- [🧠 ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md) - 复杂条件逻辑
- [🗨️ message - 发送消息](️message—发送消息语句.md) - 在消息中使用变量

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 10:11:01</p>
</div>
