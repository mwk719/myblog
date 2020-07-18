---
layout: post
title: 字节流运算实现文件的加密解密1.0
date: 2020-06-24
Author: minweikai
tags: [工具,好玩]
comments: true
---

- [x] [字节流运算实现文件的加密解密1.0](https://minwk.top/FileEncryptAndDecrypt1.0/) 实现对从文件字节流的操作、复制文件，到字节运算实现文件的加密解密
- [x] [字节流运算实现文件的加密解密2.0](https://minwk.top/FileEncryptAndDecrypt2.0/) 优化文件字节流的操作、探讨字节运算问题
- [x] [你还不知道吗？这里有最简单的使用Java进行文件加密](https://minwk.top/FileEncryptAndDecrypt3.0/) 实现文件加密解密的实战应用，可制作为jar包用于日常文件与文件夹的加密解密

### 简介

- byte：Java中基本类型之一，值域为-128~127

- 在Java中所有文件都可以使用IO以字节流的方式进行读写；

- 常见使用：文件复制

### 原理

- #### 加密

  获取文件的字节码数组然后对其进行加盐运算，将运算后的字节码信息生成新的文件。因为文件的字节信息已被修改，所以生成的文件跟原来的文件已经相差很远，可称其为加密文件。

- #### 解密

  读取加密文件中的字节信息，进行去盐运算，可以得出原文件的字节信息，然后输出为解密文件=原文件。

### 字节流读取与写入

使用FileInputStream读取文件的字节数组

```java
//实现代码
public static void main(String[] args) {
   //普通文本文件
   File file = new File("E:\\test\\1.txt");
   System.out.println(Arrays.toString(readBytes(file)));
}

static byte[] readBytes(File file) {
   long len = file.length();
   byte[] bytes = new byte[(int) len];
   try (FileInputStream in = new FileInputStream(file)) {
      int readLength = in.read(bytes);
      if ((long) readLength < len) {
         throw new IOException("文件读取失败");
      }
   } catch (IOException e) {
      e.printStackTrace();
   }
   return bytes;
}
```

使用FileOutputStream根据字节数组生成文件，结合以上的读取字节数组方法就可以实现文件的复制

```java
//实现代码
public static void main(String[] args) {
		//原文件
		File src = new File("E:\\test\\1.txt");
		//目标文件
		File dest = new File("E:\\test\\2.txt");
		//原文件的字节数组
		byte[] srcBytes = readBytes(src);
		System.out.println("原文件字节数组：" + Arrays.toString(srcBytes));
		//根据原字节数组生成目标文件，实现文件的复制
		writeBytes(srcBytes, dest);
	}

	/**
	 * 根据字节流生成文件
	 *
	 * @param data
	 * @param dest
	 */
	static void writeBytes(byte[] data, File dest) {
		try (FileOutputStream out = new FileOutputStream(dest)) {
			out.write(data);
			out.flush();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	static byte[] readBytes(File file) {
		long len = file.length();
		byte[] bytes = new byte[(int) len];
		try (FileInputStream in = new FileInputStream(file)) {
			int readLength = in.read(bytes);
			if ((long) readLength < len) {
				throw new IOException("文件读取失败");
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		return bytes;
	}
```

当然如果你只是要实现文件的复制，这只是其实现方式之一，还有：

1. BufferedInputStream/BufferedOutputStream缓冲字节流进行复制文件，使用该方式可对以上方法进行优化；
2. FileReader/FileWriter字符流进行文件复制，顾名思义只能针对字符文件进行操作；
3. BufferedReader/BufferedWriter缓冲字符流进行文件复制，顾名思义只能针对字符文件进行操作。

### Bingo

知道了字节流读取与写入的方法后，结合上述加解密原理，相信你已经知道后面要怎么做了。

你猜的没错，就是对readBytes方法获取的字节数组进行加盐运算。

比如：

- 加密就是对字节数组进行每一个字节加1，获取到新的字节数组；

- 解密就是对字节数组进行每一个字节减1，获取到原来的字节数组。

当然这里的加盐运算也可以有其他的运算规则：

1. 对字节数组部分字节进行加盐运算；
2. 倒序排列字节数组；
3. 字节数组进行AES加密解密运算；

这个运算规则可玩性很高，生成的加密文件，如果别人不知道加密方式，应该是无法解密的，**你可能会说这个世界上只要是双向加密的信息就没有不能破解的**。嗯！这句话我认同。当然加密方式越复杂，解密时间只是指数增长了。

代码稍多，请移步这个地址：https://gitee.com/mwk719/spring-learn/blob/master/src/main/java/com/mwk/encrypt/FileEncryptAndDecryptSalt_1.java

### 问题

因为读取文件方法使用的是FileInputStream获取文件字节码流的方式，所以不建议特别大的文件；

在文件读取过程中会获取文件的字节数组存在内存中，文件过大获取到的数组也会很大，容易造成内存溢出，而且读取字节数组的时间也会增长。

### 思考

因为字节的范围为-128~127，在运算过程中肯定有超出范围的数字127+1=128。为什么生成加密文件未出错，而且被加密的文件还可以正常解密？？