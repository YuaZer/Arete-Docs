# ⏱️ cooldown — 技能冷却管理

对指定玩家的某个技能冷却进行设置、尝试开启、增加、减少或清除。支持将尝试结果写入变量，便于后续条件判断与分支执行。

---

## 参数一览

| 参数名      | 类型     | 必填 | 默认值   | 说明 |
|-------------|----------|------|---------|------|
| `op`        | String   | 是   | —       | 操作类型：`set` / `try` / `add` / `reduce` / `clear` |
| `skill`     | String   | 是   | —       | 技能标识（ID/名字），由冷却系统识别 |
| `ticks`     | Long     | 否   | `0`     | 冷却时长（单位：tick，20 tick ≈ 1s）。仅 `set`/`try`/`add`/`reduce` 需要 |
| `player`    | String   | 否   | `@self` | 目标玩家解析：`@self` / `@target`（若为空则取施法者） |
| `store`     | String   | 否   | —       | 仅当 `op = try` 时有效：将是否成功开启冷却的结果（`true/false`）写入变量 |

> 冷却时长单位为 Minecraft 服务器 tick：1 秒 = 20 tick。

---

## 快速示例

```plain
cooldown {
  op = "try"             # 若无冷却则开始冷却并返回 true
  skill = "fireball"
  ticks = 80             # 4 秒
  player = "@self"
  store = "cdResult"
}
ifchain {
  case { ${cdResult} } {
    message { text = "&a技能已进入冷却！" }
  }
  else {
    message { text = "&c当前仍在冷却中！" }
  }
}
```

---

## 用法详解

### 1) 设置冷却（覆盖）
```plain
cooldown { op = "set" skill = "dash" ticks = 60 }
```

### 2) 尝试开始冷却（常用于“是否可施法”判断）
```plain
cooldown { op = "try" skill = "dash" ticks = 60 store = "ok" }
ifchain {
  case ${ok} {
    message { text = "&a冲刺释放成功，进入冷却。" }
  }
  else {
    message { text = "&c冲刺仍在冷却中！" }
  }
}
```

### 3) 增加剩余冷却
```plain
cooldown { op = "add" skill = "ice_barrier" ticks = 40 }
```

### 4) 减少剩余冷却
```plain
cooldown { op = "reduce" skill = "ice_barrier" ticks = 20 }
```

### 5) 清除冷却
```plain
cooldown { op = "clear" skill = "ultimate" }
```

### 6) 指定目标玩家
```plain
cooldown { op = "set" skill = "heal" ticks = 40 player = "@target" }
```

---

## ⚠️ 注意事项

- `op` 与 `skill` 为必填；`ticks` 仅对 `set`/`try`/`add`/`reduce` 有效，`clear` 不需要。
- `player` 未提供时默认取当前施法者（`@self`）。若无法解析到玩家，将跳过执行。
- `try` 操作会返回是否成功开启冷却：成功表示之前无冷却或冷却已结束，并已开始新冷却。
- `store` 仅在 `op = try` 时生效，其它操作不会写入变量。

---

## 🔗 相关语句

- [⚖️ ifchain - 多分支条件](ifchain—多分支条件执行_case…else链.md)
- [️message - 发送消息](️message—发送消息语句.md)
- [var - 变量定义](var—定义_覆盖上下文变量.md)

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-30 12:10:00</p>
</div>


