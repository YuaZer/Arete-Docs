# 📝 display-text & display-text-control 使用教程

本教程介绍 Arete 中的两个虚拟文本展示语句：

- `display-text` — 创建一个 `TextDisplay` 虚拟文字实体，并写入 `ctx.vars[var]`
- `display-text-control` — 对已创建的 `TextDisplay` 进行文本/样式/动画控制

> ⚠ 这两个语句都只创建/操作 **显示实体**，不会更改真实方块或原版标题。

---

## 一、display-text —— 创建虚拟文字

### 1. 功能简介

`display-text` 用于在世界中创建一个虚拟文字实体（`TextDisplay`），可以理解为“悬浮字幕”。  
创建后会把这个实体对象保存在 `ctx.vars[var]` 中，供后续语句（如 `display-text-control`）引用和控制。

### 2. 基本语法

```plain
display-text {
    var  = "title1"    # 必填：保存变量名
    at   = "@self"     # 可选：位置表达式
    text = "&a欢迎来到 &bArete &f系统"  # 可选：显示文本（支持 & 颜色）

    # 变换参数（全可选）
    offset   = "0,2,0"     # 平移 (x,y,z)
    scale    = "1.2"       # 统一缩放
    rotation = "0,0,0"     # 欧拉角 (yaw,pitch,roll)

    center    = "true"     # 是否对齐方块中心
    viewRange = "32"       # 可视距离
    billboard = "fixed"    # fixed|center|vertical|horizontal

    # 文本样式（可选）
    lineWidth        = "200"
    shadowed         = "true"
    seeThrough       = "false"
    textOpacity      = "255"
    background       = "#000000"
    defaultBackground= "false"
    alignment        = "center"   # left|center|right
}
```

### 3. 参数详解

#### 3.1 必填参数

| 参数名 | 类型 | 说明 |
|-------|------|------|
| `var` | String | 必填。创建的 `TextDisplay` 会被保存到 `ctx.vars[var]` 中，例如 `ctx.vars["title1"]`。 |

#### 3.2 位置与文字

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `at`   | String | `@self` | 位置表达式，交给 `ExecutionContext.resolveLocation` 解析，如 `@self` / `@target` / `loc(x,y,z)` 等（取决于你自己的实现）。 |
| `text` | String | `""` | 显示文本，支持 `&` 颜色符号，会自动转为 `§`。 |

#### 3.3 变换（Transform）

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `offset` | String | `"0,0,0"` | 本地平移 `"x,y,z"`。例如 `"0,2,0"` 表示在基准点上方 2 格。 |
| `scale`  | String | `"1,1,1"` | 可写 `"sx,sy,sz"` 或 `"s"`。例如 `"1.2"` 会当成 `1.2,1.2,1.2`。 |
| `rotation` | String | `"0,0,0"` | 欧拉角 `"yaw,pitch,roll"`（单位：度）。yaw 绕 Y 轴，pitch 绕 X 轴，roll 绕 Z 轴。 |
| `center` | Boolean | `true` | 是否把实体生成在方块中心（自动 +0.5,+0.5,+0.5）。 |
| `viewRange` | Float | `32` | 最大可视距离。 |
| `billboard` | String | `"fixed"` | 文本朝向模式：`fixed|center|vertical|horizontal`。 |

#### 3.4 文本样式

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `lineWidth` | Int | `200` | 文本自动换行宽度（像素）。`0` 表示不限制。 |
| `shadowed` | Boolean | `false` | 是否启用文字阴影。 |
| `seeThrough` | Boolean | `false` | 是否穿墙可见。 |
| `textOpacity` | Int | `255` | 文本透明度（0~255，-1 使用默认）。 |
| `background` | String | _(不设置)_ | 文本背景颜色：`"#RRGGBB"` / `"RRGGBB"` 或 `"r,g,b"` 形式。 |
| `defaultBackground` | Boolean | _(不设置)_ | 是否使用默认背景风格，如果设置会通过 `applyTextStyle(... useDefaultBackground=...)` 应用。 |
| `alignment` | String | `"center"` | 对齐方式：`left` / `center` / `right`。 |

### 4. 示例：在玩家头顶生成标题

```yml
# 在玩家头顶 2 格生成一个绿色标题
display-text {
  var = "welcomeTitle"
  at = "@self"
  text = "&a欢迎来到 &bArete &f系统"
  offset = "0,2,0"
  scale = "1.2"
  lineWidth = "180"
  shadowed = "true"
  alignment = "center"
}
```

此时，在后续语句中就可以通过：

```plain
ctx.vars["welcomeTitle"] as TextDisplay
```

或 DSL 语句里的 `target = "welcomeTitle"` 来引用这个实体。

---

## 二、display-text-control —— 控制虚拟文字

### 1. 功能简介

`display-text-control` 用于控制已经通过 `display-text` 创建好的 `TextDisplay`，支持：

- 修改文本内容（设置 / 清空）
- 更新样式（行宽、阴影、背景、透明度、对齐…）
- 平移/缩放/旋转（与 `display-block-control` 一致）
- 动画：
  - 线性移动（`anim-move`）
  - 自转（`anim-spin`）
  - 透明度渐变（`anim-opacity-*`）
  - 打字机效果（`anim-typewriter-*`）

### 2. 基本语法

```plain
display-text-control {
    target = "welcomeTitle"  # 必填：要控制的 TextDisplay 变量名

    # 文本
    set-text   = "&e新的标题文本"
    clear-text = "false"

    # 样式
    lineWidth        = "200"
    shadowed         = "true"
    seeThrough       = "false"
    textOpacity      = "255"
    background       = "#000000"
    defaultBackground= "false"
    alignment        = "center"

    # 变换
    set-translation = "0,2,0"
    add-translation = "0,0.5,0"
    set-scale       = "1.2,1.2,1.2"
    uniform-scale   = "1.1"
    set-rotation    = "0,0,0"
    add-rotation    = "5,0,0"

    # 动画：移动
    anim-move          = "0,3,0"
    anim-move-duration = "20"

    # 动画：自转
    anim-spin          = "2,0,0"
    anim-spin-duration = "100"

    # 动画：透明度
    anim-opacity-to       = "0"
    anim-opacity-from     = "255"
    anim-opacity-duration = "40"

    # 动画：打字机效果
    anim-typewriter-text        = "&b逐字显示的标题文本~"
    anim-typewriter-chars       = "1"
    anim-typewriter-stepTicks   = "2"
    anim-typewriter-delay       = "10"
    anim-typewriter-keep-prefix = "false"
}
```

### 3. 参数详解

#### 3.1 必填参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `target` | String | 必填。要控制的 `TextDisplay` 对象变量名，对应 `ctx.vars[target]`。若不是 `TextDisplay` 会直接 return。 |

#### 3.2 文本相关

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `set-text` | String | _(不设置)_ | 将文本设置为指定内容，支持 `&` 颜色（内部会转为 `§`）。 |
| `clear-text` | Boolean | `false` | 若为 `true`，先把文本清空为 `""`。如果同时配置 `set-text`，则先清空，后再设置。 |

#### 3.3 样式相关（走 `VirtualDisplayUtil.applyTextStyle`）

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `lineWidth` | Int | _(不改)_ | 自动换行宽度（像素）。 |
| `shadowed` | Boolean | _(不改)_ | 是否使用阴影。 |
| `seeThrough` | Boolean | _(不改)_ | 是否穿墙可见。 |
| `textOpacity` | Int | _(不改)_ | 透明度 0~255，-1 默认。 |
| `background` | String | _(不改)_ | 背景颜色，格式同 `display-text`。 |
| `defaultBackground` | Boolean | _(不改)_ | 是否使用默认背景样式。 |
| `alignment` | String | _(不改)_ | `left` / `center` / `right`。 |

#### 3.4 变换相关（与 display-block-control 一致）

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `set-translation` | String | `"x,y,z"`，绝对平移。 |
| `add-translation` | String | `"dx,dy,dz"`，在当前基础上平移。 |
| `set-scale` | String | `"sx,sy,sz"`，绝对缩放。 |
| `uniform-scale` | String | `"s"`，统一缩放。 |
| `set-rotation` | String | `"yaw,pitch,roll"`，绝对欧拉角。 |
| `add-rotation` | String | `"yaw,pitch,roll"`，在当前基础上追加欧拉角。 |

#### 3.5 动画：移动

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `anim-move` | String | _(不动画)_ | `"x,y,z"`，目标 translation。 |
| `anim-move-duration` | Long | _(不动画)_ | 持续 tick 数，必须 >0 才会触发动画。 |

内部调用：

```kotlin
VirtualDisplayUtil.animateMoveTo(plugin, disp, tx, ty, tz, durationTicks)
```

#### 3.6 动画：自转

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `anim-spin` | String | _(不动画)_ | `"yawPerTick,pitchPerTick,rollPerTick"`。 |
| `anim-spin-duration` | Long | `0` | 持续 tick 数，<=0 或缺失 => 无限旋转。 |

内部调用：

```kotlin
VirtualDisplayUtil.animateSpin(plugin, disp, yawPerTick, pitchPerTick, rollPerTick, durationTicks)
```

#### 3.7 动画：透明度渐变

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `anim-opacity-to` | Int | _(不动画)_ | 目标透明度 0~255。 |
| `anim-opacity-from` | Int | 当前值 | 可选，起始透明度，不填则读取当前 `disp.textOpacity`。 |
| `anim-opacity-duration` | Long | _(不动画)_ | 持续 tick 数，>0 才会执行动画。 |

内部调用：

```kotlin
VirtualDisplayUtil.animateTextOpacity(plugin, disp, toOpacity, durationTicks, fromOpacity)
```

#### 3.8 动画：打字机效果

| 参数名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `anim-typewriter-text` | String | _(不动画)_ | 完整文本，支持 `&` 颜色。 |
| `anim-typewriter-chars` | Int | `1` | 每一步新增的可见字符数量。 |
| `anim-typewriter-stepTicks` | Long | `1` | 每一步间隔 tick 数。 |
| `anim-typewriter-delay` | Long | `0` | 开始前的延迟 tick。 |
| `anim-typewriter-keep-prefix` | Boolean | `false` | 若为 `true`，保留现有 `disp.text` 作为前缀，在其后面追加打字机内容。 |

内部调用：

```kotlin
VirtualDisplayUtil.animateTypewriter(
    plugin = AreteAPI.plugin,
    display = disp,
    fullText = animTypewriterText,
    charsPerStep = chars,
    stepTicks = stepTicks,
    startDelayTicks = delay,
    keepExistingPrefix = keepPrefix
)
```

---

## 三、实战示例

### 示例 1：创建一个欢迎标题并逐字显示

```yml
# 1) 先创建一个 TextDisplay
display-text {
  var = "title1"
  at = "@self"
  text = ""                # 初始为空，后面打字机写入
  offset = "0,2,0"
  scale = "1.2"
  lineWidth = "180"
  shadowed = "true"
  alignment = "center"
}

# 2) 使用打字机效果逐字显示文字
display-text-control {
  target = "title1"

  anim-typewriter-text      = "&a欢迎来到 &bArete &f系统!"
  anim-typewriter-chars     = "1"   # 每次显示 1 字符
  anim-typewriter-stepTicks = "2"   # 每 2 tick 更新一次（约 0.1 秒）
  anim-typewriter-delay     = "10"  # 延迟 10 tick 再开始
}
```

### 示例 2：显示提示文字，缓慢上浮并淡出

```yml
# 创建提示文字
display-text {
  var = "hint1"
  at = "@self"
  text = "&e+10 经验值"
  offset = "0,1.5,0"
  scale = "1.0"
  shadowed = "true"
  textOpacity = "255"
}

# 控制：缓慢上浮并淡出
display-text-control {
  target = "hint1"

  # 位移动画：从 (0,1.5,0) 移动到 (0,3,0)
  anim-move          = "0,3,0"
  anim-move-duration = "40"

  # 透明度动画：从 255 减少到 0
  anim-opacity-to       = "0"
  anim-opacity-from     = "255"
  anim-opacity-duration = "40"
}
```

### 示例 3：在空中展示静态告示牌

```yml
display-text {
  var = "board1"
  at = "${pos}"  # 具体坐标
  text = "&6[ 信息公告 ]\n&7PVP 区域内禁止下线"
  offset = "0,0,0"
  scale = "1.0"
  rotation = "180,0,0"    # 朝向玩家（示例）
  lineWidth = "160"
  shadowed = "true"
  background = "#000000"
  defaultBackground = "false"
  alignment = "center"
}
```

---

## 四、注意事项

1. **线程与主线程**
   - 所有实体创建与动画调用都通过 `onMain { ... }` 和 `BukkitRunnable` 在主线程执行，脚本本身可以是 `suspend` 协程。
2. **变量生命周期**
   - `display-text` 创建的实体对象被保存到了 `ctx.vars[var]`，只在当前技能/脚本执行上下文内有效。
   - 若需要跨技能/跨脚本保存，请自行在外部用你的 API 或额外 Map 缓存引用。
3. **性能建议**
   - 不要无限创建而不销毁 TextDisplay，否则长期运行可能堆积实体。
   - 对于长期存在的“告示牌”类文本，可以手动统一管理、复用或定期清理。


