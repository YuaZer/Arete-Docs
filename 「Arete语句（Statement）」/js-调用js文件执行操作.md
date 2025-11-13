# JS 语句使用教程

本文档介绍如何在 Arete 中使用 `js`/`javascript` 语句调用 `script` 目录中的 JavaScript 脚本。阅读完后，你将能够：

- 理解脚本文件的放置位置与命名规则
- 在技能配置中正确编写 `js { ... }` 语句
- 在脚本内部获取 Arete 提供的上下文对象与自定义参数
- 排查常见的执行错误

## 1. 脚本目录结构

- 插件首次启动时会在插件数据目录（通常为 `plugins/Arete/`）下创建 `script/` 文件夹。
- `script/` 文件夹中的所有子文件夹都会被递归扫描，后缀为 `.js` 的文件会被注册为可调用的脚本。
- `js` 语句的 `path` 参数使用**相对路径**（不含 `.js` 后缀）。例如：
    - 文件位于 `plugins/Arete/script/示例/传送到指定坐标上方.js`
    - 语句中的写法为 `path = "示例/传送到指定坐标上方"`
- 保存或新增脚本后，可通过插件提供的重载命令（如 `/arete reload`，视服务器配置而定）或重启服务器，让 `JsScriptManager` 重新加载脚本。

## 2. 在技能中调用脚本

`js` 与 `javascript` 两个关键字等价。语法示例：

```kether
js {
  path = "示例/传送到指定坐标上方"
  上下文实体变量 = "${aimEnt}"      # 自定义的实体参数
  上下文Location变量 = "${aimPos}"   # 自定义的坐标参数
  高度 = "5"                          # 额外的数值参数
}
```

编写技巧：

- `path` 必填，其余键值对会被作为参数传给脚本。
- 所有值默认以字符串形式出现，如果需要传递变量，请使用 `"${变量名}"` 的写法，Arete 会自动解析变量。
- 你也可以直接写常量，例如 `消息 = "你好，世界"` 或 `开关 = "true"`。

### 内置可用变量

即使未显式传参，以下键也会自动注入到脚本的参数表中，便于访问：

| 参数键      | 类型                       | 说明                                     |
|----------|--------------------------|----------------------------------------|
| `source` | `CommandSender`/`Entity` | 语句的发起者（通常是技能释放者）。                      |
| `target` | `Entity?`                | 当前技能命中的实体，可为空。                         |
| `player` | `Player`                 | `ctx.playerOrNull()`，若为空则回退到 `source`。 |
| `origin` | `Location`               | 技能的起始坐标，已复制为独立对象。                      |

以上参数也会以中文键名写入 `参数` 集合，如 `参数["源"]` 不存在，推荐通过英文键访问。

## 3. 脚本内获取参数与上下文

当脚本执行时，Arete 会提供以下全局变量：

| 变量名                                | 内容                                                          |
|------------------------------------|-------------------------------------------------------------|
| `ctx` / `context`                  | `ExecutionContext`，可调用 `ctx.vars`、`ctx.resolveVar()` 等工具方法。 |
| `source`、`target`、`player`         | 与上文描述一致，便于快速访问。                                             |
| `参数` / `params` / `param` / `args` | `MutableMap<String, Any?>`，包含所有传入参数与内置参数。                   |
| `scriptPath`                       | 当前执行脚本的相对路径（含子目录）。                                          |

在脚本中推荐使用 `参数`（或 `params`）读取你在技能中传入的键值。示例：

```javascript
var 参数表 = 参数 || {};
var entity = 参数表["上下文实体变量"] || ctx.player;
var location = 参数表["上下文Location变量"] || ctx.origin;
var height = parseFloat(参数表["高度"] || 3);

if (!location) {
  throw new Error("未提供可用的坐标");
}

var targetLocation = location.clone();
targetLocation.add(0, height, 0);

entity.teleport(targetLocation);
```

> 提示：为避免 `ClassCastException`，如果参数可能是字符串，先进行类型判断或转换。

## 4. 调试与错误处理

- 如果脚本路径填写错误或文件未被加载，控制台会输出：`Script not found: ...`。
- 执行过程中出现的异常会被捕获并写入控制台，例如语法错误、调用未定义方法等。利用日志中的栈追踪定位问题。
- 当脚本报错时，后续语句仍会继续执行；若希望中断流程，可在脚本中抛出异常并在技能中自行处理。
- 为提升可读性，可在脚本顶部使用多行注释说明参数用途与注意事项。

## 5. 完整示例

### 5.1 技能配置片段

```arete
# 假设前面已有 raycast 语句写入 aimPos / aimEnt
js {
  path = "示例/传送到指定坐标上方"
  上下文实体变量 = "${aimEnt}"
  上下文Location变量 = "${aimPos}"
  高度 = "5"
}
```

### 5.2 对应脚本 `script/示例/传送到指定坐标上方.js`

```javascript
/**
 * 将实体传送到目标坐标的上方。
 */
var 参数表 = 参数 || {};
var teleportEntity = 参数表["上下文实体变量"] || 参数表.source || ctx.player || ctx.source;
var baseLocation = 参数表["上下文Location变量"] || 参数表.origin || ctx.origin;
var extraHeight = 参数表["高度"] || 3;

if (!baseLocation) {
  throw new Error("未提供可用的目标坐标，无法执行传送");
}

if (typeof extraHeight === "string") {
  extraHeight = parseFloat(extraHeight) || 0;
}

var targetLocation = baseLocation.clone();
targetLocation.add(0, extraHeight, 0);

if (teleportEntity && typeof teleportEntity.teleport === "function") {
  teleportEntity.teleport(targetLocation);
  if (ctx.player && typeof ctx.player.sendMessage === "function") {
    ctx.player.sendMessage("§a已通过 JS 脚本传送至指定位置上方");
  }
} else {
  throw new Error("传入的实体不支持传送: " + teleportEntity);
}
```

将上述片段保存后，即可在技能中通过 `js` 语句调用脚本，完成自定义逻辑。

---

# JS 模块脚本编写提示词模板

以下提示词可直接粘贴给 AI，让其按照 Arete 插件的 JS 模块要求生成脚本。请在标注為 `___` 的位置补充你的具体需求或变量名称。

---

```
你现在是一名 Minecraft 服务器插件脚本开发助手，请基于下列信息编写 JavaScript 脚本：

【脚本基础信息】
- 目标：___（描述脚本要完成的功能，例如“让命中的实体受到燃烧效果并播放粒子”）。
- 存放路径：script/___/___ .js（在 `config.yml` 同级的 `script` 目录下创建对应的子目录和文件）。
- 运行环境：Arete 自带的 JS 模块，可通过 `js { path = "..." }` 语句调用。

【可使用的上下文参数】
- `参数.source`：触发技能的默认实体，与 `ctx.source` 相同。
- `参数.上下文实体变量`：在技能语句中通过 `上下文实体变量 = "${...}"` 传入的实体对象。
- `参数.上下文Location变量`：在技能语句中通过 `上下文Location变量 = "${...}"` 传入的坐标对象。
- 其他通过 `参数.xxx` 形式注入的附加参数（若有，请在此处列出：___）。

【脚本设计要求】
1. 详细说明脚本需要执行的步骤，例如：
   - 第一步：___
   - 第二步：___
2. 如果脚本需要读取或写入变量，请说明变量用途（例如 `let target = 参数.上下文实体变量;`）。
3. 如需对坐标进行偏移或计算，请明确说明偏移量或算法。
4. 若要向玩家发送消息或播放效果，请描述消息内容或效果参数。
5. 请确保脚本最后包含必要的容错或判空逻辑（例如判断 `参数.上下文实体变量` 是否存在）。

【输出格式】
- 返回完整的 JavaScript 代码块，包含必要的注释，便于我理解每一步。
- 代码顶部注明作者或用途：`// 功能：___`。
- 如果需要额外说明或部署步骤，请在代码块后单独写出。
```

---

使用方式：将模板复制给 AI，并补充 `___` 部分即可生成符合 JS 模块规范的脚本。