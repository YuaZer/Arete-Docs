<!-- 侧边栏导航 -->

* [🏠 首页](/)
* [📖 概述](概述.md)
* [📁 文件说明](文件说明.md)
* [⌨️ 命令列表](命令列表.md)
* [🔧 变量说明](变量.md)

---

* **快速入门**
  * [🎯 创建简单技能](如何创建一个简单的技能.md)
  * [📦 创建技能模板](如何创建一个简单的技能模板/index.md)
    * [调用技能模板](如何创建一个简单的技能模板/如何在技能配置中调用该模板.md)
  * [🔧 创建Kether模板](如何创建一个简单的Kether模板/index.md)
    * [调用Kether模板](如何创建一个简单的Kether模板/如何在技能配置中调用Kether模板.md)

---

* **📚 Arete语句参考**
  * [语句总览](「Arete语句（Statement）」/index.md)
  
  * **基础语句**
    * [🧰 var - 变量定义](「Arete语句（Statement）」/var—定义_覆盖上下文变量.md)
    * [⚖️ if - 条件判断](「Arete语句（Statement）」/⚖️if—条件判断语句.md)
    * [🧠 ifchain - 多分支条件](「Arete语句（Statement）」/ifchain—多分支条件执行_case…else链.md)
    * [🔁 for - 循环语句](「Arete语句（Statement）」/for—计次数_区间循环_临时变量.md)
    * [🎲 random - 随机执行](「Arete语句（Statement）」/random—按权重随机执行一个子块.md)
    * [⏳ delay - 延迟执行](「Arete语句（Statement）」/⏳delay—延迟执行后续语句.md)
  
  * **消息与界面**
    * [🗨️ message - 发送消息](「Arete语句（Statement）」/️message—发送消息语句.md)
    * [📢 nearby-message - 范围消息](「Arete语句（Statement）」/nearby-message—向附近玩家发送消息.md)
    * [🪶 title - 标题显示](「Arete语句（Statement）」/title—显示标题与副标题.md)
  
  * **视觉特效**
    * [🌌 particle - 粒子特效](「Arete语句（Statement）」/particle—粒子特效语句.md)
    * [🌈 bedrockparticle - 基岩粒子](「Arete语句（Statement）」/bedrockparticle—播放基岩粒子特效.md)
    * [🔊 sound - 音效播放](「Arete语句（Statement）」/sound—播放音效.md)
  
  * **实体操作**
    * [🎯 target-entity - 选取实体](「Arete语句（Statement）」/target-entity—范围选取实体_写入上下文变量.md)
    * [🗡️ damage - 造成伤害](「Arete语句（Statement）」/️damage—造成伤害.md)
    * [❤️ heal - 恢复生命](「Arete语句（Statement）」/heal—恢复生命.md)
    * [🧪 effect - 药水效果](「Arete语句（Statement）」/effect—施加药水效果.md)
    * [🎯 projectile - 发射投掷物](「Arete语句（Statement）」/projectile—投掷物.md)
    * [🍞 hunger-cost - 扣除饱食度](「Arete语句（Statement）」/hunger-cost—扣除饱食度.md)
    * [💨 velocity - 设置速度](「Arete语句（Statement）」/velocity—设置实体速度_冲刺击退.md)
    * [💥 knockback - 击退目标](「Arete语句（Statement）」/knockback—按方向击退目标_震退敌人.md)
    * [🗑 despawn - 移除实体](「Arete语句（Statement）」/despawn—移除实体_清理载体.md)
    * [📕 playerdata - 玩家数据持久化](「Arete语句（Statement）」/playerdata—玩家数据持久化.md)
    * [🧱 display-block — 创建虚拟方块展示](「Arete语句（Statement）」/display-block—创建虚拟方块展示.md)
    * [📦 display-item - 创建虚拟物品展示](「Arete语句（Statement）」/display-item—创建虚拟物品展示.md)
    * [🧊 display-block-control — 控制虚拟方块展示](「Arete语句（Statement）」/display-block-control—控制虚拟方块展示.md)
    * [📦 display-item-control - 控制虚拟物品展示](「Arete语句（Statement）」/display-item-control—控制虚拟物品展示.md)
    * [🖊 display-text - 文字展示/控制](「Arete语句（Statement）」/display-text-文字展示.md)

  
  * **运动与定位**
    * [🎯 raycast - 射线检测](「Arete语句（Statement）」/raycast—获取视线命中点_命中实体指针.md)
    * [🧭 move - 平滑移动](「Arete语句（Statement）」/move—平滑移动实体_插值位移（含缓动）.md)
    * [🚀 move-accel - 加速推进](「Arete语句（Statement）」/move-accel—加速度推进_追踪位移（限速可控）.md)
    * [🌀 nurbs - NURBS曲线](「Arete语句（Statement）」/nurbs—三维曲线采样语句（NURBS曲线生成）.md)
  
  * **高级功能**
    * [🛡️ armorstand - 盔甲架](「Arete语句（Statement）」/️armorstand—生成与驱动隐形盔甲架（可穿物、可运动）.md)
    * [🖊 cooldown - 设置技能冷却](「Arete语句（Statement）」/cooldown—技能冷却.md)
    * [🧾 command - 执行指令](「Arete语句（Statement）」/command—控制台执行指令_支持占位符.md)
    * [🐉 mm-cast - MythicMobs技能](「Arete语句（Statement）」/mm-cast—调用MythicMobs技能_触发外部连锁.md)
    * [💻 kether - 执行Kether脚本](「Arete语句（Statement）」/kether—执行Kether脚本语句.md)
    * [💻 js - 执行js脚本](「Arete语句（Statement）」/js-调用js文件执行操作.md)
  
  * **ArcartX集成**
    * [🦋 arcartx-model - 实体模型](「Arete语句（Statement）」/arcartx-model—设置ArcartX实体模型.md)
    * [🐺 arcartx-anim - 播放动画](「Arete语句（Statement）」/arcartx-anim—播放ArcartX动画.md)
    * [✉️ arcartx-sendpacket — 向 ArcartX 客户端发送自定义数据包](「Arete语句（Statement）」/arcartx-sendpacket%20—%20向ArcartX客户端发送自定义数据包.md)
  * **CloudPick集成**
    * [🧩 cloudpick-model — 设置 CloudPick 实体模型](「Arete语句（Statement）」/cloudpick-model—设置CloudPick实体模型.md)
    * [🧩 cloudpick-anim — 播放 CloudPick 动画](「Arete语句（Statement）」/cloudpick-anim—播放CloudPick动画.md)
    * [✉️ cloudpick-sendpacket — 向 CloudPick 玩家发送自定义数据包](「Arete语句（Statement）」/cloudpick-sendpacket—向CloudPick客户端发送自定义数据包.md)
---

* **👨‍💻 开发文档**
  * [开发总览](附属开发以及对应API/index.md)
  * [🧩 AreteAPI - 核心接口](附属开发以及对应API/AreteAPI—技能系统核心接口文档.md)
  * [🧩 语句注册教程](附属开发以及对应API/Arete语句注册教程（StatementRegistryGuide）.md)

---

* [📝 更新记录](更新记录.md)

---

<div style="text-align: center; padding: 20px 0; color: #999; font-size: 12px;">
  <p>Arete - 声明式脚本驱动的技能引擎</p>
  <p>QQ交流群：923280954</p>
</div>

