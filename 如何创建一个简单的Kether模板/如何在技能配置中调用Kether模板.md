# 如何在技能配置中调用Kether模板

正如前文中的简单技能的示例配置所示

一个技能的释放前的Kether检查有**checkBeforeCast**和**checkBeforeCastConfig**两项配置

我们的Kether模板的调用，需要写在**checkBeforeCastConfig**配置里



格式举例:

```yaml
checkBeforeCastConfig:
  - "模板路径/模板配置名1"
  - "模板路径/模板配置名2"
  - "模板路径a/模板路径b/模板配置名1"
```

例如:

```yaml
checkBeforeCastConfig:
  "check/checkKether"
```



> 更新: 2025-10-17 09:22:21  
> 原文: <https://www.yuque.com/yuazer/blow95/bm98plnlpsu76xm0>