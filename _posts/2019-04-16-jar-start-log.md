---
layout: post
title: jar后台启动并打印日志脚本
date: 2019-04-16
Author: minweikai
tags: [服务器,工具]
comments: true
---

```bash
#!/bin/bash
HOME="/home/gas/server/"   #项目路径
PROJECT="guilin-gas-1.0"   #项目名称
ACTIVE=dev                 #运行环境，生产环境
RUNNAME=$HOME$PROJECT".jar"  #项目运行名称
LOG=$HOME"log/"$PROJECT    #日志名称与路径


#kill All LISTEN
#判断端口，杀死项目
PORT=8077
echo "change dir: `pwd`" 
PID=`lsof -i:${PORT} | grep LISTEN | awk '{print $2}'`
echo "PID : ${PID}"
kill -9 ${PID}
#ps -ef | grep $PROJECT | grep -v grep | awk '{print $2}' | xargs kill

if [ ! -n "$PROJECT" ]; then
  echo "PROJECT IS NULL"
    exit 2
fi

if [ ! -n "$ACTIVE" ]; then
  nohup java  -jar $RUNNAME > $LOG.out 2>> $LOG.err &
else
  nohup java  -jar $RUNNAME > $LOG.out 2>> $LOG.err &
fi

#控制台打印同步日志
#tail -f $PROJECT.out
```