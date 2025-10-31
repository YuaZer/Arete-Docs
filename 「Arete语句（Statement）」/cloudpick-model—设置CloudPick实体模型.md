# 🧩 cloudpick-model — 设置 CloudPick 实体模型

用于为实体（或玩家）设置 **CloudPick 模型**（即绑定 CloudPick 模型资源）。  
支持单体 / 范围 / 指针 / 变量目标，多种选取模式。

---

## ⚙️ 参数列表

| 参数名                               | 类型     | 必填 | 默认值      | 说明                                                                                                        |
|-----------------------------------|--------|----|----------|-----------------------------------------------------------------------------------------------------------|
| `model`<br/> / `id`<br/> / `name` | String | ✅  | —        | 模型 ID（注册于 CloudPick 资源中）                                                                                  |
| `target`                          | String | 否  | `nearby` | 目标模式：<br/>• `self` — 自身<br/>• `target` — 触发目标<br/>• `pointer` — 指针目标（需上下文有 pointer）<br/>• `nearby` — 附近实体 |
| `targetVar`                       | String | 否  | —        | 从变量池读取目标（优先级最高）                                                                                           |
| `at`                              | String | 否  | `@self`  | 搜索中心位置，仅 `nearby` 模式下有效                                                                                   |
| `rx`                              | Double | 否  | 4.0      | 搜索范围 X                                                                                                    |
| `ry`                              | Double | 否  | 2.0      | 搜索范围 Y                                                                                                    |
| `rz`                              | Double | 否  | 4.0      | 搜索范围 Z                                                                                                    |

---

## ⚡ 快速示例

### ① **命中设置 CloudPick 模型**

```plain
raycast { range = 16 storeEntity = "lock" }
cloudpick-model { model = "slime_king" targetVar = "lock" }
```

> 当命中实体时，为该实体应用 CloudPick 模型 `slime_king`。

---

### ② **自身换模**

```plain
cloudpick-model { model = "hero_warrior" target = "self" }
```

> 将执行者自身的 CloudPick 模型更改为 `hero_warrior`。

---

### ③ **范围换模**

```plain
cloudpick-model {
  model = "stone_golem"
  target = "nearby"
  at = "@self"
  rx = 6
  ry = 3
  rz = 6
}
```

> 为周围 6×3×6 范围内所有实体设置 CloudPick 模型 `stone_golem`。

---

## 🧠 机制说明

- `targetVar` 优先级最高，可绑定任意变量中保存的目标（`Entity` 或 `Collection<Entity>`）。
- 在 `nearby` 模式下，`at`、`rx`、`ry`、`rz` 用于控制搜索中心与范围。  
- 语句在异步线程执行，不会阻塞主线程。  
- 仅对 `LivingEntity` 生效（非生物实体会自动跳过）。

---

## 🧩 CloudPick API 对应调用

服务端执行逻辑中调用：

```kotlin
ModelAPI.setEntityModel(living.uniqueId, modelId)
```

客户端会自动更新对应实体的 CloudPick 模型显示。

---

## 📚 对比 ArcartX

| 功能 | ArcartX | CloudPick |
| --- | --- | --- |
| 底层接口 | `ArcartXAPI` | `yslelf.cloudpick.api.ModelAPI` |
| 目标类型 | Player / Entity | LivingEntity |
| 用途 | 模型动画、骨骼动作 | 模型外观、展示绑定 |
| 是否带动画 | ✅ 可播放动画 | ❌ 仅替换模型 |

---

> 更新: 2025-11-01 01:58:30  
> 原文: <https://www.yuque.com/yuazer/blow95/cloudpick-model>
