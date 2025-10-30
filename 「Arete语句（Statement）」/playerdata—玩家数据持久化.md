# 🗂️ playerdata — 玩家数据持久化

在服务器运行期间为玩家维护一份可写入的持久化数据（关闭前保留，由 `PlayerDataStore` 托管）。支持设置/读取/删除/判断存在性，以及数值自增自减、字符串列表追加，并可把结果写入上下文变量供后续语句使用。

---

## 参数一览

| 参数名      | 类型     | 必填 | 默认值     | 说明                                                         |
|----------|--------|----|---------|------------------------------------------------------------|
| `op`     | String | 否  | `get`   | 操作类型：`set` / `get` / `remove                               |del|delete` / `has|exists` / `inc` / `dec` / `add` |
| `player` | String | 否  | `@self` | 目标玩家解析：`@self` / `@target` / 也可直接写玩家名                      |
| `key`    | String | 否  | —       | 数据键名。除极少数场景外应提供；为空将跳过并记录告警                                 |
| `value`  | String | 否  | —       | 写入/运算值：`set`/`inc`/`dec`/`add` 需要；`get`/`has`/`remove` 不需要 |
| `to`     | String | 否  | —       | 将结果写入变量名：`get` 写入取到的值；`has` 写入布尔；`inc/dec` 写入新数值           |

> `value` 会自动类型推断：优先尝试 `Int`/`Double`，其后 `true/false` 布尔，最后按原字符串存储。

---

## 快速示例

```plain
# 设置：记下玩家首次进入时间戳
playerdata { op = "set" key = "firstJoinTs" value = "${nowMillis}" }

# 读取：放入变量并提示
playerdata { op = "get" key = "firstJoinTs" to = "ts" }
message { text = "&7你的首次进入时间戳: ${ts}" }
```

---

## 用法详解

### 1) set — 写入/覆盖值
```plain
playerdata { op = "set" key = "title" value = "Brave" }
```

### 2) get — 读取值并可写入变量
```plain
playerdata {
  op = "get"
  key = "title"
  to = "pTitle"
}
message { text = "&7称号: ${pTitle}" }

说明：
读不到这个键 → 得到的是 null
写了 to → ctx.vars[to] = 读到的值（可以是 null）
没写 to → 只读，不落到上下文
```

### 3) remove/del/delete — 删除键
```plain
playerdata { op = "remove" key = "title" }
```

### 4) has/exists — 判断是否存在并写入布尔
```plain
playerdata {
  op = "has"
  key = "quest.daily.2025-10-30"
  to = "hasDaily"
}

ifchain {
  case { cond = "var.hasDaily == true" } {
    message { text = "&a你今天做过日常啦！" }
  }
  else {
    message { text = "&c今天还没做日常，去做一下~" }
  }
}

```

### 5) inc/dec — 数值自增/自减（默认步长 1）
```plain
# 击杀计数 +1
playerdata { op = "inc" key = "kills" }

# 经验 -5（写入新值到变量）
playerdata { op = "dec" key = "exp" value = "5" to = "newExp" }
message { text = "&7当前经验: ${newExp}" }
```

### 6) add — 追加到字符串列表
```plain
# 往背包记录清单里追加条目；原值若非列表会被转为单元素后再追加
playerdata { op = "add" key = "bagItems" value = "iron_ingot" }
```

### 7) 指定目标玩家
```plain
playerdata { op = "set" player = "@target" key = "assist" value = "1" }
```
---

## 一个完整的枪械示例
```plain
    # 0) 先判断这个玩家是否已经有这把枪的弹药数据
    playerdata {
      op = "has"
      key = "gun.ak47.ammo"
      to = "hasAmmo"
    }
    # 1) 没有 -> 初始化一次（只在第一次进来时成立）
    if { cond = "var.hasAmmo == false" } {
      playerdata { op = "set" key = "gun.ak47.ammo" value = "6" }
      playerdata { op = "set" key = "gun.ak47.ammoMax" value = "6" }
    }
    # 2) 不管刚才是新建的还是原来就有的，再读真正的数值到本次技能的 ctx.vars
    playerdata {
      op = "get"
      key = "gun.ak47.ammo"
      to = "ammo"
    }
    playerdata {
      op = "get"
      key = "gun.ak47.ammoMax"
      to = "ammoMax"
    }
    # 3) 开始真正的开枪逻辑
    ifchain {
      # 有子弹
      case { cond = "var.ammo > 0" } {
        # 子弹-1（内存）
        var { ammo = "${ammo} - 1" }
        # 回写到玩家持久化（这里会把上面算出来的数字真正存进去）
        playerdata { op = "set" key = "gun.ak47.ammo" value = "${ammo}" }
        # 发射一发子弹
        projectile {
          type = "snowball"
          shooter = "self"
          speed = 3.0
          spread = 0.03
          damage = 5.0
        }
        message { text = "&a砰！剩余弹药: &e${ammo}/${ammoMax}" }
      }
      # 没子弹 -> 换弹
      else {
        message { text = "&c当前子弹 ${ammo}/${ammoMax}" }
        message { text = "&c弹夹为空，开始换弹..." }
        # 给这把技能上 40 tick 冷却，防止狂点
        cooldown { op = "set" skill = "枪械/ak47" ticks = 40 }
        # 40tick 后真正回满
        delay { ticks = 40 } {
          # 再读一遍最大弹药（防止中途你改配置）
          playerdata { op = "get" key = "gun.ak47.ammoMax" to = "ammoMax" }
          # 持久化回满：这里必须要能解析 ${ammoMax}
          playerdata { op = "set" key = "gun.ak47.ammo" value = "${ammoMax}" }
          # 内存也回满
          var { ammo = "${ammoMax}" }
          message { text = "&a换弹完成！弹药已回满: &e${ammo}/${ammoMax}" }
        }
      }
    }
```
---
## ⚠️ 注意事项

- 未能解析到玩家（既非 `@self`/`@target` 玩家，也非有效名字）将跳过执行并输出告警。
- `key` 为空会跳过并输出告警；请确保键名有效。
- `get` 找不到键时会得到 `null`；若写入变量，变量值将为 `null`。
- `inc/dec` 在原值不可解析为数值时，按 `0.0` 起步进行运算，结果以数值形式存储。
- `add` 会将非列表旧值转换为单元素列表再追加；列表仅存储字符串。
- 数据由 `PlayerDataStore` 管理，生命周期覆盖服务器运行期（关闭前持久化）。

---

## 🔗 相关语句

- [var - 变量定义](var—定义_覆盖上下文变量.md)
- [⚖️ ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md)
- [️message - 发送消息](️message—发送消息语句.md)

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-30 12:20:00</p>
</div>


