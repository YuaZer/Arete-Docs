# 💖 heal — 恢复生命

## 简介
`heal` 用于为一个或一组实体恢复生命值（回血）。  
它支持与 `damage` 语句**完全一致的目标选取方式**，并且同样支持路径模式 (`path`) 扫描。  

当执行时，会对每个目标触发 `EntityRegainHealthEvent`（类型为 `CUSTOM`），  
若事件未被取消，则根据 `amount` 调整血量。  

---

## 语法
```plain
heal {
    amount = 6.0          # 回复血量（Double，1 = 半颗心）
    # 目标四选一（优先级从高到低）：
    # 1) targetVar = "变量名"
    # 2) target = "pointer" | "self" | "target"
    # 3) nearby：不填 target/targetVar 时，使用 at/rx/ry/rz 选取范围内实体
    at = "@self"         # nearby 模式的中心（可省略，默认 @self）
    rx = 4.0             # X 半径
    ry = 2.0             # Y 半径
    rz = 4.0             # Z 半径
}
```

---

## 参数
| 参数名          | 类型      | 默认值       | 说明                                                  |
|--------------|---------|-----------|-----------------------------------------------------|
| `amount`     | Double  | `1.0`     | 回复的生命值（1 = 半颗心）                                     |
| `targetVar`  | String? | `null`    | **优先使用**。变量名指向 `Entity`<br/> 或 `Collection<Entity>` |
| `target`     | String? | `null`    | `"pointer" / "self" / "target"`                     |
| `at`         | String? | _(解析为位置)_ | 用于 nearby 模式的中心，支持 `@self/@target/${var}/坐标`        |
| `rx`         | Double  | `4.0`     | nearby 的 X 半径                                       |
| `ry`         | Double  | `2.0`     | nearby 的 Y 半径                                       |
| `rz`         | Double  | `4.0`     | nearby 的 Z 半径                                       |
| `path`       | String? | `null`    | nurbs路径所有接触到的实体                                     |
| `pathRadius` | Double  | `4.0`     | nurbs路径检测实体的体积                                      |

解析顺序：先看 `targetVar` → 再看 `target` → 否则进入 **nearby** 模式（`at+rx/ry/rz`）。

---

## 常用示例
### 1) 对变量中的实体群体恢复生命（推荐）
通常搭配 `target-entity` 使用：

```plain
target-entity {
    center = "@self"
    radius = 6.0
    type = "living"
    includeSelf = false
    alive = true
    storeList = "allies"
}

heal {
    amount = 5.0
    targetVar = "allies"
}
```

### 2) 指针模式（`pointer`）
```plain
# 某处将 pointer 指向了一个实体
heal {
    amount = 4.0
    target = "pointer"
}
```

### 3) 直接指定 self / target
```plain
seq {
    heal { amount = 6.0; target = "self" }    # 自己回血
    heal { amount = 3.0; target = "target" }  # 治疗目标
}
```

### 4) nearby 范围治疗（未提供 target/targetVar）
```plain
heal {
    amount = 5.0
    at = "@self"
    rx = 4.0
    ry = 2.0
    rz = 4.0
}
```

### 5) 路径治疗（与粒子路径联动）
```plain
# 前面某处将路径采样进 vars.curve
heal {
    amount = 4.0
    path = "curve"
    pathRadius = 1.2
}
```

---

## 与其它语句的组合
### 搭配 `effect` / `message`
```plain
target-entity { center = "@self"; radius = 5; type = "living"; includeSelf = true; storeList = "friends" }

parallel {
    heal    { amount = 8.0; targetVar = "friends" }
    effect  { type = "HEART"; time = 40; targetVar = "friends" }
    message { text = "&a你恢复了生命！"; targetVar = "friends" }
}
```

### 搭配 `random` 实现治疗暴击
```plain
random { weights = "4,1" } {
    seq { heal { amount = 4.0; target = "self" } }        # 普通治疗
    seq { heal { amount = 10.0; target = "self" } }       # 暴击治疗
}
```

### 搭配 `delay` 做多段持续恢复
```plain
seq {
    heal { amount = 2.0; target = "target" }
    delay { ticks = 20 } { heal { amount = 2.0; target = "target" } }
    delay { ticks = 40 } { heal { amount = 3.0; target = "target" } }
}
```

