---
layout: post
title: logstash配合spring boot-logback项目实时传输日志
date: 2019-07-05
Author: minweikai
tags: [工具]
comments: true
---

#        在此博客“[Ubuntu16 ELK+Filebeat日志管理平台搭建-笔记](https://blog.csdn.net/weixin_41187876/article/details/86694720)”基础上继续深入学习；

1. 首先配置logstash-6.5.4中的logstash.conf文件

   ```bash
   input {
     tcp {
         port=> 8065
         codec => "json"
     }
   }
   
   output {
     elasticsearch {
       hosts => "129.0.0.1:9200"
       index => "%{[appname]}-%{+YYYY.MM.dd}"
     }
     stdout {
        codec => rubydebug { }
     }
   }
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.  启动logstash

   ```bash
   #启动logstash    
   ./bin/logstash -f logstash.conf &
   ```

   ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3.  配置简单的spring-boot

   1. 

      在pom文件中添加依赖



      ```java
      <dependency>
      	<groupId>net.logstash.logback</groupId>
      	<artifactId>logstash-logback-encoder</artifactId>
      	<version>4.11</version>
      </dependency>
      ```

      ![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   2. 配置logback-spring.xml   

​               ![img](https://img-blog.csdnimg.cn/20190131112901731.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​              \3. logback-spring.xml配置

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} -
				%msg%n</pattern>
		</encoder>
	</appender>

	<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径 -->
	<property name="LOG_HOME" value="/home" />


	<appender name="eventFile"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- daily rollover 保存历史记录到这个文件夹一日起为后缀 -->
			<FileNamePattern>${LOG_HOME}/log/%d{yyyy-MM-dd}.log</FileNamePattern>
			<!-- keep 30 days' worth of history -->
			<maxHistory>30</maxHistory>
		</rollingPolicy>
		<encoder>
			<Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{35} -
				%msg %n</Pattern>
			<charset>UTF-8</charset> <!-- 此处设置字符集 -->
		</encoder>
	</appender>

	<appender name="LOGSTASH"
		class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<destination>127.0.0.1:8065</destination>
		<!-- encoder必须配置,有多种可选 -->
		<encoder charset="UTF-8"
			class="net.logstash.logback.encoder.LogstashEncoder">
			<customFields>{"appname":"guilin"}</customFields>
		</encoder>
	</appender>

    
	<springProfile name="dev">
		<!-- 打印 日志级别 -->
		<root level="info">
			<appender-ref ref="eventFile" />
			<appender-ref ref="STDOUT" />
		</root>
	</springProfile>

	<springProfile name="prod">
		<!-- 打印 日志级别 -->
		<root level="info">
			<appender-ref ref="eventFile" />
			<appender-ref ref="STDOUT" />
			<appender-ref ref="LOGSTASH" />
		</root>
	</springProfile>
</configuration>
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意：

****配置该项<springProfile name="dev">，则可以在application.yml中配置线上与线下环境日志的输出；

dev本地运行，不传输日志到logstash;

prod线上环境，实时传输日志到logstash；

****配置<customFields>{"appname":"test"}</customFields>，则会在logstash.conf中对数据json化，读取字段appname，在kibana界面的index pattern处会以该名称显示。则可以配置不同项目的appname，达到对多个项目的日志进行查找；

 \4. application.yml配置如下启动项目

```
spring:
  profiles:
    active:
    - prod
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\5.  在kibana界面的management

![img](https://img-blog.csdnimg.cn/2019013111442491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)