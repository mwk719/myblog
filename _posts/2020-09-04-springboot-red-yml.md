---
layout: post
title: application.yml变成红色图标如何解决
date: 2020-09-04
Author: minweikai
tags: [工具]
comments: true
---

### 前言

记录一下我遇到的application.yml变成红色图标解决方法，我使用的是idea的解决方法，希望可以帮到其它遇到这个问题的人

### 现状

![image-20200904163110630](http://qn.minwk.top/img/image-20200904163110630.png)

### 正常

![image-20200904163307402](http://qn.minwk.top/img/image-20200904163307402.png)

### 解决

1. 选中图标变成红色的项目
2. 在idea中快捷键 ctrl+shift+alt+s 打开Project Structure
3. ![image-20200904163834031](http://qn.minwk.top/img/image-20200904163834031.png)
4. 如上图可以发现正常服务有个**绿色spring**。**如果你的红色图标项目是少这个绿色spring**的话就可以用这里的方法解决
5. 如下操作![image-20200904164907453](http://qn.minwk.top/img/image-20200904164907453.png)
6. add 绿色spring![image-20200904164530049](http://qn.minwk.top/img/image-20200904164530049.png)
7. 看到这些最后apply就好了，变绿了![image-20200904164657465](http://qn.minwk.top/img/image-20200904164657465.png)

如此就大功告成