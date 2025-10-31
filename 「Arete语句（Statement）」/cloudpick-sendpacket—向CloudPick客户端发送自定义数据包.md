# ✉️ cloudpick-sendpacket — 发送 CloudPick 自定义数据包

通过 CloudPick 的网络层向客户端发送一条**自定义数据包**，可携带任意顺序参数。  
语法与 `arcartx-sendpacket` 基本一致，只是底层调用改为 `PacketSender.sendCustomData(...)`。

---

## ⚙️ 参数列表

| 参数名             | 类型     | 必填 | 默认值     | 说明                                                        |
|-----------------|--------|----|---------|-----------------------------------------------------------|
| `id` / `packet` | String | ✅  | —       | 要发送的数据包 ID（客户端监听的包名）                                      |
| `target`        | String | 否  | `@self` | 目标模式：`@self` / `self` / `source`（保留），当未指定 `targetVar` 时生效 |
| `targetVar`     | String | 否  | —       | 从变量池中读取目标玩家，可为 `Player` 或 `Collection<Player>`，**优先级最高**  |
| 其他任意键           | String | 否  | —       | 所有非保留键都会作为 payload 参数，**按书写顺序发送**，支持 `${变量}` 插值           |

保留键（不会进入 payload）：`id`、`packet`、`target`、`targetVar`

---

## ⚡ 快速示例

### ① 向自身发送一个 UI 打开指令包

```plain
cloudpick-sendpacket {
  id = "ui_open_bag"
  tab = "pokemon"
  page = "1"
}
```

客户端会收到：

```text
("ui_open_bag", ["pokemon", "1"])
```

---

### ② 从变量中读取目标玩家

```plain
selectPlayer { name = "Yuazer" store = "p" }
cloudpick-sendpacket {
  id = "notify"
  targetVar = "p"
  title = "你有新的任务"
  content = "请前往主城找NPC"
}
```

当 `targetVar` 存在时，`target` 会被忽略。

---

### ③ 携带位置信息的包

```plain
storeLocation { at = "@self" store = "aimPosStr" }
cloudpick-sendpacket {
  id = "spawn_indicator"
  target = "self"
  pos = "${aimPosStr}"
  color = "green"
}
```

> 在服务端执行时，`${aimPosStr}` 会被替换为变量池中的真实值。

---

## 🧠 机制说明

1. **目标解析顺序**  
   1) 如果有 `targetVar` → 从变量池中取玩家 / 玩家集合  
   2) 否则如果执行者是 Player → 给执行者自己  
   3) 否则不发送

2. **参数顺序**  
   - 参数按照你在 DSL 中的**书写顺序**加入数据包
   - 如果运行环境不支持有序 entries，则会兜底读取 `data0` ~ `data50`

3. **变量插值**  
   - 所有参数都支持 `${变量名}` 的占位替换  
   - 替换来源是 `ctx.vars[...]`

---

## 🧩 对应服务端调用

语句内部最终会调用：

```kotlin
PacketSender.sendCustomData(player, packetId, *finalData)
```

---


> 更新: 2025-11-01 02:37:55  
> 原文: <https://www.yuque.com/yuazer/blow95/cloudpick-sendpacket>
