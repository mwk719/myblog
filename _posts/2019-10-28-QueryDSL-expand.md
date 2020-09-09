---
layout: post
title: 使用QueryDSL过程中你肯定遇到过的问题
date: 2019-10-28
Author: minweikai
tags: [技巧,工具]
comments: true
---

*来自官网的介绍（翻译版）*

### 前言

Querydsl是一个框架，可用于构造静态类型的类似SQL的查询。可以通过诸如Querydsl之类的流畅API构造查询，而不是将查询编写为内联字符串或将其外部化为XML文件。

例如，与简单字符串相比，使用流利的API的好处是

- 在IDE中使用代码完成；会有代码提示和自动补全，较为高效
- (几乎)语法安全；
- 可以安全地引用域类型和属性；可以直接使用领域模型进行操作，毕竟本质就是面向对象
- 更好地重构域类型的更改；
- 跟写SQL一样的方便；

### 1. 简介

#### 1.1. 背景

Querydsl是出于以类型安全的方式维护HQL查询的需要而诞生的。HQL查询的增量构造需要String连接，并导致难以阅读的代码。通过纯字符串对域类型和属性的不安全引用是基于字符串的HQL构造的另一个问题。

随着域模型的不断变化，类型安全性在软件开发中带来了巨大的好处。域更改直接反映在查询中，而查询构造中的自动完成功能使查询构造更快，更安全。

用于Hibernate的HQL是Querydsl的第一种目标语言，但如今它支持JPA，JDO，JDBC，Lucene，Hibernate Search，MongoDB，Collections和RDFBean作为后端。

#### 1.2. 原则

*类型安全* 是Querydsl的核心原则。查询是根据生成的反映查询类型的属性来构造的。函数/方法调用也以完全类型安全的方式构造。

*一致性* 是另一个重要原则。在所有实现中，查询路径和操作都是相同的，而且Query接口具有公共的基本接口。

要了解Querydsl查询和表达式类型的表达能力，请访问javadocs并进行探索`com.querydsl.core.Query`，`com.querydsl.core.Fetchable` 以及`com.querydsl.core.types.Expression`。

**如果是初次使用QueryDSL的同学建议去这篇博客：**[SpringDataJPA+QueryDSL玩转态动条件/投影查询](https://blog.csdn.net/phapha1996/article/details/83614975)，

本文针探讨的是使用时遇到的一些问题

### 2. 拓展示例

#### 1. Projections简化代码，使代码更优雅

使用Projections方法可以更简单更方便的返回自定义的参数属性

```Java
QHajOrderDetails orderDetails = QHajOrderDetails.hajOrderDetails;
 
return jpaQueryFactory.select(
			Projections.bean(
				CommoditySalesDto.class,
                  // 取别名，与CommoditySalesDto实体中字段相同
				orderDetails.number.sum().as("sales"),
				orderDetails.commodityNo
			)
		).from(orderDetails)
			.where(orderDetails.commodityNo.in(commodityNos))
			.groupBy(orderDetails.commodityNo)
			.fetch();
```

Projections的bean方法第一个属性是要查询对象的泛型类，对象中orderDetails.“commodityNo”属性就是CommoditySalesDto对应属性，大小写相同。如属性不同时可以使用as来为指定结果集添加别名对应dto内属性。

#### 2. 关联同一张表两次进行查询

有时遇到一些查询需要在同一张表关联查询两次或多次，知道在sql中怎么写，但是在querydsl中就不知道怎么下手了，方法其实很简单

```Java
QHajCommodityType type1 = new QHajCommodityType("type1");
QHajCommodityType type2 = new QHajCommodityType("type2");
 
return jPAQueryFactory.select(type2.id)
	    .from(type1)
	    .join(type2).on(type1.id.eq(type2.parentId))
		.fetch();
```

创建对应对象和别名，这样关联查询时才会区分。

#### 3. 格式化字段进行查询

```java
//获取到每日订单数量
QHajOrder order = QHajOrder.hajOrder;
		//格式化字段，按每日格式化
		StringTemplate dateExpr = Expressions.stringTemplate("DATE_FORMAT({0},'%Y-%m-%d')", order.createTime);
		return jpaQueryFactory.select(dateExpr,order.id.count())
				.from(order)
				.orderBy(order.id.desc())
				.groupBy(dateExpr)
				.limit(5)
				.fetch();
```

#### 4. 可添加判断逻辑，根据业务需要拼接，在代码中书写更便捷

```java
QHajOrder order = QHajOrder.hajOrder;
JPAQuery<Integer> jpaQuery = jpaQueryFactory.select(order.id.sum()).from(order);
Integer count;
if (ObjectUtil.isNotNull(type)) {
    count = jpaQuery.where(order.status.eq(0)).fetchOne();
} else {
    count = jpaQuery.where(order.status.eq(1)).fetchOne();
}
// 数据为空时，值为null
if (ObjectUtil.isNull(count)) {
    count = 0;
}
return count;
```

#### 5. 分页处理

```Java
public List<HajOrder> getOrderListByPage(Pager page){
	QHajOrder order = QHajOrder.hajOrder;
	QueryResults<HajOrder> queryResults = jpaQueryFactory
			.selectFrom(order)
			.where(order.status.eq(0))
        	 // 分页逻辑
			.offset((page.getCurrentPage() - 1) * page.getPageSize())
			.limit(page.getPageSize())
			.fetchResults();
    // 总条数
	page.setRecordTotal((int) queryResults.getTotal());
	return queryResults.getResults();
}
```

#### 6.多数据源配置使用

如果你还没配置多数据源使用，可以参照这个博客 [Spring Boot 2.x基础教程：Spring Data JPA的多数据源配置](http://blog.didispace.com/spring-boot-learning-21-3-8/)

之前一直是单数据源使用，在多数据源中使用可能会报这样的错

```java
java.lang.IllegalArgumentException: org.hibernate.hql.internal.ast.QuerySyntaxException: * is not mapped

at org.springframework.orm.jpa.ExtendedEntityManagerCreator$ExtendedEntityManagerInvocationHandler.invoke(ExtendedEntityManagerCreator.java:350)
	at com.sun.proxy.$Proxy119.createQuery(Unknown Source)
	at com.querydsl.jpa.impl.AbstractJPAQuery.createQuery(AbstractJPAQuery.java:101)
	at com.querydsl.jpa.impl.AbstractJPAQuery.fetchResults(AbstractJPAQuery.java:211)
```

配置多个实体管理器EntityManager

```
@SpringBootApplication
@EnableJpaAuditing
public class ApiApplication {

	public static void main(String[] args) {
		SpringApplication.run(ThirdApiApplication.class, args);
	}

	/**
	 * Spring管理JPAQueryFactory
	 * 默认
	 * @param entityManager
	 * @return
	 */
	@Bean
	public JPAQueryFactory jpaQueryFactory(@Qualifier("entityManagerPrimary") EntityManager entityManager) {
		return new JPAQueryFactory(entityManager);
	}

	/**
	 * 新配置数据源wmsJpaQueryFactory
	 *
	 * @param entityManager
	 * @return
	 */
	@Bean
	public JPAQueryFactory wmsJpaQueryFactory(@Qualifier("entityManagerSecondary") EntityManager entityManager) {
		return new JPAQueryFactory(entityManager);
	}

}
```

业务使用中按如下操作就可以了

```java
//默认的数据源
@Autowired
private JPAQueryFactory jpaQueryFactory;

//新的数据源
@Autowired
private JPAQueryFactory wmsJpaQueryFactory;

public void test() {
	QCustomer customer = QCustomer.customer;
    Customer bob = wmsJpaQueryFactory.select(customer)
      .from(customer)
      .where(customer.firstName.eq("Bob"))
      .fetchOne();
}

```

### 资料：

[github-querydsl资源](https://github.com/querydsl/querydsl/)

[Querydsl官网](http://www.querydsl.com/)

[官网querydsl-jpa示例](http://www.querydsl.com/static/querydsl/latest/reference/html/ch02.html#jpa_integration)

[Querydsl参考指南](http://www.querydsl.com/static/querydsl/4.1.3/reference/html_single/#preface)

