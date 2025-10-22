# 💥 knockback — 按方向击退目标 / 震退敌人

## 🧩 功能简介
`knockback` 用于**根据施法者与目标的相对位置，自动计算方向并施加击退力**。  
它非常适合制作“击飞”“挑退”“环形震退”等战斗效果。

与 `velocity` 不同：

+ `velocity` → 你手动指定 `(x, y, z)` 向量；
+ `knockback` → 自动根据方向（施法者 → 目标）计算。

---

## ⚙️ 语法格式
```plain
knockback {
  target = "self|target|pointer"   # 固定目标（单体模式）
  targetVar = "变量名"              # 从上下文变量取目标（单体或列表）
  s = 0.5                          # 或 strength = 0.5，水平击退强度
  y = 0.2                          # 垂直上挑分量
}
```

---

## 🔢 参数说明
| 参数 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `target` | String | `"self"` | 固定目标：`self`<br/> / `target`<br/> / `pointer` |
| `targetVar` | String? | `null` | 从上下文变量中取目标（优先级高于 target） |
| `s`<br/> 或 `strength` | Double | 0.5 | 击退强度（水平分量长度） |
| `y` | Double | 0.2 | 垂直分量（上挑高度） |


✅ **方向自动计算**：`受击者位置 - 施法者位置`。  
若两者重叠，则使用施法者朝向；若依然为零，则兜底 `(0, 0, 1)`。

---

## ⚡ 快速示例
### 🧍‍♂️ 击退当前锁定目标
```plain
knockback { target = "target" s = 0.8 y = 0.25 }
```

### 🎯 对射线命中的目标击退
```plain
raycast { range = 12 storeEntity = "pointer" }
knockback { target = "pointer" strength = 1.0 y = 0.3 }
```

### 🌪 范围环形震退
```plain
target-entity {
  center = "@self"
  radius = 4.0
  type = "living"
  includeSelf = false
  storeList = "victims"
}
knockback { targetVar = "victims" s = 0.9 y = 0.25 }
```

---

## 🧠 组合技巧
### ✴ 连段：上挑 → 击退
```plain
seq {
  velocity { x = 0 y = 0.6 z = 0 target = "target" }
  delay 2
  knockback { target = "target" s = 1.0 y = 0.25 }
}
```

### 💨 射线突进 + 命中震退
```plain
seq {
  raycast { range = 10 storeEntity = "hit" }

  velocity { x = 0 y = 0.3 z = 1.2 mul = 1.1 }     # 自身突进
  knockback { targetVar = "hit" s = 1.1 y = 0.3 }  # 目标震退
  velocity { x = 0 y = 0 z = 0 }                   # 刹车
}
```

### 💣 环形爆破（视觉与音效）
```plain
seq {
  target-entity { center = "@self" radius = 4.5 type = "living" includeSelf = false storeList = "victims" }
  particle { type = "explosion" at = "@self" }
  sound { key = "entity.generic.explode" v = 1 p = 1 at = "@self" }
  knockback { targetVar = "victims" s = 1.2 y = 0.28 }
}
```

---

## 📊 参数建议
| 类型 | 推荐范围 | 说明 |
| --- | --- | --- |
| `s`<br/> / `strength` | 0.5 ~ 1.2 | 水平击退力度 |
| `y` | 0.15 ~ 0.35 | 垂直上挑手感 |
| ⚠️ 注意 | 超过 1.5 可能触发反作弊 / 橡皮筋 | |


---

## 💡 技巧提示
+ 群体击退时，`targetVar` 支持任意实体集合（如 `storeList` 保存的结果）。
+ 若要“叠加速度”而不是覆盖，可在 `knockback` 后再调用：

```plain
velocity { add = true x = 0 y = 0 z = 0.3 targetVar = "victims" }
```

+ 可搭配 `damage` / `particle` / `sound` / `delay` 形成完整战斗链。

---

## 🔮 综合示例：地爆震·破
```plain
seq {
  target-entity { center = "@self" radius = 5.0 type = "living" includeSelf = false storeList = "victims" }
  particle { type = "cloud" at = "@self" }
  sound { key = "entity.ender_dragon.growl" v = 1 p = 0.8 at = "@self" }

  knockback { targetVar = "victims" s = 0.6 y = 0.32 }
  delay 3
  knockback { targetVar = "victims" s = 1.1 y = 0.24 }
  damage { amount = 6.0 targetVar = "victims" }
}
```

---

## 📘 总结
**💥**** knockback** 是最常用的战斗位移语句之一，  
可与 `velocity`、`damage`、`raycast`、`target-entity` 等组合形成复杂的技能链路。  
善用 `s` 与 `y` 的比例，可实现“上挑”“击退”“爆震”“后坐”等高级打击感！



> 更新: 2025-10-17 14:38:41  
> 原文: <https://www.yuque.com/yuazer/blow95/bkq50c12gw86ogq6>