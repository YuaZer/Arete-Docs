# 🧭 move — 平滑移动实体 / 插值位移（含缓动）

## 🧩 功能简介
`move` 用来在 **若干 tick 内** 将某个实体从**当前坐标**平滑移动到**目标坐标**，并可选用不同 **缓动曲线**（`linear / easein / easeout / easeinout`）来控制加速/减速手感。  
常用于：**飞剑/特效盔甲架轨迹**、Boss 漂移、子弹/法阵移动、镜头载具平移等。

---

## ⚙️ 语法
```plain
move {
  target = "变量名"      # ctx.vars 中保存的 Entity（如 armorstand/spawn 出来的 s1）
  to = "@self|${pos}|x,y,z|选择器"  # 终点：支持选择器、变量、绝对/相对坐标解析
  move = 10             # 持续 tick 数（≥1），默认 10
  easing = "linear"     # 缓动：linear / easein / easeout / easeinout
}
```

### 参数说明
| 键 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `target` | String | ✅ | `ctx.vars[target]`<br/> 必须是 `Entity`<br/>（否则直接返回） |
| `to` | String | ✅ | 目标位置，交给 `ctx.resolveLocation(to)`<br/> 解析（可用 `@self`<br/>、`${posVar}`<br/>、选择器、`world,x,y,z`<br/> 等） |
| `move` | Int | 否 | 补间总时长（tick）。每 tick 线性插值一次并 `teleport`<br/>。默认 10 |
| `easing` | String | 否 | `linear / easein / easeout / easeinout`<br/>，默认 `linear` |


内部实现：

+ 起点 = 执行当刻实体位置；终点 = `resolveLocation(to)`。
+ 每 tick 计算插值点并 **主线程**`teleport`；同时插值 `yaw/pitch`，使朝向也平滑过渡。
+ 使用 `delayOneTick()` 保证一帧一帧推进。

---

## ⚡ 快速示例
### ① 把盔甲架从 A 点移到 B 点（2 秒）
```plain
# 预先生成并存入变量 s1
armorstand { id = "s1" at = "@self" marker = true }

# 光束终点
raycast { range = 16 storePos = "beamEnd" }

# 平滑移动 40 tick，先慢后快（easein）
move { target = "s1"; to = "${beamEnd}"; move = 40; easing = "easein" }
```

### ② 敌人被“拘束”拉回施法者身边
```plain
target-entity { center = "@self" radius = 8 type = "living" includeSelf = false storeList = "v" }
for { var = "e"; times = 1 } {
  # 这里假设你已把目标实体单个写入 var e，也可以直接 targetVar 方案外层封装
  move { target = "e"; to = "@self"; move = 15; easing = "easeout" }
}
```

### ③ 多段路径（折线巡航）
```plain
# 生成飞剑
armorstand { id = "sword"; at = "@self"; item = "minecraft:iron_sword"; slot = head; marker = false }

# 路点
raycast { range = 10 storePos = "p1" }
raycast { range = 18 storePos = "p2" }
raycast { range = 26 storePos = "p3" }

seq {
  move { target = "sword"; to = "${p1}"; move = 10; easing = "easein" }
  move { target = "sword"; to = "${p2}"; move = 15; easing = "linear" }
  move { target = "sword"; to = "${p3}"; move = 20; easing = "easeout" }
}
```

### ④ 追踪移动 + 末端爆裂
```plain
raycast { range = 20 storeEntity = "lock" storePos = "hitPos" }
ifchain {
  case { cond = "var.lock != null" } {
    move { target = "s1"; to = "@lock"; move = 25; easing = "easeinout" }
  }
  else {
    move { target = "s1"; to = "${hitPos}"; move = 20; easing = "easeout" }
  }
}
particle { type = "explosion" at = "${hitPos}" }
```

---

## 🧪 缓动曲线手感对照
+ `linear`：匀速；轨迹稳、机械感强。
+ `easein`：**起步慢，后段加速**；适合起势感/突刺感。
+ `easeout`：**起步快，末段减速**；适合“刹停到位”的精确落点。
+ `easeinout`：前慢中快后慢；最自然的“往返/巡航/追踪”手感。



> 更新: 2025-10-17 15:02:35  
> 原文: <https://www.yuque.com/yuazer/blow95/ik3pgn8736q3ewxd>