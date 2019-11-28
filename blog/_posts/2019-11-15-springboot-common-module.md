---
layout: post
title: springboot中抽出公共实体模型以依赖方式注入
date: 2019-11-23
Author: minweikai
tags: [技巧, 抽象]
comments: true
---

## 项目使用spring boot+jpa

在公司做业务开发时，可能会建多个子项目。

而每建一个子项目都有依赖相应的实体（Entity，对应数据库中的某个表）、数据仓库（Repository）。

这些Entity、Repository基本在每个子项目都相同。

所以我就将这些Entity、Repository抽成一个依赖包，使每个子项目都依赖着这个包。

好处：维护Entity、Repository和修改某个方法就相当方便；要添加新字段也很方便，不用去每个子项目中逐个添加。

注意：因为spring boot默认扫描本包下的Entity、Repository，所以当这些在其他包时需要在启动方法处添加自定义扫描路径注解。

```java
@EnableJpaAuditing
@SpringBootApplication
@EntityScan("com.cxd.repository.*.pojo.entity")
@EnableJpaRepositories(basePackages = "com.cxd.repository.*.dao.repository")
public class CxdWebApplication {
public static void main(String[] args) {
	SpringApplication.run(CxdWebApplication.class, args);
   }
}
```

```java
@EntityScan //扫描实体所在路径
@EnableJpaRepositories //扫描Repository所在路径
```

## 项目结构

![20191115181722460](https://img-blog.csdnimg.cn/20191115181722460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)

## 依赖模型包

![](https://img-blog.csdnimg.cn/20191115181905413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)

## 业务项目包

在业务项目中导入依赖模型包则可

![](https://img-blog.csdnimg.cn/20191115182033983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)

### 可能会遇到的启动项目时找不到某个实体/Repository

- 检查自己的依赖路径与注解路径是否一致
- 先clean依赖包后，install依赖包，clean项目包，然后再启动项目
- 多检查、多测试、多查询