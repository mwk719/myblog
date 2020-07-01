---
layout: post
title: 字节流运算实现文件的加密解密2.0
date: 2020-07-01
Author: minweikai
tags: [工具,好玩]
comments: true
---

### 前言

1. 在前一篇博客[《字节流运算实现文件的加密解密1.0》](https://minwk.top/FileEncryptAndDecrypt1.0/)里介绍了实现文件的加密解密的原理。因为读取文件方法使用的是FileInputStream对文件加密时效率较低，优化为使用BufferedOutputStream提升效率。

   

2. 遗留的思考：字节运算超出范围，但还是可以正常加密解密。

### 简述

使用BufferedInputStream提升读取效率

```java
/**
	 * 获取文件的byte数组
	 * 在{@link FileEncryptAndDecryptSalt_1#readBytes(File)}示例中使用FileInputStream字节流读取
	 * 执行read时，每次都从硬盘中读取文件字节。
	 * 在{@link FileEncryptAndDecryptSalt_2#readBytes(File)}示例中使用BufferedInputStream缓冲字节流读取
	 * 执行read时，每次都会先从缓冲区进行读取，默认缓冲区大小是8192字节。操作完缓冲区数据后，会从硬盘中获取
	 * 下一部分字节流放在缓冲区中。缓冲区即是内存，总所周知操作内存比操作硬盘效率要高得多。
	 * 所以一般情况下使用BufferedInputStream & BufferedOutputStream进行操作文件，因为效率最高
	 * 但是当你操作的文件小于8192字节时，read时直接读取缓存，使用BufferedInputStream效率最高
	 * 当文件字节越来越大时，使用FileInputStream的效率就趋近于使用BufferedInputStream
	 * <p>
	 * 文件过大时容易造成内存溢出，最好的方式还是边读边写，内存里不要留过多的数据
	 *
	 * @param file
	 * @return
	 * @see BufferedInputStream 缓冲输入流
	 * @see FileInputStream 缓冲流
	 */
	private static byte[] readBytes(File file) {
		try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file))) {
			byte[] flush = new byte[(int) file.length()];
			while (bis.read(flush) != -1) {
				return flush;
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		return new byte[0];
	}
```

字节运算

```java
/**
	 * 字节码数组加盐
	 * bytes[i] = (byte) (bytes[i] + SALT);
	 * 在含byte类型数据运算时，Java编译器会自动转换byte类型为int
	 * 这里加盐运算(bytes[i] + SALT)会转换为两个int类型相加，然后强转为byte接收
	 * 因为byte的取值范围为-128~127，可以将其类比为一个时钟：
	 * 从-128开始到127结束，然后又从-128开始
	 * 比如：(byte)(127+1) = -128
	 * 这样的话就可以理解加盐运算 {@link FileEncryptAndDecryptSalt_2#bytesAddSalt(byte[])}
	 * 后：(byte)(127+1) = -128，还是正常byte数据所以文件可以正常生成；
	 * 去盐运算{@link FileEncryptAndDecryptSalt_2#bytesSubSalt(byte[])}
	 * 后：(byte)(-128-1) = 127，恢复为原来的byte，这样就可以生成原有文件
	 *
	 * @param bytes
	 * @return
	 */
	private static byte[] bytesAddSalt(byte[] bytes) {
		for (int i = 0; i < bytes.length; i++) {
			bytes[i] = (byte) (bytes[i] + SALT);
		}
		return bytes;
	}

	/**
	 * 字节码数组去盐
	 *
	 * @param bytes
	 * @return
	 */
	private static byte[] bytesSubSalt(byte[] bytes) {
		for (int i = 0; i < bytes.length; i++) {
			bytes[i] = (byte) (bytes[i] - SALT);
		}
		return bytes;
	}
```

### 总结

以上两篇博客便是介绍操作文件流对文件进行加密解密的具体原理，使用该方法可以对文件本身实现深加密。

##### 那什么是电脑文件？

来自某度的解释：[电脑文件，也可以称之为计算机文件，是存储在某种长期储存设备或临时存储设备中的一段数据流，并且归属于计算机文件系统管理之下。](https://baike.baidu.com/item/电脑文件/21421152?fr=aladdin)

- 深加密：对文件数据流进行加密，本质上文件已经被修改，文件无法打开或打开乱码。

  难破解，可以对重要文件加密。

- 浅加密：不修改文件数据流，本质上文件并没有被加密。

  常见方式：修改文件名称使文件无法打开、修改、删除或替换文件图标为系统图标；隐藏文件或路径。 

  很容易被破解，可以防止菜鸟偷看。

我的实现只是简单的对文件进行数据流加密，可以用来学习，还可以再优化。

完整代码地址：https://gitee.com/mwk719/spring-learn/blob/master/src/main/java/com/mwk/encrypt/FileEncryptAndDecryptSalt_2.java