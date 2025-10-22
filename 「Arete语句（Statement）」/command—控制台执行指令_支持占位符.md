# 🧾 command — 控制台执行指令 / 支持占位符

## 🧩 功能简介
`command` 用于**以控制台身份**执行一条服务器指令。  
支持两类占位符：

+ **内置**：`${player}` → 当前语句上下文中的玩家名（`ctx.playerOrNull()?.name`，无则退化为 `ctx.source.name`）
+ **PlaceholderAPI**：当 `source` 是玩家时，对整行进行 `PAPI` 替换（`%player_name%`、自定义变量等）

由于以**控制台**身份执行，**不受权限限制**（除非插件自行校验）。

---

## ⚙️ 语法
```plain
command { cmd = "要执行的完整指令（不要带/）" }
```

+ `cmd`（必填，字符串）：最终发送给 `Bukkit.dispatchCommand(console, cmd)` 的指令文本。
+ 不要写前导斜杠 `/`，Bukkit API 会当做纯文本处理。

---

## 🔢 占位符与替换时机
执行前，会按以下顺序处理占位符：

1. `**${player}**` —— 先替换为触发者玩家名；若不存在玩家对象，退化为 `ctx.source.name`。
2. **PAPI 替换** —— 仅当 `ctx.source` 是 `Player` 时，整行调用 `replacePlaceholder(source)`，例如：
    - `%player_name%`、`%cmi_user_balance%`、`%luckperms_primary_group_name%` 等等

⚠️ 如果 `source` 不是玩家（如实体、盔甲架、控制台），**不会**进行 PAPI 替换，但 `${player}` 仍会被替换。

---

## ⚡ 快速示例
### ① 发送公告
```plain
command { cmd = "broadcast &a${player} 触发了技能！" }
```

### ② 发放经济
```plain
command { cmd = "eco give ${player} 500" }
```

### ③ 结合 PAPI：带变量的私信
当执行者是玩家时，以下占位符会替换：

```plain
command { cmd = "tell ${player} 你的主世界坐标: %player_world% %player_x% %player_y% %player_z%" }
```

### ④ MythicMobs 技能触发（控制台身份）
```plain
command { cmd = "mm mobs cast SkillName ${player}" }
```

### ⑤ 连携链路：命中 → 加分 → 公告
```plain
seq {
  raycast { range = 15 storeEntity = "hit" }

  # 命中才执行
  if { cond = "${hit} != null" } then {
    command { cmd = "points add ${player} 3" }
    command { cmd = "broadcast &e${player} 命中了目标，+3 点数！" }
  }
}
```

---

## 🧠 进阶用法建议
+ **与语句链搭配**：常与 `if`、`raycast`、`target-entity`、`damage` 等组合，实现“命中→结算→公告/奖励”的闭环。
+ **变更执行身份**：当前实现**固定控制台**执行；若要“以玩家身份”执行，请单独实现 `player-command` 语句（`player.performCommand(...)`）。
+ **安全与审计**：
    - 避免让不可信配置直接拼接 `cmd`，以防注入风险。
    - 需要审计可在语句执行前后加自定义 `log` 语句记录。

---

## 📏 常见坑位
+ **不要带 **`**/**`：`cmd = "say hello"` ✅；`cmd = "/say hello"` ❌
+ **PAPI 替换条件**：只有当 `source` 是 `Player` 时才会生效；否则只会替换 `${player}`。
+ **跨插件参数**：不同插件对控制台/玩家执行身份要求不同，请查阅目标插件文档；必要时改为玩家执行版本的语句。

---

## 🧪 综合示例：击杀结算
```plain
seq {
  # 假设 elsewhere: 已把最后击杀者名存入 killOwner 变量
  if { cond = "${killOwner} != null" } then {
    command { cmd = "eco give ${player} 200" }           # 给触发技能的玩家钱
    command { cmd = "broadcast &6${player} 击杀成功，奖励 200！" }
    command { cmd = "lp user ${player} parent add killer" }  # 临时授予称号（示例）
  }
}
```

---

## ✅ 小结
+ **定位**：以控制台执行一条指令，内置 `${player}`，玩家源上下文下支持 `PAPI`。
+ **场景**：奖励、公告、权限发放、跨插件联动的“桥梁语句”。
+ **搭配**：建议与 `if / delay / seq / raycast / target-entity` 一起用，形成完整技能与结算链路。



> 更新: 2025-10-17 14:41:31  
> 原文: <https://www.yuque.com/yuazer/blow95/vr78e38r3tmag6cq>