
# 通过rsyslog给kafka推送日志（适于Linux）

----
### 说明：
>  1、ATD支持多种给kafka推送日志的方式<br/>
> <br/>
> 针对Linux系统支持如下方式（按推荐顺序排列）：<br/>
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

[bsc-juhe-salt]

name=juhe

baseurl=http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/salt/el$releasever/$basearch/

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
## 3、安装拦截器、salt和ipset
（1）安装拦截器扩展

```
$ yum install bsc-fw-agent-extension
```

添加supervisor配置文件`/etc/supervisord.d/reset.ini`：
```
[program:fw-agent]

command=/usr/local/bsc/bsc-fw-agent-extension/bin/fw-agent 0 cc

autostart = true

startsecs = 5

autorestart=true

user=root

killasgroup=true

stopasgroup=true
```
重启supervisor：

```
$ /etc/init.d/supervisord restart   #CentOS 6版本
$ systemctl start supervisord   #CentOS 7版本
```
查看kafkacat进程是否启动：

```
$ supervisorctl status
fw-agent            RUNNING    pid 17176, uptime 4 days, 16:30:51
```

（2）安装salt minion和ipset
```
$ yum install salt-minion
```

修改/etc/salt/minion配置文件，添加如下信息：
```
master: 192.168.0.89
id: ip
```
> 备注：id的value为本机内网ip即可

启动salt minion并查看运行状态
```
#CentOS 6版本上如下操作：

$ /etc/init.d/salt-minion start

$ /etc/init.d/salt-minion status

$ yum install ipset

#CentOS 7版本上如下操作：

$ systemctl start salt-minion

$ systemctl status salt-minion

$ yum install ipset
```
## 4，常见的问题及解决办法：
（1）如果安装的主机本身已经安装过salt-minion，可能已经有之前配置的认定salt-master，会存在/etc/salt/pki/minion/minion_master.pub导致无法认证新master的问题，可以选择移除或配置双master。

移除不需要的master：
```
# mv /etc/salt/pki/minion/minion_master.pub /etc/salt/pki/minion/minion_master.pub.b
```

（2）salt配置双master：

配置多Master时，Minion需要知道连接的每个Master的网络地址。需要在Minion的配置文件中进行配置，默认的配置文件是/etc/salt/minion。找到master的配置项，更新需要新增的Master，下面是一个多Master的配置例子：

```
master:
 - master1.example.tld
 - master2.example.tld
```

（3）确认是否有新产生的日志进入到kafka中：
登陆到ATD部署机器192.168.0.89，消费对应kafka的topic数据，如果日志源有新日志产生且推送日志流程正常，使用如下命令能看到日志：
```
# /usr/hdp/2.6.2.0-205/kafka/bin/kafka-console-consumer.sh --bootstrap-server $(hostname):6667 --topic juhe-1710116uSh
```
（4）如果（3）步骤中没有消费到日志，则自查如下：
```
查看推送日志的机器到kafka机器的网络是否连通：
# telnet 192.168.0.89 6667

查看rsyslog日志，看是否有相关报错
```



  [1]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87kafkacat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [2]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [3]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87rsyslog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [4]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87logstash%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [5]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87nxlog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EWindows%EF%BC%89.md
