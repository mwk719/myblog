---
layout: post
title: Ubuntu16 ELK+Filebeat日志管理平台搭建
date: 2019-01-31
Author: minweikai
tags: [服务器,工具]
comments: true
---

# 1 系统环境

- Ubuntu 16.04 LTS

- elasticsearch-6.5.4.tar.gz

- logstash-6.5.4.tar.gz

- kibana-6.5.4-linux-x86_64.tar.gz

- filebeat-6.5.4-linux-x86_64.tar.gz


  工具网址：<https://www.elastic.co/downloads>

# 2 ELK介绍

#    Elasticsearch：是一个分布式、可扩展、实时的搜索与数据分析引擎，具有高可伸缩、高可靠等特点。基于全文搜索引擎库Apache Lucene基础之上，能对大容量的数据进行接近实时的存储、搜索和分析操作。

Logstash：数据处理引擎，能够同时从多个来源采集数据，通过过滤器解析、转换数据，最后输出到存储库Elasticsearch。

Kibana：数据分析与可视化平台，结合Elasticsearch使用，对数据进行汇总、搜索和分析，利用图表、报表、地图组件对数据进行可视化分析。

# 3 搭建过程

## 3.1 安装ElasticSearch

\1. 解压elasticsearch tar包

```bash
tar -zxvf elasticsearch-6.5.4.tar.gz
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

\2.  修改配置config/elasticsearch.yml，在es目录下创建data和logs文件夹

```html
path.data: /opt/apps/elk/elasticsearch-6.5.4/data
path.logs: /opt/apps/elk/elasticsearch-6.5.4/logs
network.host: 0.0.0.0
network.port: 9200
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 network.host: 0.0.0.0  测试，可以添加自己的IP以限制访问；

3.启动elasticsearch服务

```bash
./elasticsearch &
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

出现异常提示：不能以[root](https://www.baidu.com/s?wd=root&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)用户运行elasticsearch

```java
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

原因：由于elasticsearch可以接收用户输入的脚本并且执行，所以系统为了安全考虑设置了条件。

解决：root用户下创建新用户用于测试elasticsearch

```bash
# 创建elk用户组
groupadd elk
# 创建elk用户并指定用户组elk
useradd elk -g elk
passwd elk
#输入两次密码 回到上级目录并更改elasticsearch的拥有者：
# 修改elasticsearch-6.1.2文件夹拥有者和所属群组
chown -R elk:elk elasticsearch-6.5.4
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这时切换到elk用户，执行

```bash
su elk
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

切换至elk用户，再次启动 elasticsearch，出现以下错误提示

```
ERROR: bootstrap checks failed
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

原因：elasticsearch默认使用混合mmapfs/niofs目录来存储其索引，对mmap计数的默认操作系统限制可能过低，可能导致内存不足异常。

解决：切换root用户，修改内核参数vm.max_map_count



```bash
# 内核参数配置文件/etc/sysctl.conf添加配置
vm.max_map_count = 262144
# 使配置生效
sysctl -p
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

切换回elk用户，重新启动elasticsearch

```bash
#启动elasticsearch
./bin/elasticsearch -d #-d为后台启动
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.验证是否启动成功

curl 'http://1270.0.1:9200'

![img](https://img-blog.csdnimg.cn/20190129202450556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

返回以上内容说明启动成功。

## 3.2 安装Logstash

1.解压logstash tar包

```bash
tar -zxvf logstash-6.5.4.tar.gz
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Logstash Pipeline有两个必须的部分，输入和输出，以及可选部分过滤器。输入插件消费某个来源的数据，过滤器插件根据设置处理数据，输出插件将数据写到目标存储库。 
![Logstash Pipeline](https://img-blog.csdn.net/2018040716281623?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xhcmF2ZWxzaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​



2.运行logstash最基础的pipeline

```bash
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

-e：可以直接用命令行配置，无需使用文件配置。当前pipeline从标准输入获取数据stdin，并把结构化数据输出到标准输出stdout。

Pipeline启动之后，控制台输入hello world，可看到对应输出。

3.安装logstash-input-beats插件

```bash
./bin/logstash-plugin install logstash-input-beats
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.创建pipeline配置文件logstash.conf，配置端口5044监听beats的连接，并创建elasticsearch索引。

```bash
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => "127.0.0.1:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

5.启动logstash服务

```bash
./bin/logstash -f logstash.conf &
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输入exit退出，logstash 还在后台运行

## 3.3 安装Kibana

1.解压kibana tar包

```bash
tar -zxvf kibana-6.5.4-linux-x86_64.tar.gz
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.修改kibana配置

```bash
#访问kibana的端口
server.port: 5601
# kibana地址
server.host: "127.0.0.1"
# elasticsearch地址
elasticsearch.url: "http://127.0.0.1:9200"
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3.启动kibana服务

```bash
./bin/kibana &
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.浏览器访问确认服务启动、

[http://X.X.X.X:5601](http://x.x.x.x:5601/) (修改成你所用内网或公网地址)

![img](https://img-blog.csdnimg.cn/20190129204448363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTE4Nzg3Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 3.4 安装Filebeat

1.解压filebeat tar包

```bash
tar -zxvf filebeat-6.5.4-linux-x86_64.tar.gz
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2.配置filebeat

我们这里输出到logstash，需要添加logstash信息，注释elasticsearch信息

```bash
filebeat.prospectors:
- type: log
  enabled: true
  paths:
- /opt/apps/elk/*.log
#output.elasticsearch:
  #hosts: ["localhost:9200"]
output.logstash:
  hosts: ["127.0.0.1:5044"]
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

3.测试配置

```bash
./filebeat -configtest -e
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.导出索引模板配置文件

```bash
./filebeat export template > filebeat.template.json
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

5.手动安装模板

curl -XPUT -H 'Content-Type: application/json' http://127.0.0.1:9200/_template/filebeat-6.1.2 -d@filebeat.template.json

6.如果已经使用filebeat将数据索引到elasticsearch中，则索引可能包含旧文档。加载 Index Pattern 后，您可以从filebeat- * 中删除旧文档，以强制 Kibana 查看最新的文档

```bash
curl -XDELETE 'http://127.0.0.1:9200/filebeat-*'
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

7.启动filebeat

```bash
./filebeat -e -c filebeat.yml -d "publish" &
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 4 Kibana简单使用

1.访问kibana地址([http://X.X.X.X:5601](http://x.x.x.x:5601/)）

2.Discover模块默认是没有Index Pattern，需要在 Management 模块中创建一个，这里我们创建一个 filebeat-* 的Index Pattern。 
![kibana02](https://img-blog.csdn.net/20180407163757889?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xhcmF2ZWxzaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)​



3.返回Discover模块，如下说明添加成功。 

![kibana04](https://img-blog.csdn.net/2018040716385011?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xhcmF2ZWxzaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4.在[搜索框](https://www.baidu.com/s?wd=%E6%90%9C%E7%B4%A2%E6%A1%86&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)输入关键字可在当前选择的Index Pattern基础上进行筛选。

![kibana04](https://img-blog.csdn.net/2018040716385011?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xhcmF2ZWxzaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 5. 参考

基本是按照这位仁兄的博客搭建的，遇到问题搜一搜，基本都能解决

### <https://blog.csdn.net/laravelshao/article/details/79842827>