# ✉️ arcartx-sendpacket — 向 ArcartX 客户端发送自定义数据包

用于通过 **ArcartX 网络层**向客户端发送自定义数据包（Custom Packet）。  
可自定义目标玩家、参数顺序及变量替换，用于客户端 UI、特效、脚本通信等扩展。

---

## ⚙️ 参数列表

| 参数名                        | 类型     | 必填 | 默认值     | 说明                                                               |
|----------------------------|--------|----|---------|------------------------------------------------------------------|
| `id` / `packet`            | String | ✅  | —       | 要发送的数据包 ID（客户端监听的包名）                                             |
| `target`                   | String | 否  | `@self` | 目标模式：<br/>• `@self` — 执行者自身<br/>• `target` — 当前触发目标<br/>• 其他（保留） |
| `targetVar`                | String | 否  | —       | 从变量池读取目标（可为 Player 或 Collection\<Player\>），优先级高于 `target`        |
| `data0`, `data1`... / 任意键名 | String | 否  | —       | 按顺序传入的数据字段，都会发送给客户端作为 payload 参数（支持 `${变量}` 插值）                  |

---

## ⚡ 快速示例

### ① 向命中的实体发送 UI 包

```plain
raycast { range = 20 storeEntity = "hit" }
arcartx-sendpacket {
  id = "ui_hit_feedback"
  targetVar = "hit"
  text = "你被击中了！"
  power = "${damage}"
}
