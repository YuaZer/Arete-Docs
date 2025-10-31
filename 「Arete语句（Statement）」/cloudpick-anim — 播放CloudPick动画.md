# 🐉 cloudpick-anim — 播放 CloudPick 动画

用于为实体播放 **CloudPick 模型动画**，支持范围、多目标、变量、指针等多种模式。  
功能与 `arcartx-anim` 类似，但底层基于 CloudPick 的 `ModelAPI`。

---

## ⚙️ 参数列表

| 参数名                       | 类型     | 必填 | 默认值     | 说明                                      |
|---------------------------|--------|----|---------|-----------------------------------------|
| `animation`<br/> / `name` | String | ✅  | —       | 动画名（CloudPick 模型定义内的动画）                 |
| `transition`              | Int    | 否  | 1       | 动画过渡时间（tick）                            |
| `speed`                   | Double | 否  | 1.0     | 动画播放速度倍数（旧字段，保留兼容性）                     |
| `keep`                    | Long   | 否  | 1000    | 动画保持时长（旧字段，保留兼容性）                       |
| `target`                  | String | 否  | nearby  | 目标模式：`self`、`target`、`pointer`、`nearby` |
| `targetVar`               | String | 否  | —       | 从变量池读取目标（优先级最高）                         |
| `at`                      | String | 否  | `@self` | 搜索中心位置                                  |
| `rx`                      | Double | 否  | 4.0     | 搜索范围 X                                  |
| `ry`                      | Double | 否  | 2.0     | 搜索范围 Y                                  |
| `rz`                      | Double | 否  | 4.0     | 搜索范围 Z                                  |

---

## ⚡ 快速示例

### ① **命中播放动画**

```plain
raycast { range = 20 storeEntity = "hit" }
cloudpick-anim { animation = "HitReact_Small" transition = 2 targetVar = "hit" }
```

> 当命中实体时，为该目标播放 CloudPick 动画 `HitReact_Small`。

---

### ② **自身待机动画**

```plain
cloudpick-anim { animation = "Idle_Breath" transition = 5 target = "self" }
```

> 让执行者自身播放待机动画 `Idle_Breath`。

---

### ③ **范围群体动画**

```plain
cloudpick-anim {
  animation = "Spawn_In"
  transition = 3
  target = "nearby"
  at = "@self"
  rx = 6
  ry = 3
  rz = 6
}
```

> 为周围 6×3×6 范围内所有实体播放动画 `Spawn_In`。

---

## 🧠 机制说明

- `targetVar` 优先级最高，可指定变量池中的目标（`Entity` 或 `Collection<Entity>`）。  
- 旧字段 `speed` 和 `keep` 已被保留，仅用于兼容旧版脚本，不再影响实际逻辑。  
- 执行时自动在异步线程中运行，不会阻塞主线程。  
- 仅对 `LivingEntity` 有效。

---

## 🧩 CloudPick API 对应调用

服务端执行逻辑：

```kotlin
ModelAPI.setEntityAnimation(living, animation, transitionTime)
```

客户端会自动执行该模型动画。

---

## 📚 对比 ArcartX

| 功能 | ArcartX | CloudPick |
| --- | --- | --- |
| 底层接口 | `ArcartXAPI` | `yslelf.cloudpick.api.ModelAPI` |
| 可用实体 | Player / Entity | LivingEntity |
| 动画系统 | 内置骨骼动画 | 模型定义动画 |
| 是否支持动画保持 | ✅ 可设置 keep 时间 | ❌ 由模型本身控制 |

---

> 更新: 2025-11-01 02:21:40  
> 原文: <https://www.yuque.com/yuazer/blow95/cloudpick-anim>
