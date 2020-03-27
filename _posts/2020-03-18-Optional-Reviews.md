---
llayout: post
title: 最常见的Optional操作
date: 2020-03-18
Author: minweikai
tags: [技巧]
comments: true
---

## 新特性

1. Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。
2. Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。
3. Optional 类的引入很好的解决空指针异常。

## 好处

- 可以对不确定对象（可能为null），取其中某属性的值（可能为nul）用很简洁的方式进行提取；好处也不止如此。。

## 使用技巧

1. ```java
   Optional.of() //指定的属性不能为null，否则会抛出NPE；
   ```

2. ```java
   Optional.ofNullable() //指定的属性可以为null；
   ```

3. ```java
   //.isPresent()是否存在 
   Map m =new HashMap();
   Optional.ofNullable(m.get(1)).isPresent() //判断该map中key值为1是否存在，m可以为null
   ```

4. ```java
   Map m =new HashMap();
   Optional.ofNullable(m.get(1)).get()； //取出该值，不存在时抛出NPE；
   //.orElse(default)值不存在时取出default的值
   Optional.ofNullable(m.get(1)).orElse(10); //取出该值，不存在时取10；
   ```

5. ```java
   UserDto userDto =new UserDto();
   userDto.setUserName("http://minwk.top/");
   //.map()可以映射返回对象指定属性的值
   System.out.println(Optional.ofNullable(userDto).map(UserDto::getUserName).get()); //返回用户名
   System.out.println(Optional.ofNullable(userDto).map(u->u.getUserName()).get()); //返回用户名
   ```

6. ```java
   UserDto userDto = null;
   Optional.ofNullable(userDto).map(u->u.getUserName()).orElse("http://minwk.top/"); //userDto对象为空时返回默认值，u.getUserName()为空时也会返回默认值，相当之简洁
   
   Map<Integer, CommodityAreaInfo> commodityInfoMap = null;
   System.out.println(Optional.ofNullable(commodityInfoMap)
                      //取commodityInfoMap中key=1的对象
   				.map(c->c.get(1))
                      //取commodityInfoMap中key=1的对象，再取该对象的getSellingPrice
   				.map(CommodityAreaInfo::getSellingPrice).orElse(BigDecimal.TEN));
   ```

目前就用到这么多，等用到新的再补充！

