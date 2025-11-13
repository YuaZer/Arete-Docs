# Arete JS 脚本教程

> 本教程适用于在 Arete 技能系统中使用 JS 引擎（JsEngineProvider）编写脚本的开发者。内容涵盖脚本结构、上下文对象、参数处理、常用工具函数以及调试技巧，帮助你快速编写可维护的 JS 语句脚本。

## 1. 基础概念

- **脚本入口**：所有脚本必须提供 `main(ctx, 参数)` 函数作为执行入口，JS 引擎会自动调用该函数。
- **上下文对象 (`ctx`)**：由技能运行时传入，通常包含玩家、技能、目标等上下文信息。
- **参数对象 (`参数`)**：在技能配置中通过 `js { ... }` 传递的参数集合，可使用中文或英文键名。

```javascript
function main(ctx, 参数) {
    // 入口逻辑
}
```

## 2. 推荐的脚本结构

保持入口函数简洁，并将复杂逻辑拆分为辅助函数以便复用与测试。

```javascript
function main(ctx, 参数) {
    const 实体 = 解析实体(ctx, 参数);
    const 目标 = 计算目标位置(ctx, 参数);
    传送实体(实体, 目标);
}

function 解析实体(ctx, 参数) {
    return 参数["上下文实体变量"] || 参数.entity || 参数.source || ctx.player || ctx.source;
}

function 计算目标位置(ctx, 参数) {
    const 基础坐标 = 参数["上下文Location变量"] || 参数.location || 参数.origin || ctx.origin;
    if (!基础坐标) {
        throw new Error("未提供可用的目标坐标");
    }
    const 高度 = 解析数字(参数["高度"], 3);
    const 目标 = 基础坐标.clone();
    目标.add(0, 高度, 0);
    return 目标;
}

function 传送实体(实体, 位置) {
    if (!实体 || typeof 实体.teleport !== "function") {
        throw new Error("传入的实体不支持传送");
    }
    实体.teleport(位置);
}

function 解析数字(原始值, 默认值 = 0) {
    if (原始值 == null) return 默认值;
    const 数值 = typeof 原始值 === "string" ? parseFloat(原始值) : 原始值;
    return isNaN(数值) ? 默认值 : 数值;
}
```

## 3. ctx 常见字段

| 字段          | 类型       | 说明               |
|-------------|----------|------------------|
| `player`    | Player   | 触发技能的玩家实体        |
| `source`    | Entity   | 触发来源（可能是怪物、投射物等） |
| `origin`    | Location | 技能执行时的原点坐标       |
| `skill`     | Skill    | 当前执行的技能对象        |
| `variables` | Map      | 技能变量，可从中读取自定义数据  |

> 注意：具体字段可能因触发方式或版本而异，建议在脚本中做好空值判断。

## 4. 参数处理技巧

1. **允许多种键名**：兼容中文与英文键名，提升脚本的通用性。
2. **提供默认值**：对于可选参数使用 `??` 或辅助函数提供默认值。
3. **字符串数字转换**：在需要数字的场景使用 `parseFloat` 或 `Number` 转换输入。
4. **布尔值解析**：
   ```javascript
   function 解析布尔(值, 默认值 = false) {
       if (值 == null) return 默认值;
       if (typeof 值 === "boolean") return 值;
       return ["true", "1", "yes", "是"].includes(String(值).toLowerCase());
   }
   ```

## 5. 常见上下文工具函数

```javascript
function 发送玩家消息(ctx, 文本) {
    if (ctx.player && typeof ctx.player.sendMessage === "function") {
        ctx.player.sendMessage(文本);
    }
}

function 获取目标列表(ctx, 参数) {
    if (Array.isArray(参数.targets)) return 参数.targets;
    if (ctx.targets && Array.isArray(ctx.targets)) return ctx.targets;
    return ctx.player ? [ctx.player] : [];
}
```

将通用函数放在脚本顶部或单独模块中，方便重复使用。

## 6. 调试与日志

- 使用 `logger`：若运行环境提供 `logger` 对象，可调用 `logger.info(...)` 输出调试信息。
- 临时使用 `print`：`print()` 也可用于快速打印，但请在上线前移除以免污染日志。
- 异常信息要清晰：在 `throw new Error()` 时提供明确的错误描述，便于定位问题。

## 7. 编写可维护脚本的建议

1. **命名清晰**：变量、函数使用描述性名称，必要时添加注释。
2. **早期返回**：在参数不足或条件不满足时尽早返回或抛错。
3. **封装重复逻辑**：将多处使用的逻辑提取为函数，保持入口简洁。
4. **保持幂等**：尽量避免脚本重复执行带来副作用，可通过状态变量控制。

## 8. 常见问题 FAQ

- **Q：为什么脚本不执行？**
    - 请确认脚本内是否定义并导出 `main(ctx, 参数)`。
    - 检查技能配置是否正确引用该脚本。

- **Q：如何传递多个参数？**
    - 在技能中使用 `js { 键1: 值1, 键2: 值2 }`，在脚本中通过 `参数.键1` 获取。

- **Q：如何调用其他脚本？**
    - 建议将共用逻辑抽成函数并复制到需要的脚本，或通过模块化机制（若引擎支持）。

## 9. 完整示例

```javascript
function main(ctx, 参数 = {}) {
    const 玩家 = ctx.player;
    const 提示文本 = 参数.message || "§a技能已触发";
    const 是否广播 = 解析布尔(参数.broadcast, false);

    if (!玩家) {
        throw new Error("未找到触发该技能的玩家");
    }

    if (是否广播 && typeof ctx.server?.broadcast === "function") {
        ctx.server.broadcast(提示文本);
    } else {
        发送玩家消息(ctx, 提示文本);
    }
}

function 解析布尔(值, 默认值 = false) {
    if (值 == null) return 默认值;
    if (typeof 值 === "boolean") return 值;
    return ["true", "1", "yes", "是"].includes(String(值).toLowerCase());
}

function 发送玩家消息(ctx, 文本) {
    if (ctx.player && typeof ctx.player.sendMessage === "function") {
        ctx.player.sendMessage(文本);
    }
}
```

## 10. 后续资源

- 查看 `/script/示例` 目录下的官方示例脚本。
- 关注 Arete 项目的更新日志了解脚本引擎的最新特性。

> 若你在编写脚本过程中遇到问题，欢迎反馈以便我们完善教程内容。

# Arete JS 脚本生成 AI 提示词

> 通过以下提示词模板，可以快速指导 AI 生成符合 Arete 引擎规范的 JS 脚本及其调用示例。根据实际需求填充其中的占位符即可。

## 1. 通用提示词模板

```
你是一名熟悉 Arete 技能系统的脚本开发者，现需编写一段 JS 脚本并给出示例调用。

【背景信息】
- 技能目标：<描述技能要实现的效果，例如“将玩家传送到最近的安全区并广播提示”>
- 可用上下文：ctx.player, ctx.source, ctx.origin, ctx.variables, ctx.server
- 额外素材：<可选，列出相关 API 或已有函数，如“有一个工具函数 safeTeleport(entity, location)”>

【脚本要求】
1. 必须定义 `function main(ctx, 参数 = {})` 作为入口。
2. 入口逻辑应拆分成 1~3 个辅助函数，保持结构清晰。
3. 需要对关键参数进行校验，并提供默认值或报错信息。
4. 所有提示信息使用彩色文本，例如 `§a`、`§c`。
5. 在给出脚本后，再提供一段技能配置中的 `js { ... }` 调用示例，展示参数如何传入。

【输出格式】
1. 标题：`## 脚本代码`
2. 使用 ```javascript 代码块输出脚本。
3. 标题：`## 调用示例`
4. 使用 ```text 代码块展示 `js { ... }` 配置写法。
5. 若有注意事项，使用项目符号列出。
```

> 使用时只需替换尖括号内的内容，如有更多上下文或约束，也可添加到“背景信息”或“脚本要求”中。

## 2. 不同场景的补充说明

根据实际需求，可在模板中加入以下补充：

- **多目标操作**：提醒 AI `参数.targets` 或 `ctx.targets` 可能为目标列表，需要循环处理。
- **冷却/条件判断**：要求脚本检查 `ctx.variables` 中的状态或冷却时间。
- **数值计算**：说明需要解析浮点数、百分比或随机值，并给出范围限制。
- **安全校验**：强调在调用可能为空的对象方法前进行类型判断。

## 3. 示例：群体治疗技能

以下演示如何使用模板填充信息并得到完整输出：

```
你是一名熟悉 Arete 技能系统的脚本开发者，现需编写一段 JS 脚本并给出示例调用。

【背景信息】
- 技能目标：对附近队友施加持续治疗效果，并在聊天栏提示玩家。
- 可用上下文：ctx.player, ctx.origin, ctx.targets, ctx.variables, ctx.server

【脚本要求】
1. 必须定义 `function main(ctx, 参数 = {})` 作为入口。
2. 入口逻辑拆分为：解析目标、施加治疗、广播提示。
3. 参数包含 `radius`（半径，默认 5）、`hotDuration`（持续时间，秒，默认 10）。
4. 对缺失的玩家或目标列表给出报错或警告。
5. 在给出脚本后，再提供技能配置中的 `js { ... }` 示例。

【输出格式】
1. 标题：`## 脚本代码`
2. 使用 ```javascript 代码块输出脚本。
3. 标题：`## 调用示例`
4. 使用 ```text 代码块展示 `js { ... }` 配置写法。
```
```

> 将以上描述提交给 AI 后，可直接获得结构完善的脚本与调用示例，大幅提升编写效率。

## 4. 提示词使用小贴士

- **明确上下文**：尽量列出可用的 ctx 字段与外部函数，避免 AI 假设不存在的 API。
- **限定输出格式**：提前约定标题、代码块类型和语言，可以减少后续整理工作。
- **强调错误处理**：让 AI 主动校验参数并输出友好错误信息，提高脚本可靠性。
- **复用模板**：将模板保存到常用笔记中，按需增删条目即可适配不同脚本需求。

## 5. 相关资源

- [《Arete JS 脚本教程》](js脚本教程.md)：了解脚本规范与最佳实践。
- `src/main/resources/script/示例/` 目录：官方示例脚本集合，可作为 AI 生成脚本的参考风格。