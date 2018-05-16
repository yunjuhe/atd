
# 通过rsyslog给kafka推送日志（适于Linux）

----
### 说明：
>  1、ATD支持多种给kafka推送日志的方式<br/>
> <br/>
> 针对Linux系统（redhat、centos）支持如下方式（按推荐顺序排列）：<br/>
> （1）[通过kafkacat给kafka推送日志][1]<br/>
> （2）[通过filebeat给kafka推送日志][2]<br/>
> （3）[通过rsyslog给kafka推送日志][3]<br/>
> （4）[通过logstash给kafka推送日志][4]<br/>
> <br/>
> 针对Windows系统支持如下方式：<br/>
> （1）[通过nxlog给kafka推送日志][5]<br/>
> <br/>
> 2、注意：您需要将日志推送到kafka相应的TopicName中，但是，请勿将不同格式的域名日志推到同一个TopicName下，否则ATD将无法完成日志解析。<br/>

## 以下是通过rsyslog给kafka推送日志的操作文档（适于Linux）：

## 1，添加需要的repo和绑定hosts
（1）下载rsyslog官方最新的repo，来安装rsyslog和rsyslog给kafka推送日志的插件rsyslog-kafka：
```
# cd /etc/yum.repos.d/
# wget http://rpms.adiscon.com/v8-stable/rsyslog.repo
```

（2）配置白山的yum repo(centos/redhat 6/7使用如下yum源)：


```
# vim /etc/yum.repos.d/bsc-juhe.repo

[bsc-juhe]

name=juhe

baseurl=http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/bsc/el$releasever/$basearch/

gpgcheck=0

enabled=1


```

（3）绑定hosts
绑定kafka server，在/etc/hosts文件中追加：
```
# vim /etc/hosts
192.168.0.89 bgp-beijing-beijing-1-123-59-102-46
```

## 2，安装和配置rsyslog
（1）安装rsyslog和给kafka推送日志的插件rsyslog-kafka
```
# yum install rsyslog rsyslog-kafka -y
```

（2）配置rsyslog来收集日志并把日志推送到kafka

确保/etc/rsyslog.conf配置中如下配置处于生效状态：
```
$IncludeConfig /etc/rsyslog.d/*.conf
```

添加/etc/rsyslog.d/bsc.conf配置文件，实现从本地读取日志文件发送给kafka server（其中/data0/logs/nginx_access.log为web服务器的日志文件，需要您手动指定)
```
module(load="imfile" mode="inotify")
input(type="imfile"
    File="/data0/logs/nginx_access.log"
    Tag="bsc"
    Facility="local1"
    Severity="info"
    freshStartTail="on"
)

module(load="omkafka")
$template l7_msg,"%msg:1:$%"
local1.info action(
                type="omkafka"
                broker="bgp-beijing-beijing-1-123-59-102-46:6667"
                topic="juhe-1710116uSh"
                partitions.number="32"
                confParam=[
                        "socket.keepalive.enable=true"
                        ]
                template="l7_msg"
                action.resumeretrycount="1"
)
```

（3）启动rsyslog
```
centos/redhat 6
# /etc/init.d/rsyslog restart

centos/redhat 7
# systemctl restart rsyslog
```

## 3、常见的问题及解决办法：
（1）确认是否有新产生的日志进入到kafka中：
登陆到ATD部署机器192.168.0.89，消费对应kafka的topic数据，如果日志源有新日志产生且推送日志流程正常，使用如下命令能看到日志：
```
# /usr/hdp/2.6.2.0-205/kafka/bin/kafka-console-consumer.sh --bootstrap-server $(hostname):6667 --topic juhe-1710116uSh
```
（2）如果（1）步骤中没有消费到日志，则自查如下：
```
查看推送日志的机器到kafka机器的网络是否连通：
# telnet 192.168.0.89 6667

查看rsyslog日志，看是否有相关报错
```
（3）supervisor启动失败：
在`/etc/supervisord.conf`文件中查看是否有以下内容，如果没有，请添加。
```
[include]
files = supervisord.d/*.ini
```




  [1]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87kafkacat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [2]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [3]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87rsyslog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [4]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87logstash%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [5]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87nxlog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EWindows%EF%BC%89.md
