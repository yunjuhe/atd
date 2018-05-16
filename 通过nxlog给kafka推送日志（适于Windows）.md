
# 通过nxlog给kafka推送日志（适于Windows）
----
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
  

## 以下是通过nxlog给kafka推送日志的操作文档（适于Windows）：

## 1，windows下nxlog安装和配置
（1）下载nxlog，用来在windows下推送日志，此agent属于开源产品，广泛应用于windows系统<br/>
下载页面：https://nxlog.co/products/nxlog-community-edition/download<br/>
（2）在windows下双击如上下载的msi文件进行nxlog的安装<br/>
（3）配置nxlog<br/>
nxlog默认配置文件位置在：C:\Program Files (x86)\nxlog\conf，我们打开配置文件nxlog.conf,更改成如下配置（如果我们日志文件的目录为C:\test目录，请将目录改成自己的日志目录）：

```
###
define ROOT C:\Program Files (x86)\nxlog

Moduledir %ROOT%\modules
CacheDir %ROOT%\data
Pidfile %ROOT%\data\nxlog.pid
SpoolDir %ROOT%\data
LogFile %ROOT%\data\nxlog.log

<Extension _syslog>
    Module      xm_syslog
</Extension>

<Input in>
    Module im_file
    File "C:\\test\\\*.log"
    SavePos TRUE
    PollInterval 1
    ReadFromLast TRUE
    Exec   $SyslogFacility = "local1"; $SyslogSeverity = "ALERT";
</Input>

<Output out>
    Module      om_udp
    Host        192.168.0.89
    Port        514
    Exec to_syslog_bsd();
</Output>

<Route 1>
    Path        in => out
</Route>
```
（4）启动或者重启nxlog
右键我的电脑->管理->服务和应用程序->服务，然后查找名称为nxlog的服务，点击右键选择启动或者停止服务

## 2，常见的问题及解决办法：
（1）确认是否有新产生的日志进入到kafka中：
登陆到ATD部署机器192.168.0.89，消费对应kafka的topic数据，如果日志源有新日志产生且推送日志流程正常，使用如下命令能看到日志：

```
# /usr/hdp/2.6.2.0-205/kafka/bin/kafka-console-consumer.sh --bootstrap-server $(hostname):6667 --topic juhe-1710116uSh
```

（2）如果（1）步骤中没有消费到日志，则自查如下：

```
查看推送日志的机器到kafka机器的网络是否连通：
# telnet 192.168.0.89 6667

查看nxlog日志(默认位置在C:\Program Files (x86)\nxlog\data\nxlog.log)，看是否有相关报错
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

