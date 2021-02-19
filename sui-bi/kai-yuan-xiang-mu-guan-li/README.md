# 开源项目管理

### 问题处理流程

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAg5oql5ZGK6ICFLS0-PumXrumimDogMS4g5Yib5bu66Zeu6aKYXG4gICAgTm90ZSByaWdodCBvZiDpl67popg6IOKRoCDov5DooYznjq_looM8YnIvPuKRoSDpl67popjlpI3njrA8YnIvPuKRoiDpooTmnJ_nu5Pmnpw8YnIvPuKRoyDmoIfnrb5zdGFnZTogd2FpdGluZ1xuICAgIOaPkOS6pOiAhS0tPj7pl67popg6IDIuIOaMh-WumumXrumimOWkhOeQhuW5tuabtOaWsOagh-etvlxuICAgIE5vdGUgcmlnaHQgb2Yg6Zeu6aKYOiDmoIfnrb5zdGFnZTogZG9pbmdcbiAgICDmj5DkuqTogIUtLT4-5LuT5bqTOiAzLiDlj5Hotbfmj5DkuqTmiJZQUlxuICAgIE5vdGUgcmlnaHQgb2Yg5o-Q5Lqk6ICFOiDmj5DkuqTpmYTluKbpl67popjnvJblj7dcbiAgICDmj5DkuqTogIUtLT4-6Zeu6aKYOiA0LiDmm7TmlrDmoIfnrb5cbiAgICBOb3RlIHJpZ2h0IG9mIOmXrumimDog5qCH562-c3RhZ2U6IGRvbmVcbiAgICDmj5DkuqTogIUtLT4-5a6h6ZiF6ICFOiA1LiDmjIflrprpl67popjlrqHpmIVcbiAgICBOb3RlIHJpZ2h0IG9mIOWuoemYheiAhTog5a6h6ZiF5bm26aqM6K-B57uT5p6cPGJyLz7kuI3pgJrov4fliJnov5Tlm57mraXpqqQyXG4gICAg5a6h6ZiF6ICFLS0-PumXrumimDogNi4g5pu05paw5qCH562-XG4gICAgTm90ZSByaWdodCBvZiDpl67popg6IOagh-etvmxndG1cbiAgICDlrqHpmIXogIUtLT4-57u05oqk6ICFOiA3LiDmjIflrprpl67popjlrqHpmIVcbiAgICBOb3RlIHJpZ2h0IG9mIOe7tOaKpOiAhTog5a6h6ZiF5LiN6YCa6L-HPGJyLz7liJnov5Tlm57mraXpqqQyXG4gICAg57u05oqk6ICFLS0-PumXrumimDogOC4g5YWz6ZetaXNzdWVcbiAgICDnu7TmiqTogIUtLT4-5o-Q5Lqk6ICFOiA5LiDmjIflrprpl67popjmnIDnu4jlrozmiJDogIUiLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9fQ)

**备注**

* 问题报告者\(reporter\)可以是任何成员
* 提交者提交完代码后需要在issue中说明验证的方法
* 审阅者一般选择除提交者外较为熟悉相关代码模块的成员
* 审阅者需要根据提交者提供的验证方法验证结果

### 问题模板

**bug模板**

```text
**发生了什么**:
<!-- 此处补充问题产生的结果 -->

**预期的结果**:
<!-- 此处补充正常情况下应该是什么结果 -->

**如何重现**:
<!-- 此处补充复现问题的步骤 -->

**环境**:

- 操作系统: 
- 程序版本:
- 机器配置:
```

**新特性模板**

```text
**哪些方面的提升**
<!-- 此处补充该新特性所带来的好处 -->

**原因或需求**
<!-- 此处补充实现该新特性的缘由或现实需求 -->
```

### 提交信息格式

```text
{{ 所属模块 }} {{ issue编号 }}: {{ 简单描述 }}

{{ 详细描述 }}

Signed-off-by: {{ 用户名 }} <{{ 用户邮箱 }}>
```

### 分支管理

![](../../.gitbook/assets/image%20%2812%29.png)

