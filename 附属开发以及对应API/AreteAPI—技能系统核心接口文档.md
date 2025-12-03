# AreteAPI —— 技能系统核心接口文档

本页汇总核心 API 的调用方式，重点覆盖 `AreteAPI` 与 `AreteJavaAPI`，并补充语句注册、脚本执行与冷却等常用场景的示例。示例均以 Kotlin/Java 插件开发为背景。核心类路径：`io.github.yuazer.arete.api.AreteAPI`、`io.github.yuazer.arete.api.AreteJavaAPI`。

## AreteAPI 基础能力

### 初始化与关闭
- **初始化**：在插件启动时调用 `AreteAPI.init(plugin, registry)` 完成插件实例与语句注册表的绑定，后续所有 API 都依赖此步骤。
- **关闭**：在 `onDisable` 中调用 `AreteAPI.shutdown()`，会取消所有在运行的技能协程并清空编译缓存。
- **调试开关**：`AreteAPI.debugMode` 为 `true` 时，注册输出会包含耗时信息，适合排查性能。

### 语句注册
- Kotlin 语句直接用 `AreteAPI.register(name, factory)`；`factory` 需返回实现了 `Statement` 接口的对象。
- Java 侧建议使用后文的 `AreteJavaAPI` 以免手写协程。

### 脚本编译与执行
- **编译字符串**：`AreteAPI.compile(script)` 将多行脚本解析为 `Statement` 对象，可复用或手动执行。
- **协程执行**：`AreteAPI.run(script, ctx)` 在现有协程中执行脚本；`normalRun` 会自动在内部作用域发起执行，无需自行管理协程。
- **YML 入口**：提供对 `FileConfiguration`、`ConfigurationSection` 以及 TabooLib 配置的 `run` 重载，可直接按路径执行配置中的脚本段落，支持可选缓存键。
- **带缓存执行**：`runCached(rawScript, ctx, cacheKey)` 会先规范化脚本并复用编译缓存，适合高频调用。
- **缓存管理**：`normalizeMultiline`、`invalidate`、`clearCache`、`compileCached` 便于自行控制或预热脚本缓存，其中 `compileCached` 会显式写入 `ScriptRunner` 的内部缓存并返回 `Statement` 供持有。

### 技能施放入口
- **玩家施放**：`AreteAPI.cast(player, skillIdOrName, vararg args)` 会以玩家为上下文异步施放技能，`args` 自动写入 `ctx.vars["0".."n"]` 供脚本引用。
- **代理玩家**：`cast(ProxyPlayer, …)` 会尝试转换为 `Player` 后沿用相同逻辑。
- **任意实体**：`cast(Entity, …)` 允许非玩家实体触发技能；若不是 `Player`，不会触发事件、冷却或前置 Kether 判定，只执行技能主体。
- **核心流程**：当施放者为玩家时，`doCast` 会：
    1. 触发 `AreteSkillPreCastEvent`，可被取消；
    2. 运行内联和配置化的 Kether 前置检查；
    3. 判断冷却，发送冷却提示；
    4. 通过检查后调用 `SkillManager.runCast` 并设置冷却；
    5. 检查失败时触发 `AreteSkillCheckFailEvent` 并发送提示。

### 冷却控制
提供独立的冷却控制方法，便于脚本外部手动干预：`setCooldown`、`tryStartCooldown`、`addCooldown`、`reduceCooldown`、`clearCooldown`，均以玩家与技能标识为键。

## AreteJavaAPI：面向 Java 的语句注册

`AreteJavaAPI` 让 Java 插件无需处理 Kotlin 的 `suspend`/协程即可注册语句。

### 核心接口
- **JavaStatement**：同步执行接口，签名为 `void execute(ExecutionContext ctx)`，在 Arete 协程环境中同步调用。
- **JavaFactory**：根据参数创建 `JavaStatement` 的工厂，传入的 `JavaArgs` 提供 `get`、`getOrDefault`、`asMap` 便捷方法。

### 注册示例（Java）
```java
// MyMessageStatement.java
public class MyMessageStatement implements AreteJavaAPI.JavaStatement {
    private final String text;
    public MyMessageStatement(String text) { this.text = text; }
    @Override
    public void execute(ExecutionContext ctx) {
        ctx.getSource(Player.class).sendMessage(text);
    }
}

// 注册           语句名👇
AreteJavaAPI.register("message", args -> {
                    //          参数名👇参数默认值👇
    String text = args.getOrDefault("text", "Hello");
    return new MyMessageStatement(text);
});
```
`register` 会把 Java 工厂自动适配为内部的 `Statement.Factory` 并交由 `AreteAPI.register` 完成注册，无需关心协程或包装逻辑。

## 常见用法速查

### 触发技能（Kotlin）
```kotlin
// 在命令或监听器中触发技能
AreteAPI.cast(player, "fireball", "targetX", "targetY")
```

### 运行配置中的脚本
```kotlin
// 假设 config.yml 内有 scripts.cast 节点
AreteAPI.run(config, "scripts.cast", ctx, cacheKey = "script:cast")
```

### 预热并复用脚本
```kotlin
val stmt = AreteAPI.compileCached(rawScript, "script:heavy")
stmt.execute(ctx) // 手动复用已编译语句
```

### 手动操作冷却
```kotlin
if (AreteAPI.tryStartCooldown(player, "blink", ticks = 40L)) {
    // 通过返回值判断是否成功设置冷却
}
```

以上内容覆盖插件开发中最常用的接口与流程，可直接嵌入到开发文档或 Readme 中，作为技能系统的快速参考。