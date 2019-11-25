---
layout: post
title: windowns中使用sh命令对springmvc项目进行不包含依赖打包
date: 2019-11-23
Author: minweikai
tags: [技巧]
comments: true
---

### 遇到的问题

1. 公司有一些springmvc项目，每次修改问题都要打成war包，而依赖的好多jar包又不好剔除，很麻烦；
2. 之后每次部署项目时，不部署war，而是将target编译的文件中依赖删除，压缩该文件一般在1m左右，然后上传到服务器解压部署。
3. 但这样还是很麻烦，每次编译后都要手动删除依赖、手动压缩、手动上传。

### 使用sh脚本完成打包操作

1. 安装git
2. 在git中安装zip【[参考链接](https://blog.csdn.net/xiaomihn/article/details/102985745)】

### 打包命令

```bash
#项目名称后缀
app=test

#切换目录至项目的target目录
cd /e/my_project/haj/dev/mwk-${app}/target

#手动删除依赖
rm -rf mwk-${app}/WEB-INF/lib/

#手动删除已存在文件
rm -rf mwk-${app}.zip

#压缩文件
zip -r mwk-${app}.zip mwk-${app}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我的项目名称是：mwk-test，名称可根据自己项目名称修改

**这样打完包后就手动上传zip包到服务器，然后解压部署就可以了**

### “如何使用Cloud Toolkit结合bash脚本实现自动化打包、部署、启动，待续。。。”

