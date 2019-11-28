---
layout: post
title: QueryDSL使用拓展
date: 2019-10-28
Author: minweikai
tags: [技巧,工具]
comments: true
---

**如果是初次使用QueryDSL的同学建议去这篇博客：SpringDataJPA+QueryDSL玩转态动条件/投影查询，本文针探讨的是使用时遇到的一些问题：**

1. 如何使用Projections.bean投影属性到查询对象，简化代码？
2. 如何使用级联查询，关联同一张表两次？

## 1. Projections简化代码，使代码更优雅

使用Projections方法可以更简单更方便的返回自定义的参数属性

```Java

```

Projections的bean方法第一个属性是要查询对象的泛型类，对象中orderDetails.“commodityNo”属性就是CommoditySalesDto对应属性，大小写相同。如属性不同时可以使用as来为指定结果集添加别名对应dto内属性。

## 2. 关联同一张表两次进行查询

有时遇到一些查询需要在同一张表关联查询两次或多次，知道在sql中怎么写，但是在querydsl中就不知道怎么下手了，方法其实很简单

```Java

```

创建对应对象和别名，这样关联查询时才会区分。

资料：

[github-querydsl资源](https://github.com/querydsl/querydsl/)