# 🗡️ damage — 造成伤害

## 简介
`damage` 用于对一个或一组实体造成伤害。  
它支持 4 种目标选取方式：

1. 指定变量中的实体集合（`targetVar`，**优先级最高**）
2. 指针变量（`target = "pointer"`）
3. 直接对象（`target = "self"` / `target = "target"`）
4. 按坐标+范围临近搜索（省略 `target` 与 `targetVar` 时）

只有 `LivingEntity` 才会被真正伤害；`damager` 始终是施法者 `ctx.source`。

---

## 语法
```plain
damage {
    amount = 9.5         # 伤害数值（Double）
    # 目标四选一（优先级从高到低）：
    # 1) targetVar = "变量名"
    # 2) target = "pointer" | "self" | "target"
    # 3) nearby：不填 target/targetVar 时，使用 at/rx/ry/rz 选取范围内实体
    at = "@self"         # nearby 模式的中心（可省略，默认 @self）
    rx = 4.0             # X 半径
    ry = 2.0             # Y 半径
    rz = 4.0             # Z 半径
    # cause = "自定义标记"   # 预留字段（当前未参与逻辑）
}
```

---

## 参数
| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `amount` | Double | `1.0` | 伤害值 |
| `targetVar` | String? | `null` | **优先使用**。变量名指向 `Entity`<br/> 或 `Collection<Entity>` |
| `target` | String? | `null` | `"pointer" / "self" / "target"` |
| `at` | String? | _(解析为位置)_ | 用于 nearby 模式的中心，支持 `@self/@target/${var}/坐标` |
| `rx` | Double | `4.0` | nearby 的 X 半径 |
| `ry` | Double | `2.0` | nearby 的 Y 半径 |
| `rz` | Double | `4.0` | nearby 的 Z 半径 |
| `cause` | String? | `null` | 预留，用于你的扩展标记 |


解析顺序：先看 `targetVar` → 再看 `target` → 否则进入 **nearby** 模式（`at+rx/ry/rz`）。

---

## 常用示例
### 1) 对变量中的实体群体造成伤害（推荐）
通常搭配 `target-entity` 使用：

```plain
target-entity {
    center = "@self"
    radius = 6.0
    type = "living"
    includeSelf = false
    alive = true
    storeList = "victims"
}

damage {
    amount = 9.5
    targetVar = "victims"
}
```

### 2) 指针模式（`pointer`）
```plain
# 某处将 pointer 指向了一个实体（比如最近目标）
damage {
    amount = 6.0
    target = "pointer"
}
```

### 3) 直接指定 self / target
```plain
seq {
    damage { amount = 2.0; target = "self" }    # 自残（演示用）
    damage { amount = 8.0; target = "target" }  # 伤害当前目标
}
```

### 4) nearby 范围伤害（未提供 target/targetVar）
```plain
damage {
    amount = 5.0
    at = "@self"
    rx = 4.0
    ry = 2.0
    rz = 4.0
}
```

### 5) 和 `raycast` 联动（远程指向伤害）
```plain
raycast { range = 20; storePos = "hitPos" }

damage {
    amount = 7.0
    at = "${hitPos}"
    rx = 2.5
    ry = 2.0
    rz = 2.5
}
```

---

## 与其它语句的组合
### 搭配 `effect` / `knockback`
```plain
target-entity { center = "@self"; radius = 5; type = "living"; includeSelf = false; storeList = "victims" }

parallel {
    damage    { amount = 9.0; targetVar = "victims" }
    effect    { type = "GLOWING"; time = 60; amp = 0; targetVar = "victims" }
    knockback { s = 0.6; y = 0.25; targetVar = "victims" }
}
```

### 搭配 `random` 做暴击/变体
```plain
random { weights = "3,1" } {
    seq { damage { amount = 6.0; target = "target" } }          # 普通
    seq { damage { amount = 12.0; target = "target" } }         # 暴击
}
```

### 搭配 `delay` 做多段打击
```plain
seq {
    damage { amount = 4.0; target = "target" }
    delay { ticks = 10 } { damage { amount = 4.0; target = "target" } }
    delay { ticks = 20 } { damage { amount = 8.0; target = "target" } }
}
```



> 更新: 2025-10-17 12:06:16  
> 原文: <https://www.yuque.com/yuazer/blow95/il5irl833eocbbb0>