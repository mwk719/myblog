---
layout: post
title: Linux服务命令
date: 2018-10-15
Author: minweikai
tags: [Linux,命令]
comments: true
---

# 常用命令

## Linux维护

------

```bash
tail -f catalina.out  #tomcat实时查看日志
tail -f catalina.out -n100 #实时查看日志后100行

lsof -i: 8080  #查看端口使用
netstat -tnlp | grep 8080

top  #运行的项目
top -c #运行的项目的完整信息
top -p pid #某线程的运行信息
#top 命令界面交互命令：
 m/M #内存占用排序
 p/P #cpu占用排序

#查询进程的启动时间和运行时间
ps -eo pid,lstart,etime | grep 5176

#打印堆信息， 包括 JVM版本、垃圾收集器、堆配置、堆各个区域使用情况
jmap -heap pid   

ps -aux | grep 服务名称 #查看服务CPU和内存的占用率，内存占用数据单位k
free -m  #cpu&内存的使用
df -h  #磁盘的使用

#Linux查看/var/log/wtmp文件查看可疑IP登陆
last -f /var/log/wtmp
#登陆用户名、时间、IP
who /var/log/wtmp
#寻找可疑IP登陆次数
vim /var/log/secure
#切换到root或某个用户名下
su - 用户名/root
输入  history #能看到这个用户历史命令，默认最近的1000条

#内存泄漏处理
1. 获取内存详情 jmap -dump:format=b,file=e.bin pid 
   这种方式可以用 jvisualvm.exe 进行内存分析，或者采用 Eclipse Memory Analysis Tools (MAT)这个工具
2. https://www.cnblogs.com/baihuitestsoftware/articles/6491344.html

```



------



## service命令

------

### nginx

停止nginx:
sudo systemctl stop nginx

启动nginx:
sudo systemctl start nginx

重启nginx:
sudo systemctl restart nginx

修改配置文件后，平滑加载配置命令(不会断开用户访问）：
sudo systemctl reload nginx

默认，nginx是随着系统启动的时候自动运行。如果你不想开机启动，那么你可以禁止nginx开机启动：
sudo systemctl disable nginx

重新配置nginx开机自动启动:
sudo systemctl enable nginx

网站文件位置
/var/www/html: 网站文件存放的地方, 默认只有我们上面看到nginx页面，可以通过改变nginx配置文件的方式来修改这个位置。
服务器配置
/etc/nginx: nginx配置文件目录。所有的nginx配置文件都在这里。
/etc/nginx/nginx.conf: Nginx的主配置文件. 可以修改他来改变nginx的全局配置。
/etc/nginx/sites-available/: 这个目录存储每一个网站的"server blocks"。nginx通常不会使用这些配置，除非它们陪连接到  sites-enabled 目录 (see below)。一般所有的server block 配置都在这个目录中设置，然后软连接到别的目录 。
/etc/nginx/sites-enabled/: 这个目录存储生效的 "server blocks" 配置. 通常,这个配置都是链接到 sites-available目录中的配置文件
/etc/nginx/snippets: 这个目录主要可以包含在其它nginx配置文件中的配置片段。重复的配置都可以重构为配置片段。
日志文件
/var/log/nginx/access.log: 每一个访问请求都会记录在这个文件中，除非你做了其它设置。
/var/log/nginx/error.log: 任何Nginx的错误信息都会记录到这个文件中。

mysql

1. 重启 service mysql restart

iptsbles

1. 重启 service iptables restart
2. 配置iptables路径  /etc/sysconfig/iptables