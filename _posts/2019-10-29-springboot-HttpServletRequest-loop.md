---
layout: post
title: 踩坑-springboot中读取HttpServletRequest流的死循环
date: 2019-10-29
Author: minweikai
tags: [技巧,踩坑]
comments: true
---

## 踩坑记录  

在我们公司一个springmvc项目中有一个读取HttpServletRequest流的方法，我把这个方法用在新的springboot项目中结果陷入死循环。经过很久的测试，发现这个方法用在springboot项目有问题，现在将这个坑记录起来，方便自己，方便他人。

#### springmvc中HttpServletRequest读取流的方法，该方法在springboot中调用会陷入死循环

```java
String inputLine;
StringBuilder notifyXml = new StringBuilder();
 
try {
	while ((inputLine = request.getReader().readLine()) != null) {
		notifyXml.append(inputLine);
	}
		request.getReader().close();
	} catch (IOException e) {
			e.printStackTrace();
	}
```

#### springboot中建立缓冲流读取

```java
StringBuilder notifyXml = new StringBuilder();
  try {
	   BufferedReader in = new BufferedReader(new InputStreamReader(request.getInputStream()));
	   String line;
	   while ((line = in.readLine()) != null) {
			notifyXml.append(line);
		}
  } catch (IOException e) {
	 e.printStackTrace();
  }
```

