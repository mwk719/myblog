---
layout: post
title: 你还不知道吗？这里有最简单的使用Java进行文件加密
date: 2020-07-17
Author: minweikai
tags: [工具,好玩]
comments: true
---

### 前言

- [字节流运算实现文件的加密解密1.0](https://minwk.top/FileEncryptAndDecrypt1.0/)
- [字节流运算实现文件的加密解密2.0](https://minwk.top/FileEncryptAndDecrypt2.0/)

如果没看的话，可以去飞速的浏览一下，方便这篇博客的理解（我不会告诉你，我是让你帮我的博客增加点击量的）

好，相信你已经过去看了一下。

我是相信你的哦！

那么好，通过对前两篇博客的理解，我便可以开发出一个工具可实现对文件的深加密

这就是这篇博客[你还不知道吗？这里有最简单的使用Java进行文件加密](https://minwk.top/FileEncryptAndDecrypt3.0/)要说的可制作jar工具包用来对文件或文件夹进行加密解密，此处点题。

#### 好处

1. 可对文件进行深加密
2. 有效的防止别人偷窥你的文件
3. 加密原理开源，可对算法进行修改；理论上只要别人不知道你的加密原理，即便知道密码，也根本无法破解

简单的介绍下我做的这个加密工具的原理

### 加密原理

1. 先对文件或文件夹进行zip压缩

2. 对压缩后的文件执行加密方法，最后生成后缀为 .mwk的文件为加密文件

### 解密原理

1. 对后缀为 .mwk的加密文件进行解密，然后得到同名的压缩包文件

2. 对压缩包文件解压得到原文件

#### 部分代码如下

说明：依据上述原理进行的操作流程及所需要的参数
ps. 完整代码见文章结尾

```java
/**
 * 加密解密方法执行入口，必须传3个参数
 *
 * @param args [0] 操作类型：加密传 e ，解密传 d
 *             [1] 要操作文件的绝对路径
 *             [2] 加密或解密的密码，必须是数字
 */
public static void main(String[] args) {
   //args = new String[]{"e", "E:/test/阿里巴巴Java开发.pdf", "250"};
   //args = new String[]{"d", "E:/test/阿里巴巴Java开发.pdf.mwk", "250"};

   //参数校验
   check(args);
   if (!SUCCESS) {
      return;
   }

   System.out.println("执行已开始，请勿随意退出......");
   long start = System.currentTimeMillis();

   File src = new File(args[1]);
   File tempZip;
   String title;
   switch (args[0]) {
      //加密
      case E:
         //压缩
         tempZip = toZip(src);
         //加密
         encryptFile(tempZip);
         title = "加密";
         break;
      //解密
      case D:
         //解密
         tempZip = decryptFile(src);
         //解压
         unZip(tempZip);
         title = "解密";
         break;
      default:
         errTips(TIPS);
         return;
   }
```

总结及注意事项
- 代码中可能也有不完善的地方，欢迎各位留言指正
- 因为此代码只是我测试了一些文件加密没问题，不知道会不会有其他问题。所有仅供娱乐、技术研究使用，切勿对自己重要的文件进行加密测试哦
- 加密完后要牢记密码哦，否则忘记密码则无法解密。其实也可以破解，就是将密码从-128试到127，总共试256次总有一个是对的。这是因为我的加密算法是对byte进行计算的，而byte值的范围只有-128~127
- 这里讨论的不是说加密算法，而是加密方式。别人不知道你的加密方式，即便知道密码也无法破解，所以说还是要对症下药。你可以对整体的字节流或者部分字节流加密，这也是一种方式

资源路径：

- [https://gitee.com/mwk719-见完整代码](https://gitee.com/mwk719/spring-learn/blob/master/src/main/java/com/mwk/encrypt/FileEncryptAndDecryptSalt_3.java)

- [jar包下载](https://gitee.com/mwk719/spring-learn/releases/addpwd-1.0.jar)