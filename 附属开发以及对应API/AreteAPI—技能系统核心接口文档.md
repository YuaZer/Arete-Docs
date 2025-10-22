# 🧩 AreteAPI — 技能系统核心接口文档



包路径：`io.github.yuazer.arete.api.AreteAPI`  
版本：1.0+（协程支持版）  
模块：Arete Skill Engine 核心 API  
作用：提供脚本注册、语句执行、脚本缓存、技能施法全流程管理



## 📘 一、API 概览
AreteAPI 是整个 Arete 技能系统的 **统一入口**。  
提供了以下核心功能：

| 模块 | 功能描述 |
| --- | --- |
| 🧱 **语句注册** | 向 `StatementRegistry`<br/> 注册自定义 DSL 语句 |
| ⚙️ **脚本编译执行** | 将 YAML / 文本脚本解析为可执行结构并运行 |
| 🧠 **脚本缓存** | 多次执行时自动复用编译结果 |
| 🪄 **技能触发** | 调用指定技能（含冷却、条件检查、执行上下文） |
| 🧰 **工具函数** | 提供 `normalizeMultiline`<br/>、`invalidate`<br/>、`clearCache`<br/> 等便捷方法 |
| 🚀 **协程控制** | 内置安全的受控作用域（`AreteAPI.scope`<br/>）用于异步执行 |


---

## 📦 二、基础信息
### 属性与生命周期
| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `plugin` | `Plugin` | 当前插件实例（由 `init()`<br/> 初始化时注入） |
| `debugMode` | `Boolean` | 是否启用调试模式（打印注册耗时） |
| `scope` | `CoroutineScope` | Arete 的主协程作用域（基于 `SupervisorJob + Dispatchers.Default`<br/>） |


### 生命周期方法
| 方法 | 说明 |
| --- | --- |
| `init(plugin, registry)` | 初始化 API。   必须在插件 `onEnable`<br/> 时调用，用于设置插件实例和自定义语句注册表。 |
| `shutdown()` | 插件关闭时调用。取消所有正在运行的技能任务并清空缓存。 |


---

## 🧱 三、语句注册接口
### 🔹 注册语句
```plain
AreteAPI.register("velocity", VelocityStatement.factory)
AreteAPI.register("move", MoveEntityStatement.factory)
```

内部调用 `StatementRegistry.register`，并打印注册耗时。  
支持重复注册覆盖，debug 模式下附带注册耗时统计。

### 📖 示例输出
```plain
[Arete] Registered velocity ✔ (0.17 ms)
[Arete] Registered move ✔ (0.09 ms)
```

---

## ⚙️ 四、脚本编译与执行
### 1️⃣ 编译字符串为语句
```plain
val statement: Statement = AreteAPI.compile("""
    message { text = "&aHello, world!" }
""".trimIndent())
```

### 2️⃣ 直接执行脚本
```plain
val ctx = ExecutionContext(source = player)
AreteAPI.run("""
    velocity { x = 0 y = 1 z = 0 }
    message { text = "&a你被弹飞啦！" }
""".trimIndent(), ctx)
```

该方法是 **挂起函数（suspend）**，应在协程环境中调用。  
若在监听器/命令中使用，请通过 `AreteAPI.scope.launch { ... }` 包裹。

---

## 📜 五、配置文件脚本执行（ScriptRunner 入口）
你可以直接执行 YAML 或 `taboolib` 配置节点内的技能脚本段落。

### 1️⃣ YAML 配置执行
```plain
exampleSkill:
  cast:
    - message { text = "&a技能释放成功!" }
    - velocity { y = 0.8 }
```

```plain
AreteAPI.run(config, "exampleSkill.cast", ctx)
```

### 2️⃣ TabooLib 配置执行
```plain
AreteAPI.run(tabooConfig, "skill.cast", ctx)
```

### 3️⃣ 原始脚本文本（自动缓存）
```plain
AreteAPI.runCached("""
    for { times = 3 } {
      message { text = "&b连击!" }
    }
""".trimIndent(), ctx, cacheKey = "tripleStrike")
```

---

## 🧠 六、缓存控制与工具函数
| 方法 | 说明 |
| --- | --- |
| `normalizeMultiline(raw: String)` | 规范化多行字符串，去除缩进、格式化换行 |
| `invalidate(cacheKey: String)` | 移除指定缓存脚本 |
| `clearCache()` | 清空所有脚本缓存 |
| `compileCached(script, cacheKey)` | 编译脚本并写入缓存，返回可复用的 `Statement` |


### 💡 示例
```plain
val st = AreteAPI.compileCached(myScript, "lightStrike")
AreteAPI.scope.launch { st.execute(ctx) }
```

---

## 🪄 七、技能施法接口
AreteAPI 内部集成了 **SkillManager + CooldownManager + CacheManager**，  
提供了完整的施法流程与条件校验。

---

### 🔹 cast(player, skillIdOrName)
```plain
AreteAPI.cast(player, "Fireball")
```

+ **功能**：非挂起函数（可直接在命令、监听器中调用）。
+ **实现**：内部使用 `scope.launch` 自动开启协程。
+ **逻辑**：
    1. 从 `SkillManager` 获取技能配置
    2. 执行内联 Kether 校验（`checkBeforeCast`）
    3. 执行配置 Kether 校验（`checkBeforeCastConfig`）
    4. 冷却检测（`CooldownManager`）
    5. 若通过，执行技能主脚本 `SkillManager.runCast(prof, ctx)`
    6. 设置技能冷却时间

### ⚙️ 参数
| 参数 | 类型 | 说明 |
| --- | --- | --- |
| `player` | `Player` | 施法者 |
| `skillIdOrName` | `String` | 技能标识（支持 id 或 name） |


---

### 🔹 onCast(player, skillIdOrName)
```plain
AreteAPI.scope.launch {
    AreteAPI.onCast(player, "ThunderStrike")
}
```

+ **挂起函数（suspend）**
+ 与 `cast()` 逻辑一致，只是不再开新协程，可用于内部逻辑或脚本触发。

### ⚠️ 兼容旧版方法
```plain
AreteAPI.onCastSuspend(player, "Fireball")
```

已弃用，请改用 `AreteAPI.cast()`。

---

### 🧩 校验逻辑说明
| 校验阶段 | 来源 | 返回条件 |
| --- | --- | --- |
| `checkBeforeCast` | 技能配置内联脚本（字符串） | 若为空则视为通过 |
| `checkBeforeCastConfig` | 技能配置引用的外部脚本名列表 | 为空集合视为通过 |
| `CooldownManager` | 技能冷却时间 | 在冷却中则提示 `skill-cooldown`<br/> 语言节点 |


---

### 🧾 语言配置占位符
| 语言键 | 默认文本 |
| --- | --- |
| `skill-cooldown` | `§c技能冷却中，还需 {time} 秒！` |
| `skill-check-fail` | `§c条件未满足，无法释放技能！` |


---

## 🧰 八、协程与线程模型
+ **执行线程**：Arete 的语句执行逻辑全部在 `Dispatchers.Default` 协程线程池运行；
+ **主线程操作**：语句内部需自行使用 `onMain {}`、`delayOneTick()` 等工具确保 Bukkit 安全；
+ **并发安全**：所有运行中技能共享一个 `SupervisorJob` 根作用域，可在 `AreteAPI.shutdown()` 时整体取消。

### 推荐用法
```plain
AreteAPI.scope.launch {
    AreteAPI.runCached(myScript, ctx, "sword-dash")
}
```

---

## 💾 九、内部管理器说明（只读参考）
| 管理器 | 作用 |
| --- | --- |
| `SkillManager` | 管理技能配置与执行 |
| `CooldownManager` | 控制技能冷却与剩余时间 |
| `CacheManager` | 管理缓存的 Kether 校验脚本 |
| `ScriptParser` | 将 DSL 文本解析为 `Statement`<br/> 对象 |
| `ScriptRunner` | 实际脚本执行调度器（包含缓存、编译、上下文执行） |
| `KetherUtils` | 辅助执行 kether 表达式脚本 |


---

## 🧩 十、综合示例
### 📄 示例：完整技能注册与执行流程
```plain
// 注册自定义语句
AreteAPI.register("shout") { args, _ ->
    object : Statement {
        override suspend fun execute(ctx: ExecutionContext) {
            val msg = args["text"] ?: "啊——！"
            ctx.playerOrNull()?.sendMessage("§e${ctx.source.name} 喊叫: $msg")
        }
    }
}

// 编译技能脚本
val script = """
seq {
  message { text = "&a你释放了烈焰爆发!" }
  velocity { y = 0.8 }
}
""".trimIndent()

// 执行
val ctx = ExecutionContext(source = player)
AreteAPI.scope.launch {
    AreteAPI.runCached(script, ctx, "flameBurst")
}
```

---

## 🔧 十一、常见问题 (FAQ)
| 问题 | 原因与解决 |
| --- | --- |
| 注册不生效 | 确认是否调用了 `AreteAPI.init(plugin, registry)`<br/> 并在主插件初始化时执行。 |
| “Missing key …” 报错 | 工厂参数缺少必填项。请在 factory 内添加 `error("... is required")`<br/>。 |
| 语句未执行 | 某些语句内部未切回主线程；检查是否调用 `onMain {}`<br/>。 |
| 技能无反应 | 可能因 `checkBeforeCast`<br/> 返回 false 或 `CooldownManager`<br/> 拒绝。查看语言提示。 |
| 缓存未更新 | 需手动调用 `AreteAPI.invalidate(cacheKey)`<br/> 或 `clearCache()`<br/>。 |




> 更新: 2025-10-17 15:34:00  
> 原文: <https://www.yuque.com/yuazer/blow95/xza6ghymlfykadiw>