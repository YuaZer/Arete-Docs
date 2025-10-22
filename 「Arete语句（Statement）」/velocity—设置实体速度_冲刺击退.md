# 💨 velocity — 设置实体速度 / 冲刺击退

直接为一个或一组实体**赋值**速度向量（`Vector(x,y,z)`），可实现冲刺、击飞、上挑、水平击退等效果。

### 🎯 功能简介
`velocity` 语句用于**设置或叠加实体的速度向量**。  
常用于制作击退、弹飞、冲刺、突进、飘浮等物理位移效果。

语法：

```plain
velocity {
  x = <Double>          # X 方向速度（左右）
  y = <Double>          # Y 方向速度（上下）
  z = <Double>          # Z 方向速度（前后）
  target = "self|target|pointer"  # 作用目标（单体）
  targetVar = "变量名"             # 从 ctx.vars 中读取目标（单体或列表）
  add = true|false      # 是否叠加原速度，默认 false（覆盖）
  mul = <Double>        # 向量整体缩放系数，默认 1.0
}
```

---

## 🧩 参数说明
| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `x`<br/>, `y`<br/>, `z` | Double | 各方向速度分量 |
| `target` | String? | `self`<br/> / `target`<br/> / `pointer`<br/>，指定固定目标 |
| `targetVar` | String? | 变量名，从上下文取出 Entity 或列表 |
| `add` | Boolean | 是否在原有速度上叠加（默认 false = 覆盖） |
| `mul` | Double | 缩放整个速度向量，如 1.5 表示放大 50%，默认 1.0 |


---

## 🧠 用法举例
### ① 自己向上弹飞（轻跳）
```plain
velocity { x = 0 y = 0.6 z = 0 }
```

📖 等价于 `玩家.setVelocity(Vector(0, 0.6, 0))`

---

### ② 向前冲刺（带缩放）
```plain
velocity { x = 0 z = 1.0 y = 0.3 mul = 1.5 }
```

⚙️ 结果向量为 `(0, 0.3, 1.0) × 1.5 = (0, 0.45, 1.5)`  
玩家获得更强冲刺。

---

### ③ 对射线命中的实体进行上挑
```plain
raycast { range = 10 storeEntity = "pointer" }
velocity { x = 0 y = 0.9 z = 0 target = "pointer" }
```

---

### ④ 对范围内所有实体群体击飞
```plain
target-entity {
  center = "@self"
  radius = 3.5
  type = "living"
  includeSelf = false
  storeList = "victims"
}
velocity { x = 0.3 y = 0.7 z = 0.3 targetVar = "victims" mul = 0.8 }
```

✅ 所有选中目标都会被轻微击退 + 上挑。

---

### ⑤ 射线突进（含叠加速度）
```plain
seq {
  raycast { range = 12 storePos = "dashPos" storeEntity = "dashTarget" }

  # 自己向前突进（叠加原速度）
  velocity { x = 0 y = 0.4 z = 1.0 add = true mul = 1.2 }

  # 命中的目标被击退
  velocity { x = 0 y = 0.3 z = -1.1 targetVar = "dashTarget" }
}
```

🌀 使用 `add = true` 让冲刺自然过渡，不会打断已有速度。

---

### ⑥ 连段组合：轻抬 + 冲刺 + 回拉
```plain
seq {
  velocity { x = 0 y = 0.6 z = 0 }        # 上挑
  delay 3
  velocity { x = 0 z = 1.0 y = 0.2 mul = 1.2 }  # 向前突进
  delay 2
  velocity { x = 0 z = -0.8 y = 0 add = true }  # 回拉
}
```

---

## ⚡ 进阶技巧
### 🧭 朝向冲刺
若你有保存玩家朝向向量（例如存入 `dashVec`）：

```plain
velocity {
  x = ${dashVec.x}
  y = 0.3
  z = ${dashVec.z}
  mul = 1.4
}
```

---

### 🧮 连续加速（add 的妙用）
```plain
# 每次循环叠加一点前进速度，实现蓄力突进
repeat times=5 delay=1 {
  velocity { x = 0 z = 0.25 add = true }
}
```

---

### 📏 数值建议
| 分量 | 推荐范围 | 说明 |
| --- | --- | --- |
| `x`<br/>, `z` | 0.2 ~ 1.2 | 水平冲击力 |
| `y` | 0.2 ~ 1.0 | 垂直弹跳 |
| `mul` | 0.5 ~ 2.0 | 缩放比例（>2 可能触发反作弊） |
| 过大向量 | ⚠️ 超过 ±1000 自动防护，不会执行 | |


---

### 🧱 与其他语句配合
+ `raycast` → 锁定目标
+ `target-entity` → 范围群体击退
+ `delay` → 分段控制节奏
+ `parallel` → 同时执行视觉粒子与位移

---

## ✅ 综合示例：风遁·疾影突斩
```plain
seq {
  # 1. 射线检测
  raycast { range = 10 storeEntity = "victim" storePos = "hitPos" }

  # 2. 自身突进（叠加原速度）
  velocity { x = 0 y = 0.3 z = 1.2 add = true mul = 1.2 }

  # 3. 命中目标击退
  velocity { x = 0 y = 0.5 z = -1.0 targetVar = "victim" }

  # 4. 特效与伤害
  particle { type = "crit" at = "@self" }
  damage { amount = 7.5 targetVar = "victim" }
}
```

## 调试清单（Checklist）
+ 确认 `targetVar`/`target` 是否解析到了**实体或实体列表**。
+ 数值是否过大导致玩家被反作弊拦截？
+ 若与其他速度修改语句同帧执行，是否存在相互覆盖？
+ 是否需要“叠加速度”？

---

## 最小可用模板
**对自己：**

```plain
velocity { x = 0 y = 0.8 z = 0 }
```

**对最近单体：**

```plain
target-entity { center = "@self" radius = 3 type = "living" includeSelf = false alive = true store = "wind_primary" }
velocity { x = 0 y = 0.6 z = -0.8 targetVar = "wind_primary" }
```

**对范围列表：**

```plain
target-entity { center = "@self" radius = 4 type = "living" includeSelf = false alive = true storeList = "wind_victims" }
velocity { x = 0.2 y = 0.5 z = 0.2 targetVar = "wind_victims" }
```



> 更新: 2025-10-17 14:36:08  
> 原文: <https://www.yuque.com/yuazer/blow95/dabixwy59tcomsxb>