---
layout: post
title: 启动、停止、重启jar包并输出日志脚本
date: 2019-04-16
Author: minweikai
tags: [Linux,命令]
comments: true
---

### 1. 启动jar包并输出日志

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

### 2. 启动、停止、重启jar包并输出日志 - 2.0.0

```bash
#!/bin/sh

#项目名称
APP_NAME=hello
#项目路径
SERVICE_DIR=/home/$APP_NAME
#服务名称
SERVICE_NAME=$APP_NAME
JAR_NAME=$SERVICE_NAME\.jar
PID=$SERVICE_NAME\.pid
LOG=$SERVICE_DIR/log/$APP_NAME
ACTIVE=test
cd $SERVICE_DIR
case "$1" in

    start)
        nohup java -Xms512m -Xmx1024m -jar $JAR_NAME --spring.profiles.active=$ACTIVE > $LOG.out 2>> $LOG.err &
        echo $! > $SERVICE_DIR/$PID
        echo "=== start $SERVICE_NAME"
		tail -f $LOG.out
        ;;

    stop)
        kill `cat $SERVICE_DIR/$PID`
        rm -rf $SERVICE_DIR/$PID
        echo "=== stop $SERVICE_NAME"
        sleep 5
        P_ID=`ps -ef | grep -w "$JAR_NAME" | grep -v "grep" | awk '{print $2}'`
        if [ "$P_ID" == "" ]; then
            echo "=== $SERVICE_NAME process not exists or stop success"
        else
            echo "=== $SERVICE_NAME process pid is:$P_ID"
            echo "=== begin kill $SERVICE_NAME process, pid is:$P_ID"
            kill -9 $P_ID
        fi
        ;;

    restart)
        $0 stop
        sleep 2
        $0 start
        echo "=== restart $SERVICE_NAME success"
        ;;

    *)
        ## restart
        $0 stop
        sleep 2
        $0 start
        ;;

esac
exit 0

```

