# 🌌 particle — 粒子特效语句

## 简介
`particle` 语句用于在指定位置生成粒子效果。  
这是 **视觉特效类技能** 的核心组件之一，可用于：

+ 展示技能轨迹、爆炸、冲击波、法阵等效果；
+ 制作炫酷的技能动画（如“陨石坠落”“能量波动”等）；
+ 增强玩家动作反馈与临场感。

你可以通过调整粒子类型、偏移量、数量和速度，  
组合出无数不同的视觉表现。



粒子种类：

[https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Particle.html](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Particle.html)

---

## 语法
```plain
particle {
    type = "FLAME"
    count = 50
    ox = 0.2
    oy = 0.6
    oz = 0.2
    speed = 0.05
    at = "@self"
}
```

在玩家位置周围生成火焰粒子，模拟燃烧的效果。

---

## 参数说明
## ⚙️ 语法
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | String | ✅ | — | 粒子类型（如 `FLAME`<br/>, `END_ROD`<br/>…） |
| `count` | Int | ❌ | `20` | 数量（Bukkit 的 `count`<br/>） |
| `ox/oy/oz` | Double | ❌ | `0.2/0.2/0.2` | 偏移（Bukkit 的 offsetX/Y/Z） |
| `speed` | Double | ❌ | `0.0` | 速度（Bukkit 的 speed） |
| `at` | String | ❌ | `@self` | 单点模式的坐标；支持 `@self / ${var} / world:x,y,z` |
| `path` | String | ❌ | — | **路径变量名**：应为 `List<Location>`<br/>（如 `nurbs`<br/> 产物） |
| `step` | Int | ❌ | `1` | **抽样步长**：每 `step`<br/> 个点播放 1 次（≥1） |
| `interval` | Int | ❌ | `0` | **逐点间隔**（tick）：`0`<br/>=同 tick 一次性播放；>0=逐点播放 |


行为优先级：当 `path` 存在且类型正确（列表非空），走**路径模式**；否则**回退单点模式**。

## 特别说明
+ `at` 参数支持以下几种写法：

| 示例 | 说明 |
| --- | --- |
| `@self` | 玩家自身位置 |
| `@target` | 当前目标实体位置 |
| `${aimPos}` | 从变量中读取保存的坐标 |
| `100,65,-20` | 世界坐标 |
| `~0,1,~0` | 相对玩家位置（上方 1 格） |


---

## 示例
### 🔥 基础示例：玩家身边的火焰
```plain
particle {
    type = "FLAME"
    count = 80
    ox = 0.3
    oy = 1.0
    oz = 0.3
    speed = 0.05
    at = "@self"
}
```

效果：在玩家身边生成火焰粒子，模拟能量燃烧。

---

### 💥 能量爆炸
```plain
particle {
    type = "EXPLOSION_LARGE"
    count = 5
    at = "${impactPos}"
}
```

在技能命中的位置 (`impactPos`) 生成爆炸特效。

---

### 🌈 闪光轨迹（配合 `seq`）
```plain
seq {
    particle { type = "END_ROD"; count = 10; at = "@self" }
    delay { ticks = 5 } {
        particle { type = "END_ROD"; count = 10; at = "@target" }
    }
}
```

在施法者与目标之间形成延迟闪烁的轨迹。

---

### ⚡ 并行动画（配合 `parallel`）
```plain
parallel {
    particle { type = "CRIT"; count = 20; at = "@self" }
    particle { type = "SMOKE_NORMAL"; count = 10; at = "@self" }
}
```

同时生成两种不同类型的粒子，形成复合效果。



## 高级技巧
### 🌀 使用变量控制粒子位置
```plain
raycast { range = 20; storePos = "hitPos" }
particle { type = "CRIT_MAGIC"; count = 30; at = "${hitPos}" }
```

粒子在 `raycast` 命中的位置生成，  
可实现“指向技能”或“远程法术”效果。

---

### 💫 创建能量波动画
```plain
seq {
    particle { type = "CLOUD"; count = 50; ox = 1; oy = 0.5; oz = 1; at = "@self" }
    delay { ticks = 10 } {
        particle { type = "CLOUD"; count = 80; ox = 1.5; oy = 0.7; oz = 1.5; at = "@self" }
    }
    delay { ticks = 20 } {
        particle { type = "CLOUD"; count = 120; ox = 2; oy = 1.0; oz = 2; at = "@self" }
    }
}
```

分段延迟产生更大范围的粒子，营造能量扩散的波纹效果。

---

### 🎯 多段特效组合
```plain
parallel {
    particle { type = "CRIT"; count = 20; ox = 0.2; oy = 0.5; oz = 0.2; at = "@self" }
    particle { type = "SPELL_WITCH"; count = 10; at = "@self" }
    delay { ticks = 10 } {
        particle { type = "SMOKE_LARGE"; count = 15; at = "@self" }
    }
}
```

多种粒子配合延迟与并行，实现立体视觉特效。

## ⚡ 与nurbs组合示例
### 1) 兼容老写法（单点播放）
```plain
particle { type = "CLOUD"; count = 50; ox = 1; oy = 0.5; oz = 1; at = "@self" }
```

### 2) 沿路径一次性“刷轨迹”（同 tick）
```plain
# 假设 curve 为 List<Location>
particle {
  type = "END_ROD"
  count = 1
  path = "curve"
  step = 1
  interval = 0
}
```

### 3) 沿路径逐点“流动”（每点 1 tick）
```plain
particle {
  type = "FLAME"
  count = 1
  path = "curve"
  step = 1
  interval = 1
}
```

### 4) 抽样降载（每 3 个点取 1 个）
```plain
particle {
  type = "CRIT"
  count = 1
  path = "curve"
  step = 3
  interval = 0
}
```

---

## 🔗 与 `nurbs` 的组合示例（建议加入 `delay`）
### A. 世界坐标曲线 + 流动粒子
```plain
seq {
  nurbs {
    store = "curve"
    coord = "world"
    at = "@self"
    points = "-4,4,2; -2,10,-2; 3,10,1; 8,4,4"
    degree = 3
    precision = 1.2
    equalSpacing = true
    spacing = 0.25
  }

  # nurbs 异步 → 等 1~2 tick
  delay { ticks = 2 }

  particle {
    type = "FLAME"
    count = 2
    path = "curve"
    step = 1
    interval = 1
  }
}
```

### B. 本地坐标曲线 + 一次性刷出能量弧
```plain
seq {
  nurbs {
    store = "arc"
    coord = "local"
    at = "@self"
    points = "0,0,0; 2,0.6,0.5; 5,0.8,1.2; 8,0.2,1.8"
    precision = 1.5
    equalSpacing = true
    spacing = 0.2
  }
  delay { ticks = 1 }

  particle { type = "SPELL_WITCH"; count = 1; path = "arc"; interval = 0 }
}
```

### C. 轨迹 + 末端爆点（路径+单点混用）
```plain
seq {
  nurbs { store = "beam"; coord = "local"; at = "@self"; points = "0,0,0; 4,0.2,0; 8,0,0" }
  delay { ticks = 1 }

  # 轨迹
  particle { type = "END_ROD"; count = 1; path = "beam"; step = 1; interval = 1 }

  # 末端单点爆炸（仍用原来的单点模式）
  particle { type = "EXPLOSION_NORMAL"; count = 8; at = "${beam.last}" }
}
```



> 更新: 2025-10-21 19:05:40  
> 原文: <https://www.yuque.com/yuazer/blow95/cgkls8ol3z5c2gpi>