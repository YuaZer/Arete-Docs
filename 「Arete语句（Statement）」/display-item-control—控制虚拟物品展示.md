# 🪄 display-item-control — 控制虚拟物品展示

> [🏠 首页](/) | [📚 语句总览](index.md) | **展示实体** | [← 🎁 display-item](display-item.md)

---

## 📖 简介

`display-item-control` 与 [`display-block-control`](display-block-control.md) 用法一致，
专门用于操作 `display-item` 创建的 **ItemDisplay**。可以对展示物品进行平移、缩放、旋转，以及播放移动或自转动画。

---

## ⚙️ 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `target` | String | _(必填)_ | `ctx.vars` 中保存 ItemDisplay 的键名。 |
| `set-translation` | String | _(可选)_ | 绝对平移 `x,y,z`。 |
| `add-translation` | String | _(可选)_ | 相对平移 `dx,dy,dz`。 |
| `set-scale` | String | _(可选)_ | 缩放为 `sx,sy,sz`。 |
| `uniform-scale` | Float/String | _(可选)_ | 统一缩放为 `s`。 |
| `set-rotation` | String | _(可选)_ | 绝对欧拉角 `yaw,pitch,roll`（度）。 |
| `add-rotation` | String | _(可选)_ | 相对欧拉角 `yaw,pitch,roll`。 |
| `anim-move` | String | _(可选)_ | 动画移动到 `x,y,z`。 |
| `anim-move-duration` | Long | _(可选)_ | 移动动画持续 tick 数，需与 `anim-move` 配合。 |
| `anim-spin` | String | _(可选)_ | 每 tick 旋转量 `yawPer,pitchPer,rollPer`。 |
| `anim-spin-duration` | Long | `0` | 自转动画持续 tick 数，缺省/`<=0` 表示无限循环。 |

---

## 🧠 常见用法

### ✅ 让物品漂浮旋转
```yaml
display-item-control {
  target = "reward"
  add-translation = "0,0.1,0"
  anim-spin = "0,6,0"
}
```

### ✅ 抛射到目标位置
```yaml
display-item-control {
  target = "reward"
  anim-move = "${targetX},${targetY},${targetZ}"
  anim-move-duration = "30"
}
```

---

## 💡 使用建议

- 可与 `display-block-control` 混合使用，制作复合场景（例如物品围绕方块旋转）。
- 若需在动画结束后执行逻辑，可结合 `delay` 或 `kether` 监控状态。
- `target` 不存在时语句会直接结束，可在前一步通过 `if` 判断是否成功生成实体。

---

## ⚠️ 注意事项

- 长时间无限旋转会持续占用调度器，请在不需要时再次调用控制语句停止或移除实体。
- 平移/旋转数值为 `Float`，可使用小数实现细腻动画效果。
- 与 `display-item` 一样，展示实体不会自动清理，请自行处理生命周期。

---

## 🔗 相关语句

- [`display-item`](display-item—创建虚拟物品展示.md) — 创建 ItemDisplay
- [`display-block`](display-block—创建虚拟方块展示.md) — 创建 BlockDisplay
- [`display-block-control`](display-block-control—控制虚拟方块展示.md) — 控制 BlockDisplay