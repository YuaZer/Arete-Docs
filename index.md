# Arete 技能插件教程文档

<div style="text-align: center; padding: 20px 0;">
  <h2 style="color: #42b983;">🎴 Arete - The Art of Mastery</h2>
  <p style="font-size: 18px; color: #666;">声明式脚本驱动的 Minecraft 技能引擎</p>
  <p style="color: #999;">基于 TabooLib | 支持热加载 | 模板化管理</p>
</div>

---

## 📖 文档导航

### 🚀 快速开始

新手推荐按照以下顺序学习：

1. **[📋 概述](概述.md)** - 了解Arete插件的核心理念与功能特性
2. **[📁 文件说明](文件说明.md)** - 熟悉插件的文件结构与配置方式
3. **[⌨️ 命令列表](命令列表.md)** - 掌握插件的基本命令操作
4. **[🎯 如何创建一个简单的技能](如何创建一个简单的技能.md)** - 动手创建第一个技能

---

### 🎓 进阶教程

掌握更高级的技能配置技巧：

#### 技能模板系统
- **[📦 创建技能模板](如何创建一个简单的技能模板/index.md)**
  - [调用技能模板](如何创建一个简单的技能模板/如何在技能配置中调用该模板.md)

#### Kether脚本集成
- **[🔧 创建Kether模板](如何创建一个简单的Kether模板/index.md)**
  - [调用Kether模板](如何创建一个简单的Kether模板/如何在技能配置中调用Kether模板.md)

---

### 📚 Arete语句完整参考

> [📑 语句总览页面](「Arete语句（Statement）」/index.md)

<details>
<summary><b>🔧 基础控制语句（6个）</b></summary>

- [🧰 var - 定义与覆盖变量](「Arete语句（Statement）」/var—定义_覆盖上下文变量.md)
- [⚖️ if - 条件判断](「Arete语句（Statement）」/⚖️if—条件判断语句.md)
- [🧠 ifchain - 多分支条件](「Arete语句（Statement）」/ifchain—多分支条件执行_case…else链.md)
- [🔁 for - 循环语句](「Arete语句（Statement）」/for—计次数_区间循环_临时变量.md)
- [🎲 random - 随机执行](「Arete语句（Statement）」/random—按权重随机执行一个子块.md)
- [⏳ delay - 延迟执行](「Arete语句（Statement）」/⏳delay—延迟执行后续语句.md)

</details>

<details>
<summary><b>💬 消息与界面语句（3个）</b></summary>

- [🗨️ message - 发送消息](「Arete语句（Statement）」/️message—发送消息语句.md)
- [📢 nearby-message - 范围消息](「Arete语句（Statement）」/nearby-message—向附近玩家发送消息.md)
- [🪶 title - 标题与副标题](「Arete语句（Statement）」/title—显示标题与副标题.md)

</details>

<details>
<summary><b>✨ 视觉特效语句（3个）</b></summary>

- [🌌 particle - 粒子特效](「Arete语句（Statement）」/particle—粒子特效语句.md)
- [🌈 bedrockparticle - 基岩粒子](「Arete语句（Statement）」/bedrockparticle—播放基岩粒子特效.md)
- [🔊 sound - 音效播放](「Arete语句（Statement）」/sound—播放音效.md)

</details>

<details>
<summary><b>🎯 实体操作语句（8个）</b></summary>

- [🎯 target-entity - 选取实体](「Arete语句（Statement）」/target-entity—范围选取实体_写入上下文变量.md)
- [🗡️ damage - 造成伤害](「Arete语句（Statement）」/️damage—造成伤害.md)
- [🧪 effect - 药水效果](「Arete语句（Statement）」/effect—施加药水效果.md)
- [🎯 projectile - 发射投掷物](「Arete语句（Statement）」/projectile—投掷物.md)
- [💨 velocity - 设置速度](「Arete语句（Statement）」/velocity—设置实体速度_冲刺击退.md)
- [💥 knockback - 击退目标](「Arete语句（Statement）」/knockback—按方向击退目标_震退敌人.md)
- [🗑 despawn - 移除实体](「Arete语句（Statement）」/despawn—移除实体_清理载体.md)
- [📕 playerdata - 玩家数据持久化](「Arete语句（Statement）」/playerdata—玩家数据持久化.md)


</details>

<details>
<summary><b>🚀 运动与定位语句（4个）</b></summary>

- [🎯 raycast - 射线检测](「Arete语句（Statement）」/raycast—获取视线命中点_命中实体指针.md)
- [🧭 move - 平滑移动](「Arete语句（Statement）」/move—平滑移动实体_插值位移（含缓动）.md)
- [🚀 move-accel - 加速推进](「Arete语句（Statement）」/move-accel—加速度推进_追踪位移（限速可控）.md)
- [🌀 nurbs - NURBS曲线生成](「Arete语句（Statement）」/nurbs—三维曲线采样语句（NURBS曲线生成）.md)

</details>

<details>
<summary><b>🔨 高级功能语句（3个）</b></summary>

- [🛡️ armorstand - 盔甲架操作](「Arete语句（Statement）」/️armorstand—生成与驱动隐形盔甲架（可穿物、可运动）.md)
- [🧾 command - 执行指令](「Arete语句（Statement）」/command—控制台执行指令_支持占位符.md)
- [🐉 mm-cast - MythicMobs集成](「Arete语句（Statement）」/mm-cast—调用MythicMobs技能_触发外部连锁.md)

</details>

<details>
<summary><b>🔧 CloudPick集成（3个）</b></summary>

- [🐺 cloudpick-model - 实体模型](「Arete语句（Statement）」/cloudpick-model—设置CloudPick实体模型.md)
- [🐉 cloudpick-anim - 动画播放](「Arete语句（Statement）」/cloudpick-anim—播放CloudPick动画.md)
- [✉️ cloudpick-sendpacket — 发送自定义数据包](「Arete语句（Statement）」/cloudpick-sendpacket—向CloudPick客户端发送自定义数据包.md)
</details>

<details>
<summary><b>🎨 ArcartX集成（3个）</b></summary>

- [🦋 arcartx-model - 实体模型](「Arete语句（Statement）」/arcartx-model—设置ArcartX实体模型.md)
- [🐺 arcartx-anim - 动画播放](「Arete语句（Statement）」/arcartx-anim—播放ArcartX动画.md)
- [✉️ arcartx-sendpacket — 发送自定义数据包](「Arete语句（Statement）」/arcartx-sendpacket%20—%20向ArcartX客户端发送自定义数据包.md)
</details>


---

### 👨‍💻 开发者文档

为插件开发附属功能或集成到你的项目：

- **[🔌 开发文档总览](附属开发以及对应API/index.md)**
- **[🧩 AreteAPI - 核心接口文档](附属开发以及对应API/AreteAPI—技能系统核心接口文档.md)**
- **[📝 Arete语句注册教程](附属开发以及对应API/Arete语句注册教程（StatementRegistryGuide）.md)**

---

### 📋 其他资源

- **[📝 更新记录](更新记录.md)** - 查看插件的版本更新历史

---

## 💡 使用提示

?> **提示**：建议使用左侧侧边栏进行文档导航，使用顶部搜索框快速查找内容。

!> **注意**：本文档持续更新中，如有疑问请加入QQ交流群：**923280954**

---

<div style="text-align: center; padding: 30px 0; color: #999; border-top: 1px solid #eee; margin-top: 50px;">
  <p style="font-size: 14px;">© Arete - 由声明式脚本驱动的 Minecraft 技能引擎</p>
  <p style="font-size: 12px;">基于 TabooLib 开发 | 支持 PlaceholderAPI 与 Kether</p>
  <p style="font-size: 12px; margin-top: 10px;">
    <a href="#" style="color: #42b983; text-decoration: none;">📱 QQ交流群：923280954</a>
  </p>
</div>
