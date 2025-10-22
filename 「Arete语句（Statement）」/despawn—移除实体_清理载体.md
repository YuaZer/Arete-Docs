# 🗑 despawn — 移除实体 / 清理载体

## 🧩 功能简介
`despawn` 用于**移除（remove）一个或一批实体**。  
从 `ctx.vars` 中读取 `target` 指向的变量，变量可以是：

+ 单个 `Entity`
+ `Collection<*>`（列表/集合，内部会 `filterIsInstance<Entity>()` 批量移除）

该语句始终在**主线程**执行移除，避免异步世界访问问题。

---

## ⚙️ 语法
```plain
despawn {
  target = "变量名"   # ctx.vars 中的实体 或 实体集合
}
```

### 参数
| 键 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `target` | String | ✅ | 变量名；支持单体 `Entity`<br/> 或 `Collection`<br/>（会过滤成实体后逐个移除） |


若变量不存在或不是以上两类，语句将**安静返回**，不抛错。

---

## ⚡ 快速示例
### ① 移除单个盔甲架
```plain
# 之前用 armorstand 生成并存入 s1
armorstand { id = "s1" at = "@self" marker = true }
# ……中间做一些轨迹/特效……
despawn { target = "s1" }
```

### ② 批量移除：先选取一圈展示体，再统一清理
```plain
# 假设你把一批展示体或生物放进 victims
target-entity { center = "@self" radius = 6 type = "armorstand" includeSelf = false storeList = "victims" }
despawn { target = "victims" }
```

### ③ 延时清理：演出 2 秒后回收
```plain
armorstand { id = "orb" at = "@self" marker = true }
# …做 move/particle 等操作…
delay 40
despawn { target = "orb" }
```

### ④ 结合 ifchain 判空
```plain
ifchain {
  case { cond = "var.sword != null" } { despawn { target = "sword" } }
  else { message { text = "&7没有可清理的实体。" } }
}
```



> 更新: 2025-10-17 15:09:52  
> 原文: <https://www.yuque.com/yuazer/blow95/zp5gcl8k2vutv02c>