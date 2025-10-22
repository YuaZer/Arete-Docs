# 如何创建一个简单的Kether模板

在Kether文件夹内新建文件夹或直接新建文件 check.yml

格式:

配置名: |  
  kether内容

**注意: kether内容的最后一行的返回类型必须是一个布尔类型(也就是要么true要么false)**

示例配置

```yaml
checkKether: |
  set a to player level
  check &a > 5
```



> 更新: 2025-10-17 09:19:43  
> 原文: <https://www.yuque.com/yuazer/blow95/cilyiraaab4w2c4l>