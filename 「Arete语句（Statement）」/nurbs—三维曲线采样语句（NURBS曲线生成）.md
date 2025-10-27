# 🌀 nurbs — 三维曲线采样语句（NURBS 曲线生成）

## 🧩 功能简介
`nurbs` 用于在脚本中**生成平滑的 3D 曲线路径**，并将结果存入变量（`ctx.vars[store]`），供粒子、投射物、特效等语句沿路径使用。  
内部实现基于 **NURBS（非均匀有理 B 样条）算法**，可生成任意弯曲路径，支持相对玩家朝向或世界坐标模式。

**<font style="color:#DF2A3F;">并且我贴心的为大伙准备了nurbs曲线编辑器：</font>**  
[http://121.196.245.199:11288/Nurbs3D/index.html](http://121.196.245.199:11288/Nurbs3D/index.html)

## ⚙️ 语法（不想看的可以直接往下拉，看示例即可，非常简单）
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `store` | String | ✅ | — | 结果变量名，存入 `ctx.vars[store]`<br/>（类型为 `List<Location>`<br/>） |
| `coord` | String | ❌ | `local` | 坐标系：`local`<br/> = 相对朝向（前/右/上），`world`<br/> = 绝对世界偏移 |
| `at` | String | ❌ | `@self` | 基准点位置，支持 `@self`<br/> / `${var}`<br/> / `world:x,y,z` |
| `points` | String | ✅ | — | 控制点坐标序列，分号分隔；`local`<br/> 模式为 (forward,right,up)，`world`<br/> 模式为 (dx,dy,dz) |
| `degree` | Int | ❌ | 3 | 曲线阶数（1~6），越高越平滑 |
| `precision` | Double | ❌ | 1.0 | 采样精度（越高点越密，CPU 开销略增） |
| `equalSpacing` | Boolean | ❌ | false | 是否等距重采样（粒子更均匀） |
| `spacing` | Double | ❌ | 0.25 | 等距间隔（需 `equalSpacing=true`<br/> 生效） |
| `weights` | String | ❌ | — | 点权重（半透明影响曲率），数量与控制点一致，用 `;`<br/> 分隔 |
| `coordFacing` | String | ❌ | `self` | 当 `coord=local`<br/> 时，使用谁的朝向：`self`<br/> 或 `target` |


##   
⚡ 快速示例
### ① 世界坐标曲线（相对玩家位置）
```plain
nurbs {
  store = "curve"
  coord = "world"
  at = "@self"
  points = "-4,2,0; -2,8,-2; 3,10,0; 6,4,2"
  degree = 3
  precision = 1
}
```

### ② 局部坐标曲线（前进方向）
相对玩家朝向绘制「抛物线」：

```plain
nurbs {
  store = "parabola"
  coord = "local"
  at = "@self"
  points = "0,0,0; 2,0.5,0.5; 4,1.0,1.0; 6,0.5,1.5; 8,0,2.0"
  degree = 3
  precision = 1.2
}
```

### ③ 等距采样 + 精度增强
```plain
nurbs {
  store = "beam"
  coord = "local"
  at = "@self"
  points = "0,0,0; 3,0.3,0.5; 6,0.8,1.0; 9,0,1.5"
  precision = 1.5
  equalSpacing = true
  spacing = 0.2
}
```

## 🧪 组合示例
### 🌌 粒子沿曲线播放（NURBS + 粒子）
```plain
seq {
  nurbs {
    store = "curve"
    coord = "world"
    at = "@self"
    points = "-4.0,4.0,2.0; -2.0,10.0,-2.0; 3.0,10.0,1.0; 8.0,4.0,4.0"
    degree = 3
    precision = 1.2
    equalSpacing = true
    spacing = 0.25
  }

  # 等待异步采样完成
  delay { ticks = 2 }

  particle {
    type = "FLAME"
    count = 3
    path = "curve"
    step = 1
    interval = 1
  }
}
```

效果：

🔥 粒子会沿玩家周围生成的平滑曲线逐点播放，形成如火焰丝带般的动态轨迹。

---

### ⚔️ 动态攻击演示（NURBS + 粒子 + 沿着路径伤害）
```plain
seq {
    nurbs {
      store = "slash"
      coord = "local"
      at = "@self"
      points = "3.00,0.00,-2.00; 6.00,0.00,-2.00; 8.00,0.00,-2.00"
      degree = 1
      precision = 0.8
      equalSpacing = false
      spacing = 0.25
    }
    delay { tick = 10 } {
        particle {
        type = "SWEEP_ATTACK"
        count = 2
        path = "slash"
        step = 2
        interval = 1
      }
      damage { target = "nearby"; rx = 2; ry = 1; rz = 2; amount = 8.0; path = "slash" }
    }
  }
```

---

### 🌠 多段曲线链（多段 NURBS + 不同特效）
```plain
seq {
  nurbs { store = "arc1"; coord = "local"; points = "0,0,0; 3,1,1; 6,0,2"; precision = 1.2 }
  nurbs { store = "arc2"; coord = "local"; points = "0,0,0; -3,1,1; -6,0,2"; precision = 1.2 }
  delay { ticks = 2 }

  particle { type = "END_ROD"; count = 1; path = "arc1"; step = 1; interval = 1 }
  particle { type = "SPELL_WITCH"; count = 2; path = "arc2"; step = 2; interval = 1 }
}
```



> 更新: 2025-10-21 19:03:33  
> 原文: <https://www.yuque.com/yuazer/blow95/dn7nz2gkvz8z9rbm>