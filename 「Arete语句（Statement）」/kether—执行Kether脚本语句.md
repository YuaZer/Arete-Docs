# 🧩 kether — 执行 Kether 脚本语句

> [🏠 首页](/) | [📚 语句总览](index.md) | **逻辑与条件** | [← 🔁 while](🔁while—循环执行语句.md) | [🗃️ var →](🗃️var—定义_覆盖上下文变量.md)

---

## 📖 简介

`kether` 语句用于在 Arete 技能系统中执行一段 **Kether 脚本**，  
支持多行脚本或脚本列表，可与 Arete 的上下文变量联动。

它允许你直接在 DSL 中编写逻辑判断、权限检测、变量运算等功能，  
适合复杂技能逻辑、条件触发、数据控制等高级场景。

---

## ⚙️ 参数说明

| 参数名      | 类型                     | 默认值    | 说明                                                         |
|----------|------------------------|--------|------------------------------------------------------------|
| `script` | String / List\<String> | _(必填)_ | 要执行的 Kether 脚本，可为多行字符串或脚本列表。                               |
| `store`  | String                 | _(可选)_ | 若填写，将脚本执行结果存入 `ctx.vars[store]`，可在后续语句中以 `${vars.xxx}` 使用。 |

---

## 🧠 参数格式示例

### ✅ 单脚本（可多行）

```yaml
kether {
  script = """
    check perm "arete.use"
    tell "你拥有 arete.use 权限！"
  """
}
```
✅ 多行脚本（列表形式）
```yaml
kether {
  script.0 = "set a to 5"
  script.1 = "set b to a * 2"
  script.2 = "tell 'a=${a}, b=${b}'"
}
```
✅ 带结果存储
```yaml
kether {
  script = "set result to %player_health%"
  store = "hp"
}
message { text = "&a你的血量为: ${hp}" }
```
🔧 插值系统（变量替换）
在 script 内容中，你可以使用 Arete 的 ${} 插值语法，
这些变量会在脚本执行前被替换为 ExecutionContext 中的实际值。

| 插值形式         | 含义                          | 示例                                |
|--------------|-----------------------------|-----------------------------------|
| `${player}`  | 当前执行者的玩家名                   | `"check perm ${player}.use"`      |
| `${target}`  | 技能目标的实体名                    | `"tell '攻击目标为 ${target}'"`        |
| `${varName}` | 从上下文变量中读取值                  | `"set damage to ${damage}"`       |
| 自动类型格式化      | `Location` 与 `Entity` 将友好显示 | `${aimPos}` → `123.0,64.0,-256.0` |

?> 提示：变量未命中时会保留原样，不会被清空。

🧩 特性说明
✅ 支持多行或列表脚本：可直接在 YAML 中写多行逻辑。

✅ 自动选择 Player 执行者：会自动取 ctx.player → ctx.source → ctx.target。

✅ 结果可存储：可将 Kether 执行结果放入上下文变量。

✅ 内置 PAPI 支持：不做外部 PlaceholderAPI 替换，Kether 自身可用 papi 语句。

⚡ 示例
① 简单执行权限检测
```yaml
复制代码
kether {
  script = """
    check perm "arete.use"
    tell "你有 arete.use 权限！"
  """
}
```
② 动态参数与上下文结合
```yaml
复制代码
var { skill = "fireball" }
kether {
  script = "tell '${player} 正在释放 ${skill}'"
}
```
③ 多行逻辑与存储结果
```yaml
复制代码
kether {
  script.0 = "set dmg to %player_health% / 2"
  script.1 = "set crit to random 1 to 100"
  script.2 = "if crit > 80 { tell '&c暴击！伤害翻倍！' }"
  store = "dmg"
}
message { text = "&e最终造成伤害: ${dmg}" }
```
④ 结合条件与控制流
```yaml
复
kether {
  script = """
    if %player_health% < 10 {
      tell "&c你的生命值太低了！"
    } else {
      tell "&a状态良好，继续战斗！"
    }
  """
}
```
## 💡 使用建议

### 结合 var 与 if 实现逻辑判断与技能条件控制
### 将结果存入变量再配合 message、title 等语句进行展示

利用 Kether 原生语法（如 check perm、tell、set、if 等）构建智能逻辑

## ⚠️ 注意事项
!> 不自动解析 PlaceholderAPI
Kether 自身支持 papi 语句，请直接在脚本中使用，如 papi "%player_name%"

!> 脚本执行错误不会中断主序列
出错仅会在控制台打印 [Arete] KetherEvalStatement 执行失败 提示。

!> store 存储结果为 Any 类型
推荐在后续语句中转为字符串 ${vars.xxx} 使用。

🔗 相关语句
🗃️ var - 变量定义

🧩 if - 条件判断

🗨️ message - 消息提示

⏳ delay - 延迟执行

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;"> <p>📝 更新时间: 2025-11-04 17:28:12</p> </div> ```