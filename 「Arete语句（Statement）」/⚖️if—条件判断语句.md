# ⚖️ if — 条件判断语句

> [🏠 首页](/) | [📚 语句总览](index.md) | **基础控制** | [← 🧰 var](var—定义_覆盖上下文变量.md) | [🧠 ifchain →](ifchain—多分支条件执行_case…else链.md)

---

## 📖 简介

`if` 语句用于根据布尔条件表达式决定执行哪一段技能逻辑。  
它支持 `ctx.vars` 中的变量访问（通过 `vars.xxx`），  
支持数字、字符串、布尔、比较、逻辑与或等 JavaScript 风格表达式。

---

## ⚙️ 语法

```yaml
if { cond = "<JavaScript表达式>" } {
    # 条件为真时执行
} else {
    # 条件为假时执行（可选）
}
```

**条件表达式支持：**
- 变量访问：`vars.变量名`
- 数值比较：`>`、`<`、`>=`、`<=`、`==`、`!=`
- 逻辑运算：`&&`（与）、`||`（或）、`!`（非）
- 字符串比较：`==`、`!=`
- 空值检查：`== null`、`!= null`

---

## 📝 示例用法

### 示例 1：基础条件判断

```yaml
var { hp = 30 }

if { cond = "vars.hp > 20" } {
    message { text = "&a生命值高于 20，安全！" }
} else {
    message { text = "&c生命值过低，危险！" }
}
```

### 示例 2：字符串判断

```yaml
var { mode = "burst" }

if { cond = "'vars.mode' === 'burst'" } {
    message { text = "&c进入爆发模式！" }
}
```

### 示例 3：检测变量是否存在

```yaml
if { cond = "vars.solar_victims == null" } {
    message { text = "&c你没有命中任何目标！" }
} else {
    message { text = "&a命中了 ${solar_victims.size} 个目标！" }
}
```

### 示例 4：组合条件

```yaml
var { hp = 15; level = 50 }

if { cond = "vars.hp < 20 && vars.level >= 30" } {
    message { text = "&e低血量但等级足够，触发特殊效果！" }
    effect { type = "REGENERATION"; amp = 1; time = 60 }
}
```

---

## 💡 使用建议

?> **简单条件用 `if`，复杂多分支用 [`ifchain`](ifchain—多分支条件执行_case…else链.md)**

?> **条件表达式中必须使用 `vars.变量名`，而在其他语句参数中使用 `${变量名}`**

---

## ⚠️ 注意事项

!> **字符串比较时需要使用单引号或双引号**  
正确：`vars.mode == 'burst'`  
错误：`vars.mode == burst`

!> **访问不存在的变量会导致错误，建议先检查 null**  
```yaml
if { cond = "vars.target != null && vars.target.hp > 0" } {
    # 安全的访问
}
```

---

## 🔗 相关语句

- [🧰 var - 变量定义](var—定义_覆盖上下文变量.md) - 定义在条件中使用的变量
- [🧠 ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md) - 更强大的多分支判断
- [🎲 random - 随机执行](random—按权重随机执行一个子块.md) - 基于概率的分支

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 10:15:03</p>
</div>
