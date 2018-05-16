# 通过filebeat给kafka推送日志（适于Ubuntu）

---
  ### 说明：
>  1、ATD支持多种给kafka推送日志的方式<br/>
> <br/>
> 针对Redhat、Centos支持如下方式（按推荐顺序排列）：<br/>
> （1）[通过kafkacat给kafka推送日志][1]<br/>
> （2）[通过filebeat给kafka推送日志][2]<br/>
> （3）[通过rsyslog给kafka推送日志][3]<br/>
> （4）[通过logstash给kafka推送日志][4]<br/>
> <br/>
> 针对Ubuntu支持如下方式：<br/>
> （1）[通过filebeat给kafka推送日志][6]<br/>
> <br/>
> 针对Windows系统支持如下方式：<br/>
> （1）[通过nxlog给kafka推送日志][5]<br/>
> <br/>
> 2、注意：您需要将日志推送到kafka相应的TopicName中，但是，请勿将不同格式的域名日志推到同一个TopicName下，否则ATD将无法完成日志解析。<br/>

## 以下是通过filebeat给kafka推送日志的操作文档（适于Ubuntu）：

## 1、准备工作：请确保推送日志的机器到ATD部署机器网络连通
```
# telnet 172.16.16.3 6667
Trying 172.16.16.3...
Connected to 172.16.16.3.
Escape character is '^]'.
```
绑定hosts：
```
# vim /etc/hosts
```
添加如下内容：
```
172.16.16.3 atd-172-16-16-3
```
## 2、安装filebeat
```
# curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.2.4-amd64.deb
# sudo dpkg -i filebeat-6.2.4-amd64.deb
```
## 3、配置filebeat（/tmp/1.log应修改为要推送日志的路径）
 编辑`/etc/filebeat/filebeat.yml`文件：

```
#=========================== Filebeat prospectors =============================

filebeat.prospectors:

# Each - is a prospector. Most options can be set at the prospector level, so
# you can use different prospectors for various configurations.
# Below are the prospector specific configurations.

- input_type: log
  # Paths that should be crawled and fetched. Glob based paths.
  ignore_older: 1m
  paths:
    - /tmp/1.log

#================================ Outputs =====================================
output.kafka:
    #enabled: true
    hosts: ["atd-172-16-16-3:6667"]
    version: "0.10.1"
    topic: "juhe-180419YMCk"
    partition.round_robin:
        reachable_only: false
    required_acks: 1
    compression: gzip
    max_message_bytes: 1000000
```

## 4、启动filebeat
```
# /etc/init.d/filebeat start
```

## 快速链接

安装拦截器文档：<br/>
（1）安装拦截器文档（适于Redhat/Centos）：https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos).md
<br/>
（2）安装拦截器文档（适于Ubuntu）：https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8EUbuntu).md



[1]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87kafkacat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[2]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[3]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87rsyslog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[4]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87logstash%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[5]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87nxlog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EWindows%EF%BC%89.md
[6]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EUbuntu%EF%BC%89.md

