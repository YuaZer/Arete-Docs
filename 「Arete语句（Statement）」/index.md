# 📚 Arete 语句（Statement）参考

> [🏠 返回首页](/) | [📖 查看教程](/如何创建一个简单的技能.md)

---

## 💡 什么是「语句」？

Arete 的技能脚本由一条条**语句（Statement）**组成。解析器将脚本（DSL）转换成语法树，再由语句工厂创建具体的执行对象，并按顺序或并行方式运行。

### 📝 常见写法示例

```yaml
message { text = "准备释放技能" }
delay { ticks = 20 } {
  message { text = "1 秒后触发" }
}
```

---

## 🔧 语句的结构

语句由以下三个部分组成：

| 组成部分 | 说明 | 示例 |
|---------|------|------|
| **名字** | 语句标识符 | `message`、`raycast`、`move-accel` |
| **参数块** | 花括号内的 `key = value` 列表 | `{ text = "Hello" }` |
| **子块** | 可选的嵌套子语句 | `delay { ... } { 子语句 }` |

### 参数值类型
- **裸字面量**：直接书写，如 `count = 5`
- **引号字符串**：`text = "Hello World"`
- **变量调用**：通过 `var` 语句定义的变量，使用 `${变量名}` 访问

---

## ⚙️ 执行模型

Arete 支持多种执行模式：

### 🔄 顺序执行
默认模式，一条语句执行完毕后再执行下一条。

```yaml
seq {
  message { text = "第一步" }
  message { text = "第二步" }
}
```

### ⚡ 并行执行
同时启动多个子块，全部完成后继续后续流程。

```yaml
parallel {
  particle { type = "FLAME" }
  sound { type = "ENTITY_BLAZE_SHOOT" }
}
```

### ⏳ 延时执行
延迟指定 tick 数后执行子块（1 tick = 50ms = 0.05秒）。

```yaml
delay { ticks = 20 } {
  message { text = "1秒后触发" }
}
```

### 🔀 条件执行
基于表达式决定执行哪个分支。

```yaml
if { cond = "vars.hp > 50" } {
  message { text = "血量充足" }
}
```

---

## 📋 语句分类索引

### 🔧 基础控制语句
用于变量管理、条件判断和流程控制。

- [🧰 var - 定义与覆盖变量](「Arete语句（Statement）」/var—定义_覆盖上下文变量.md)
- [⚖️ if - 条件判断](「Arete语句（Statement）」/⚖️if—条件判断语句.md)
- [🧠 ifchain - 多分支条件](「Arete语句（Statement）」/ifchain—多分支条件执行_case…else链.md)
- [🔁 for - 循环语句](「Arete语句（Statement）」/for—计次数_区间循环_临时变量.md)
- [🎲 random - 随机执行](「Arete语句（Statement）」/random—按权重随机执行一个子块.md)
- [⏳ delay - 延迟执行](「Arete语句（Statement）」/⏳delay—延迟执行后续语句.md)

### 💬 消息与界面
向玩家发送各种形式的消息提示。

- [🗨️ message - 发送消息](️「Arete语句（Statement）」/message—发送消息语句.md)
- [📢 nearby-message - 范围消息](「Arete语句（Statement）」/nearby-message—向附近玩家发送消息.md)
- [🪶 title - 标题与副标题](「Arete语句（Statement）」/title—显示标题与副标题.md)

### ✨ 视觉特效
创建粒子特效和音效，提升技能视觉表现。

- [🌌 particle - 粒子特效](「Arete语句（Statement）」/particle—粒子特效语句.md)
- [🌈 bedrockparticle - 基岩粒子](「Arete语句（Statement）」/bedrockparticle—播放基岩粒子特效.md)
- [🔊 sound - 音效播放](「Arete语句（Statement）」/sound—播放音效.md)

### 🎯 实体操作
选择、伤害、控制游戏实体。

- [🎯 target-entity - 选取实体](target-entity—范围选取实体_写入上下文变量.md)
- [🗡️ damage - 造成伤害](️damage—造成伤害.md)
- [🧪 effect - 药水效果](effect—施加药水效果.md)
- [🍞 hunger-cost - 扣除饱食度](hunger-cost—扣除饱食度.md)
- [🎯 projectile - 发射投掷物](projectile—投掷物.md)
- [💨 velocity - 设置速度](velocity—设置实体速度_冲刺击退.md)
- [💥 knockback - 击退目标](knockback—按方向击退目标_震退敌人.md)
- [🗑 despawn - 移除实体](despawn—移除实体_清理载体.md)
- [🍞 hunger-cost - 扣除饱食度](「Arete语句（Statement）」/hunger-cost—扣除饱食度.md)

### 🚀 运动与定位
实现复杂的位移、轨迹和射线检测。

- [🎯 raycast - 射线检测](「Arete语句（Statement）」/raycast—获取视线命中点_命中实体指针.md)
- [🧭 move - 平滑移动](「Arete语句（Statement）」/move—平滑移动实体_插值位移（含缓动）.md)
- [🚀 move-accel - 加速推进](「Arete语句（Statement）」/move-accel—加速度推进_追踪位移（限速可控）.md)
- [🌀 nurbs - NURBS曲线生成](「Arete语句（Statement）」/nurbs—三维曲线采样语句（NURBS曲线生成）.md)

### 🔨 高级功能
系统集成和高级控制功能。

- [🛡️ armorstand - 盔甲架操作](「Arete语句（Statement）」/️armorstand—生成与驱动隐形盔甲架（可穿物、可运动）.md)
- [🧾 command - 执行指令](「Arete语句（Statement）」/command—控制台执行指令_支持占位符.md)
- [🐉 mm-cast - MythicMobs集成](「Arete语句（Statement）」/mm-cast—调用MythicMobs技能_触发外部连锁.md)

### 🎨 ArcartX集成
与 ArcartX 模型系统的集成功能。

- [🦋 arcartx-model - 实体模型](「Arete语句（Statement）」/arcartx-model—设置ArcartX实体模型.md)
- [🐺 arcartx-anim - 动画播放](「Arete语句（Statement）」/arcartx-anim—播放ArcartX动画.md)

---

## 💡 快速提示

?> **新手建议**：从基础控制语句开始学习，掌握 `var`、`if`、`delay` 等基础语句后，再学习特效和实体操作。

!> **注意**：所有语句都支持变量插值，使用 `${变量名}` 语法可以在参数中引用变量值。

---

## 🔗 相关链接

- [📖 如何创建一个简单的技能](如何创建一个简单的技能.md)
- [🔧 创建Kether模板](如何创建一个简单的Kether模板/index.md)
- [👨‍💻 开发文档](附属开发以及对应API/index.md)

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px; border-top: 1px solid #eee; margin-top: 50px;">
  <p>📝 更新时间: 2025-10-17 09:49:53</p>
  <p>📚 共 27 个语句 | 💬 QQ交流群：923280954</p>
</div>