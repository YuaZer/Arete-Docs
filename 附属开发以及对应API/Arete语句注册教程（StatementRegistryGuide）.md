# 🧩 Arete 语句注册教程（Statement Registry Guide）

下面这份教程基于你当前的接口与注册器实现：

+ `Statement`/`Args` 接口
+ `Statement.Factory` 工厂模式
+ `StatementRegistry` 注册中心（支持别名、调试耗时输出、Box 会话记录）

目标：让你**快速、统一、可维护**地给 Arete DSL 增加新语句，并做好别名、调试、兼容与测试。

---

## 1) 基础概念回顾
### `Statement`
```plain
interface Statement {
    suspend fun execute(ctx: ExecutionContext)

    fun interface Factory {
        fun create(args: Args, block: Statement?): Statement
    }
}
```

+ `execute(ctx)`：真正执行逻辑（可挂起，便于 `delay` / 切主线程等协程操作）。
+ `Factory.create(args, block)`：从参数 + 子块构造语句实例。**parser 负责传入 block**（例如 `seq { ... }` 的 {...}）。

### `Args`
```plain
fun interface Args {
    fun get(key: String): String?
    fun entries(): Map<String, String> = emptyMap()
}
```

+ `get(key)`：读取单个参数。
+ `entries()`：读取**完整键值对**（为一些“遍历入参”的语句预留，如 `var`/`meta` 等）。

---

## 2) 最小可用注册示例
### 2.1 写一个简单语句（例如：`message`）
```plain
// io/github/yuazer/arete/builtin/MessageStatement.kt
package io.github.yuazer.arete.builtin

import io.github.yuazer.arete.core.Args
import io.github.yuazer.arete.core.ExecutionContext
import io.github.yuazer.arete.core.Statement
import org.bukkit.entity.Player

class MessageStatement(private val text: String) : Statement {
    override suspend fun execute(ctx: ExecutionContext) {
        val p = ctx.playerOrNull() ?: return
        io.github.yuazer.arete.utils.extension.onMain {
            p.sendMessage(text)
        }
    }

    companion object {
        val factory = Statement.Factory { a, _ ->
            MessageStatement(
                text = a["text"] ?: a["t"] ?: error("message: missing text")
            )
        }
        private operator fun Args.get(k: String) = get(k)
    }
}
```

### 2.2 注册到注册表
```plain
// 例如在 onEnable 时机
StatementRegistry.register("message", MessageStatement.factory)
StatementRegistry.register("msg",     MessageStatement.factory) // 别名
```

运行期日志示例  
`§7[Arete] §7Registered §fmessage §7§a✔ §8(0.17 ms)`

---

## 3) 带子块的语句（如 `seq`/`ifchain`/`for`）
**要点**：`Factory.create(args, block)` 的 `block` 就是**大括号里的子语句**。

+ 有的语句**必须要子块**（如 `case` / `else` 的占位器）；
+ 有的语句**可选子块**（如 `for {...} { child }`）。

### 3.1 需要子块的工厂写法
```plain
val factory = Statement.Factory { a, block ->
    requireNotNull(block) { "your-statement: missing block" }
    YourBlockStatement(params..., block)
}
```

### 3.2 可选子块的工厂写法
```plain
val factory = Statement.Factory { a, block ->
    YourStatement(optional = block) // null 也能执行（比如仅初始化或仅赋值）
}
```



> 更新: 2025-10-17 15:26:59  
> 原文: <https://www.yuque.com/yuazer/blow95/shooalkoxgbvv08f>