---
layout: post
title: 如何快速定位OutOfMemoryError发生的地方和原因
date: 2020-06-19
Author: minweikai
tags: [问题,踩坑]
comments: true
---

### 问题

相信你做的项目在日常运行中肯定遇到过OOM，内存不足。什么？你没遇到过！那你也得看看，先了解下，以后肯定会遇到的，相信我！

PS. 以下可能会出现一些专业名称，你了解的话还好，不了解的话我也不过多解释，自己百度吧，只说怎么找问题。

![image-20200619112703492](http://qn.minwk.top/img/image-20200619112703492.png)

#### 我遇到的场景：

1. 慢SQL：在业务处理中前一步查到的数据交给下一步处理，而下一步又查询的很慢。导致大量线程堆积在内存中等待执行，然后大量的数据等待下一步处理无法释放，最终结果就内存溢出了。
2. 阿里OSSClient没有关闭：项目中用到了阿里oss存储照片，每次上传完图片图片后没有及时shutdown，OSSClient连接存在内存中未关，占用大量内存。

#### 常见解决方案

1. 重启项目

2. 增大项目启动的内存设置

   《阿里巴巴开发手册》的建议：**[我收集的学习资料](https://gitee.com/mwk719/learning_materials)**

   ![image-20200619150051352](http://qn.minwk.top/img/image-20200619150051352.png)

但这些都是治标不治本，项目运行久了还是会出现的；

本着大多数问题都是因为代码没写好引起的理念，所以问题应该从代码着手。但具体是哪个方法、哪一行代码如果日志里分析不出来的话，那么我这篇博客就派上用场了。

### 诊断流程

1. ##### 项目运行中发生OOM时，需要导出dump文件用来分析问题，有两种方式：

   1. 执行此命令：jmap -dump:format=b,file=oom.dump pid  ，会生成当前pid进程对应的内存镜像文件oom.dump

   2. 或者在启动命令里配置启动参数，下次出现OOM时会自动生成oom.dump

      ```bash
      -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/log/oom.dump
      ```
      
      ![image-20200810155050931](http://qn.minwk.top/img/image-20200810155050931.png)

2. ##### Windows环境 - 对已导出的dump文件进行分析

   说明：可使用jdk自带的jvisualvm.exe进行分析，jvisualvm.exe在你自己装的jdk/bin/jvisualvm.exe目录下

   1. 打开jvisualvm.exe，选择文件-装入，选择文件类型堆 Dump(*.hprof,**)，就可以装入你的文件了

   2. 装入后可以看到好多信息

      ![image-20200619120831992](http://qn.minwk.top/img/image-20200619120831992.png)

      ![image-20200619121046019](http://qn.minwk.top/img/image-20200619121046019.png)

      ![image-20200619121310897](http://qn.minwk.top/img/image-20200619121310897.png)

由上面可以看出数组占用最大，初步怀疑可能是它占用内存没有即时释放

![image-20200619121715496](http://qn.minwk.top/img/image-20200619121715496.png)

​	果然是一直往list中存放数组，不能释放，结果就OOM了。这是我设置的启动参数-Xms128m -Xmx256m，所以就可以	试出来。

3. ##### Linux环境 - 对已导出的dump文件进行分析

   说明：使用Mat工具；因为导出的dump文件往往都是好几G，最好的话就在Linux上进行分析了

   1. 下载链接https://www.eclipse.org/mat/downloads.php，选择合适的包就可以，并解压到Linux里

   2. MAT分析 dump，该命令会执行一段时间

      ```bash
      ./ParseHeapDump.sh oom.dump  org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components
      
      #使用该命令如果报MAT内存不足的异常：
      #修改MemoryAnalyzer.ini 的 -Xmx，默认是1024m，大小比dump文件大就可以。
      ```

      

   3. 结果会生成一些文件，找到这三个后缀的压缩文件，比较小，可下载到本地解压用浏览器查看index.html

      ![image-20200619141708455](http://qn.minwk.top/img/image-20200619141708455.png)

      ![image-20200619142044838](http://qn.minwk.top/img/image-20200619142044838.png)

      ![image-20200619142314134](http://qn.minwk.top/img/image-20200619142314134.png)

### 发生

看下源码，java.lang.OutOfMemoryError类为什么会抛出这个错误

![image-20200619145348337](http://qn.minwk.top/img/image-20200619145348337.png)

如何避免发生OOM，网上资料很多，我就不赘述了。

### 步骤总结

```bash
1. #下载Mat工具，并解压
https://www.eclipse.org/mat/downloads.php

2. #导出当时内存镜像
jmap -dump:format=b,file=oom.dump pid

3. #使用MAT文件中ParseHeapDump.sh脚本分析 dump，该命令测试环境会执行30秒左右，正式环境估计会执行1-2分钟
./ParseHeapDump.sh oom.dump  org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components

4. #下载生成的三个压缩包就可以分析了
```

