# 通过filebeat给kafka推送日志（适于Redhat/Centos）

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

## 以下是通过filebeat给kafka推送日志的操作文档（适于Redhat/Centos）：

## 1.下载filebeat-5.6:

- 配置yum源，编辑`/etc/yum.repos.d/bsc-juhe.repo`文件：

```

[bsc-juhe]

name=juhe

baseurl=http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/bsc/el$releasever/$basearch/

gpgcheck=0

enabled=1


```

- 编辑完成后清除一下缓存:

        yum clean all
        
- 安装:

        yum install filebeat-5.6.4-1.x86_64

## 2.绑定kafka-server机器的hostname到ip的映射：
- 绑定kafka server,对应的主机名和ip如下，那么就需要在日志服务器上绑定这些hosts，在`/etc/hosts`文件中追加：

        vim /etc/hosts
```
{{ip}} {{hostname}}
```
- 例： 

        vim /etc/hosts
```
172.18.1.2 ATD1
172.18.1.3 ATD2
172.18.1.4 ATD3
```

## 3.编辑filebeat的配置文件:
- 编辑文件`/etc/filebeat/filebeat.yml`：

```
#=========================== Filebeat prospectors =============================

filebeat.prospectors:

# Each - is a prospector. Most options can be set at the prospector level, so
# you can use different prospectors for various configurations.
# Below are the prospector specific configurations.

- input_type: log

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /tmp/1.log
    - /tmp/2.log
    - /tmp/*.log.b

#================================ Outputs =====================================
output.kafka:
    #enabled: true
    hosts: ["kafka1:6667", “kafka2:6667”, “kafka3:6667”]
    version: "0.10.1"
    topic: "juhe-test"
    partition.round_robin:
        reachable_only: false
    required_acks: 1
    compression: gzip
    max_message_bytes: 1000000
    codec.format:
        string: '%{[message]}'
```

- 注意：<br/>
1）paths为本地文件的路径，支持多个文件的导入，支持通配符，支持文件句柄释放后新的同名文件的句柄引入。<br/>
2）version为目标kafka-server的版本，ATD机器默认的kafka-server版本为0.10.0 。<br/>
3）hosts为kafka-server的地址，格式为"ATD机器hostname:kafk-server的端口号"。<br/>
4）topic为kafka-server接受相应日志的topic。<br/>

- 完成filebeat配置文件的编辑操作后，重启filebeat：
<br/><br/>
centos6:

        /etc/init.d/filebeat restart
centos7:

        systemctl restart filebeat

## 4、常见的问题及解决办法：
（1）确认是否有新产生的日志进入到kafka中：
登陆到ATD部署机器192.168.0.89，消费对应kafka的topic数据，如果日志源有新日志产生且推送日志流程正常，使用如下命令能看到日志：
```
# /usr/hdp/2.6.2.0-205/kafka/bin/kafka-console-consumer.sh --bootstrap-server $(hostname):6667 --topic juhe-1710116uSh
```
（2）如果（1）步骤中没有消费到日志，则自查如下：
```
查看推送日志的机器到kafka机器的网络是否连通：
# telnet 192.168.0.89 6667

查看filebeat日志，看是否有相关报错
```
（3）supervisor启动失败：
在`/etc/supervisord.conf`文件中查看是否有以下内容，如果没有，请添加。
```
[include]
files = supervisord.d/*.ini
```

## 快速链接

1、参考官方文档：<br/>
https://www.elastic.co/guide/en/beats/filebeat/master/configuring-howto-filebeat.html

2、安装拦截器文档：<br/>
（1）安装拦截器文档（适于Redhat/Centos）：
https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos).md
<br/>
（2）安装拦截器文档（适于Ubuntu）：https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8EUbuntu).md


[1]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87kafkacat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[2]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[3]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87rsyslog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[4]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87logstash%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos%EF%BC%89.md
[5]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87nxlog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EWindows%EF%BC%89.md
[6]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EUbuntu%EF%BC%89.md



