# 🐺 arcartx-anim — 播放 ArcartX 动画

## ⚙️ 参数列表
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `animation`<br/> / `name` | String | ✅ | — | 动画名（ArcartX 模型定义内的动画） |
| `speed` | Double | 否 | 1.0 | 动画播放速度倍数 |
| `transition` | Int | 否 | 1 | 动画过渡时间（tick） |
| `keep` | Long | 否 | 1000 | 动画保持时长（毫秒） |
| `target` | String | 否 | nearby | 目标模式：`self`<br/>、`target`<br/>、`pointer`<br/>、`nearby` |
| `targetVar` | String | 否 | — | 从变量池读取目标（优先级最高） |
| `at` | String | 否 | `@self` | 搜索中心位置 |
| `rx` | Double | 否 | 4.0 | 搜索范围 X |
| `ry` | Double | 否 | 2.0 | 搜索范围 Y |
| `rz` | Double | 否 | 4.0 | 搜索范围 Z |


## ⚡ 快速示例
① **命中播放动画**

```plain
raycast { range = 20 storeEntity = "hit" }
arcartx-anim { animation = "HitReact_Small" transition = 2 keep = 600 targetVar = "hit" }
```

② **自身待机动画**

```plain
arcartx-anim { animation = "Idle_Breath" speed = 1.0 transition = 5 keep = 5000 target = "self" }
```

③ **范围群体动画**

```plain
arcartx-anim {
  animation = "Spawn_In"
  transition = 3
  keep = 1200
  target = "nearby"
  at = "@self"
  rx = 6
  ry = 3
  rz = 6
}
```



> 更新: 2025-10-18 13:50:50  
> 原文: <https://www.yuque.com/yuazer/blow95/mr0ouck1b1iygrxu>