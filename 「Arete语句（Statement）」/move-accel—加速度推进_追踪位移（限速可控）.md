# 🚀 move-accel — 加速度推进 / 追踪位移（限速可控）

## 🧩 功能简介
`move-accel` 让实体以**逐帧加速**的方式朝某**终点**或**方向向量**移动，并支持：

+ **两种驱动模式**：`to`（追踪到点）或 `dir`（按方向推进）
+ **两类实体处理**：
    - `ArmorStand` → **逐 tick 位置推进 + 主线程 teleport**（轨迹丝滑、可穿模展示体）
    - 其他实体 → **逐 tick 设置 velocity**（与物理/反作弊更一致）
+ **速度模型**：每 tick 速度 `v += a * dir`，并以 `max` 限速
+ **重力控制**：执行期 `gravity` 可开关，结束后按 `restoreGravity` 决定是否恢复

---

## ⚙️ 语法
```plain
move-accel {
  # 目标实体（优先级：targetVar > mode）
  target     = "变量名"         # ctx.vars 中的 Entity，如 "s1"、"lock"
  mode       = "self|target|pointer"  # 当未提供 targetVar 时，从上下文选择

  # 推进方式（二选一）
  to         = "@self|${pos}|world:x,y,z"  # 追踪到指定位置（含端点阈值）
  dx = 0.0
  dy = 0.0
  dz = 1.0                                  # 或使用方向推进：dir = (dx,dy,dz)

  # 运动参数
  accel      = 0.12       # 每 tick 加速度
  max        = 1.6        # 速度上限（>0 时生效）
  duration   = 14         # 持续 tick 数（至少 1）
  stopAt     = 0.35       # 仅 to 模式：抵达阈值（欧氏距离）

  # 重力
  gravity        = false  # 执行期间是否启用重力
  restoreGravity = true   # 结束后是否恢复原重力
}
```

### 参数要点
+ **目标选择**：
    - `target`（即代码里的 `targetVar`）优先；否则看 `mode`：`self / target / pointer`
    - `pointer` 通常由 `raycast { storeEntity = "..." }` 或自动指针写入
+ **to vs dir**：必须**二选一**；都缺少则不执行
+ **限速逻辑**：当 `max > 0` 时才会生效；`max = 0` 表示**不限制**（可能过快）
+ **ArmorStand 专用**：采用**内存位置累加 → teleport**的方式，轨迹平滑、穿模友好

---

## ⚡ 快速示例
### ① 追踪锁定目标（命中则自动跟随）
```plain
raycast { range = 16 storeEntity = "lock" }
move-accel {
  target = "sword"     # 你的飞剑盔甲架
  to = "@lock"         # 追踪命中的实体
  accel = 0.10
  max = 1.4
  duration = 30
  gravity = false
}
```

### ② 沿朝向突刺（方向推进）
```plain
# 取玩家朝向向量（假设已有语句把面向向量写入 dashVec）
# dx/dy/dz 可直接写常量，也可用变量插值
move-accel {
  mode = "self"  # 推进自己
  dx = ${dashVec.x}
  dy = 0.0
  dz = ${dashVec.z}
  accel = 0.15
  max = 1.8
  duration = 12
  gravity = false
}
```

### ③ 飞剑多段追踪 + 爆裂
```plain
seq {
  raycast { range = 20 storeEntity = "hit" storePos = "hitPos" }

  ifchain {
    case { cond = "var.hit != null" } {
      move-accel { target = "sword"; to = "@hit"; accel = 0.12; max = 1.6; duration = 25 }
    }
    else {
      move-accel { target = "sword"; to = "${hitPos}"; accel = 0.12; max = 1.6; duration = 20 }
    }
  }

  particle { type = "explosion" at = "${hitPos}" }
  sound    { key = "entity.generic.explode" v = 1 p = 1 at = "${hitPos}" }
}
```

### ④ 拖拽拘束（把敌人拉回自己）
```plain
target-entity { center = "@self" radius = 8 type = "living" includeSelf = false storeList = "v" }
for { times = 1; var = "e" } {
  # 这里假定你在外层把集合拆为单个实体 e
  move-accel { target = "e"; to = "@self"; accel = 0.10; max = 1.2; duration = 15; gravity = false }
}
```

### ⑤ 轨迹弹道（ArmorStand 载体）
```plain
armorstand { id = "bullet"; at = "@self"; marker = true }
move-accel { target = "bullet"; dx = 0 dy = 0 dz = 1 accel = 0.12 max = 2.0 duration = 18 }
```



> 更新: 2025-10-17 15:05:51  
> 原文: <https://www.yuque.com/yuazer/blow95/li0fzlec65hin2un>