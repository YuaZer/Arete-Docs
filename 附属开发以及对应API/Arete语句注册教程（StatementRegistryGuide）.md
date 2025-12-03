# Arete 语句注册教程（Statement Registry Guide）

本页用最少步骤讲清楚「语句」的注册流程，帮助你把自定义逻辑接入 Arete。无需额外了解协程或底层细节。核心类路径示例：`io.github.yuazer.arete.core.StatementRegistry`、`io.github.yuazer.arete.api.AreteJavaAPI`。

## 核心概念（30 秒读完）
- **语句（Statement）**：可执行的最小脚本单元，实现 `Statement` 接口的 `execute` 方法即可。
- **语句工厂（Factory）**：根据参数创建语句实例，注册时传入。
- **注册器（StatementRegistry）**：保存所有语句，`AreteAPI.register(name, factory)` 会把语句放进去。

## Kotlin 开发者：3 步完成注册
1. **准备工厂**：实现 `Statement.Factory`，从 `args` 里取参并返回语句实例。
2. **编写语句**：在 `execute` 中写具体逻辑，可通过 `ctx.source` 拿到触发者，通过 `ctx.vars` 存取变量。
3. **调用注册**：在插件初始化时调用 `AreteAPI.register("语句名", factory)` 即可。

示例：给玩家发送一条自定义消息
```kotlin
// 注册自定义语句
AreteAPI.register("message") { args, _ ->
    val text = args.get("text") ?: "Hello, Arete!"
    object : Statement {
        override suspend fun execute(ctx: ExecutionContext) {
            ctx.source.sendMessage(text)
        }
    }
}
```
- 语句名是 `message`，脚本里写 `message text:"你好"` 即可调用。
- `args.get("text")` 读取脚本参数；未提供时使用默认值。

## Java 开发者：用 AreteJavaAPI 免去包装
1. **实现 JavaStatement**：只需同步接口 `void execute(ExecutionContext ctx)`。
2. **实现 JavaFactory**：在 `create` 里读取 `JavaArgs` 并返回语句对象。
3. **注册**：调用 `AreteJavaAPI.register("语句名", factory)`，内部会自动适配为 Kotlin 语句工厂。

示例：Java 版消息语句
```java
public class MessageStmt implements AreteJavaAPI.JavaStatement {
    private final String text;
    public MessageStmt(String text) { this.text = text; }
    @Override public void execute(ExecutionContext ctx) {
        Entity source = ctx.getSource();
        if (source instanceof Player) {
            Player player = (Player) source;
            player.sendMessage(text);
        }
    }
}

// 注册
AreteJavaAPI.register("message", args -> {
    String text = args.getOrDefault("text", "Hello, Arete!");
    return new MessageStmt(text);
});
```

## 常见问题速记
- **名字重复怎么办？** 后注册的同名语句会覆盖旧的，可用 `StatementRegistry.unregister(name)` 卸载。
- **从脚本里拿参数？** Kotlin 用 `args.get("key")`，Java 用 `JavaArgs.get/ getOrDefault/ asMap`。
- **执行时如何访问触发者？** 用 `ctx.source`（玩家或任意实体）；需要变量时操作 `ctx.vars`。

按照以上步骤就能把自定义语句接入 Arete，脚本里直接按语句名调用即可。