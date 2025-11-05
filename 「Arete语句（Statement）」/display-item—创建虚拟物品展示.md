# 🎁 display-item — 创建虚拟物品展示

> [🏠 首页](/) | [📚 语句总览](index.md) | **展示实体** | [← 🧊 display-block-control](display-block-control.md) | [→ 🪄 display-item-control](display-item-control.md)

---

## 📖 简介

`display-item` 会在指定位置生成一个 **ItemDisplay 虚拟物品展示实体**，
并将实体引用保存到 `ctx.vars[var]`。常用于悬浮物品、告示、技能提示道具等视觉效果。

你可以结合 `display-item-control` 调整展示姿态或制作动画。

---

## ⚙️ 参数说明

| 参数名 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `var` | String | _(必填)_ | 保存实体引用的变量名。 |
| `at` | String | `"@self"` | 生成位置解析表达式。 |
| `item` | String | _(必填)_ | 物品类型，对应 Bukkit `Material` 枚举。 |
| `amount` | Int | `1` | 物品堆叠数量，最小为 1。 |
| `offset` | String | `"0,0,0"` | 相对平移偏移量 `x,y,z`。 |
| `scale` | String | `"1,1,1"` | 缩放系数，可写 `sx,sy,sz` 或单值。 |
| `rotation` | String | `"0,0,0"` | 欧拉角 `yaw,pitch,roll`（度）。 |
| `viewRange` | Float | `32` | 可视距离。 |
| `mode` | String | `"ground"` | ItemDisplay 渲染模式：`ground` / `fixed` / `third_left` / `third_right` / `first_left` / `first_right` / `gui` / `head`。 |

---

## 🧠 参数格式示例

### ✅ 悬浮物品展示
```yaml
display-item {
  var = "reward"
  item = "DIAMOND"
  offset = "0,1.5,0"
  rotation = "0,0,0"
}
```

### ✅ GUI 模式展示
```yaml
display-item {
  var = "panelIcon"
  at = "${player}"
  item = "NETHER_STAR"
  mode = "gui"
  scale = "1.2"
}
```

### ✅ 与变量协作
```yaml
var { dropItem = "lootDisplay" }
display-item {
  var = "${dropItem}"
  at = "${target}"
  item = "GOLDEN_SWORD"
  amount = "1"
  rotation = "0,45,0"
}
```

---

## 💡 使用建议

- **预设道具模型**：可提前通过 `item` + 物品元数据（例如 Kether 语句修改 NBT）打造独特展示。
- **控制语句联动**：结合 [`display-item-control`](display-item-control.md) 实现旋转、缩放或动画效果。
- **配合 `var`**：可先用 `var` 语句声明变量名，再动态生成展示。

---

## ⚠️ 注意事项

- `item` 字段需是合法的物品 ID，大小写不敏感。
- 缩放或旋转过大可能造成视觉穿模，请根据场景调节。
- 展示实体不会自动清除，必要时请手动处理或定时回收。

---

## 🔗 相关语句

- [`display-block`](display-block—创建虚拟方块展示.md) — 创建 BlockDisplay
- [`display-block-control`](display-block-control—控制虚拟方块展示.md) — 控制 BlockDisplay
- [`display-item-control`](display-item-control—控制虚拟物品展示.md) — 控制 ItemDisplay
