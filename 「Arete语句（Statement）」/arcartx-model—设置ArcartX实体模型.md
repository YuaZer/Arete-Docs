# 🦋 arcartx-model — 设置 ArcartX 实体模型

## ⚙️ 参数列表
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `model`<br/> / `id`<br/> / `name` | String | ✅ | — | 模型 ID（注册于 ArcartX 资源中） |
| `scale` | Double | 否 | 1.0 | 模型缩放比例 |
| `target` | String | 否 | nearby | 目标模式：`self`<br/>、`target`<br/>、`pointer`<br/>、`nearby` |
| `targetVar` | String | 否 | — | 从变量池读取目标（优先级最高） |
| `at` | String | 否 | `@self` | 搜索中心位置（仅在 `nearby`<br/> 模式下有效） |
| `rx` | Double | 否 | 4.0 | 搜索范围 X |
| `ry` | Double | 否 | 2.0 | 搜索范围 Y |
| `rz` | Double | 否 | 4.0 | 搜索范围 Z |


## ⚡ 快速示例
① **命中设置模型**

```plain
raycast { range = 16 storeEntity = "lock" }
arcartx-model { model = "slime_king" scale = 1.2 targetVar = "lock" }
```

② **自身换装**

```plain
arcartx-model { model = "warrior_1" scale = 1.0 target = "self" }
```

③ **范围换模**

```plain
arcartx-model {
  model = "stone_golem"
  scale = 0.9
  target = "nearby"
  at = "@self"
  rx = 6
  ry = 3
  rz = 6
}
```



> 更新: 2025-10-18 13:50:12  
> 原文: <https://www.yuque.com/yuazer/blow95/vkx8zas4ddnugwrg>