---
layout: post
title: Stream-流式处理
date: 2020-03-19
Author: minweikai
tags: [技巧]
comments: true
---

## Java8的新特性

1. 对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。
2. 借助于同样新出现的Lambda表达式，极大的提高编程效率和程序可读性。
3. 提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用fork/join并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用Stream API无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。

## 使用技巧

1. 获取集合中某属性值的集合

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(1, "用户1"),
         new UserDto(2, "用户2"),
         new UserDto(3, "用户3")
   );
   List<Integer> userId = userDtos.stream().map(UserDto::getUserId).collect(Collectors.toList());
   System.out.println(userId);
   ```

2. 将集合转为map对象，以集合中某属性作为key，该属性值对应的对象做value；

   ```java
   //以userId作为key，key不能重复
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(1, "用户1"),
         new UserDto(2, "用户2"),
         new UserDto(3, "用户3")
   );
   //方式1
   Map<Integer, UserDto> userDtoMap = userDtos.stream()
         .collect(Collectors.toMap(UserDto::getUserId, Function.identity()));
   System.out.println(userDtoMap);
   
   //方式2
   Map<Integer, UserDto> userDtoMap2 = userDtos.stream()
         .collect(Collectors.toMap(UserDto::getUserId, UserDto->UserDto));
   System.out.println(userDtoMap2);
   ```

3. 以集合中某属性进行分组

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(1, 20),
         new UserDto(2, 21),
         new UserDto(3, 20)
   );
   Map<Integer, List<UserDto>> userMap = userDtos.stream()
         .collect(Collectors.groupingBy(UserDto::getAge));
   System.out.println(userMap);
   ```

4. 以集合中某属性进行排序

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(3, 22),
         new UserDto(2, 21),
         new UserDto(4, 21),
         new UserDto(1, 20)
   );
   List<UserDto> userDtoList = userDtos.stream()
       //.reversed()倒序，没有是正序
         .sorted(Comparator.comparing(UserDto::getUserId).reversed())
         .collect(Collectors.toList());
   System.out.println(userDtoList);
   ```

5. 以集合中某属性进行排序后再以另一个属性进行分组

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(3, 22),
         new UserDto(2, 21),
         new UserDto(4, 21),
         new UserDto(1, 20)
   );
   TreeMap<Integer, List<UserDto>> userDtoMap = userDtos.stream()
       //先以UserId进行排序
         .sorted(Comparator.comparing(UserDto::getUserId进行排序))
       //再以年龄进行分组
         .collect(Collectors.groupingBy(UserDto::getAge, TreeMap::new, Collectors.toList()));
   System.out.println("正序: " + userDtoMap);
   //以分组的key值倒序
   System.out.println("倒序" + userDtoMap.descendingMap());
   ```

6. 对集合中某数值属性进行求和、平均值、计数等

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(1, 20),
         new UserDto(2, 23),
         new UserDto(3, 21)
   );
   Integer age = userDtos.stream().mapToInt(UserDto::getAge).sum();
   System.out.println("总和="+age);
   
   double age = userDtos.stream().mapToInt(UserDto::getAge).average().getAsDouble();
   System.out.println("平均值="+age);
   //sum、average、count等属性，自己可以再探索
   ```

7. 对集合中某金额属性（BigDecimal）进行计算

   ```java
   List<UserDto> userDtos = Arrays.asList(
         new UserDto(1, BigDecimal.ONE),
         new UserDto(2, BigDecimal.TEN),
         new UserDto(3, BigDecimal.ZERO)
   );
   BigDecimal money = userDtos.stream().map(UserDto::getMoney).reduce(BigDecimal::add).get();
   System.out.println("总金额="+money);
   
   BigDecimal money = userDtos.stream().map(UserDto::getMoney).reduce(BigDecimal::max).get();
   System.out.println("集合中最大金额="+money);
   //add、max、divide、multiply等属性，自己可以再探索
   ```

   目前就用到这么多，等用到新的再补充！

## 资料

[Java的Stream流式处理](https://blog.csdn.net/qq_20989105/article/details/81234175)
[Java 8 Stream](https://www.runoob.com/java/java8-streams.html)

