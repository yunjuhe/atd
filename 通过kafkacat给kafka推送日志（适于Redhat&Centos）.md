# 通过kafkacat给kafka推送日志（适于Redhat/Centos）

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
  
## 以下是通过kafkacat给kafka推送日志的操作文档（适于Redhat/Centos）：
> 对于通过kafkacat给kafka推送日志，ATD提供两种方式，一种是：一键推送；另一种是：手动推送。<br/>

# 方式一：一键推送


ATD提供了一键推送日志的脚本，在进行一键推送操作时，第一步是：获取脚本， 第二步是：运行脚本。

**1、获取脚本**

获取脚本的执行命令为：

```
wget http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/push_log.sh -O ./push_log.sh
```

**2、修改脚本中对kafka topic的指定**

执行以下命令：

```
sed -i 's/juhe-log/juhe-test/g' push_log.sh
```

> 说明：<br/>
> 其中juhe-test是变量，是日志格式对应的TopicName。

**3、运行脚本**

运行脚本的执行命令格式为：

```
bash pushlog.sh {ATD机器ip1,ATD机器ip2,...} {ATD机器hostname1,ATD机器hostname2,...} {日志服务器将要传送的日志1,日志服务器将要传送的日志2,...} 
```

> 说明：<br/>
> 1、在执行脚本之前，需要获取三个变量参数：ATD机器ip，ATD机器hostname，日志服务器将要传送的日志；<br/>
> 2、“日志服务器将要传送的日志”为将要推送给ATD进行分析的日志，这个日志的格式必须和该kafka topic的日志格式一致；<br/>
> 3、如果有多个ip、hostname和日志文件，可以在相应的参数位置以 , 分隔；<br/>
> 4、多个ip和hostname的情况时，先后顺序要一一对应；<br/>
> 5、所有操作均在webserver或日志服务器上进行；

该日志格式的一键推送脚本执行命令（示例）为：

```
bash push_log.sh 123.59.102.46,123.59.102.47 bgp-beijing-123-59-102-46,bgp-beijing-123-59-102-47 /var/log/nginx/*.log,/etc/log/*.log

```



# 方式二：手动推送

如下方法适用于centos6/7和redhat6/7系统。

> 注意：
请勿将不同格式的域名日志推到同一个Topic下，否则ATD将无法完成日志解析且不能进行威胁分析

## 1、添加yum repo并绑定hosts文件

### 1.1、添加yum repo

centos/redhat 6/7使用如下yum源：


```
# vim /etc/yum.repos.d/bsc-juhe.repo

[bsc-juhe]

name=juhe

baseurl=http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/bsc/el$releasever/$basearch/

gpgcheck=0

enabled=1


```

### 1.2、绑定hosts

绑定kafka server,例如我们用ambari安装了1台kafka server，对应的主机名和ip如下，那么就需要在nginx server上绑定这些hosts，在/etc/hosts文件中追加：

```
# vim /etc/hosts

10.199.204.94 bjs0-vq6-v

```
## 2、安装和配置kafka客户端

### 2.1、安装kafka客户端

```
# yum install bsc-kafkacat

```

验证kafka client和server端通信是否正常, 输出有相关topic信息：
```
# /usr/local/bsc/kafkacat/bin/kafkacat -L -b bjs0-vq6-v:6667

```
### 2.2、配置kafkacat收集日志

安装supervisor，用supervisor管理kafkacat进程:

```
# yum install supervisor
```

添加supervisor配置文件 `/etc/supervisord.d/kafkacat.ini` (其中/data/nginx_logs/xhw.json.log为web服务器的日志文件，需要您手动指定）：

```
[program:kafkacat]

command=sh -c "tail -F -q /data/nginx_logs/xhw.json.log|/usr/local/bsc/kafkacat/bin/kafkacat -b bjs0-vq6-v:6667 -t juhe-log;sleep 10"

autostart = true

startsecs = 5

autorestart=true

user=root

killasgroup=true

stopasgroup=true
```

>备注：CentOS 6和CentOS7的操作方式略有不同，以下提供这两种版本的操作方式。
>查看CentOS版本的方式为：# cat /etc/redhat-release

启动supervisor：

```
# /etc/init.d/supervisord start   #CentOs 6版本
# systemctl start supervisord     #CentOs 7版本
```
```
# /etc/init.d/supervisord status  #CentOs 6版本
# supervisorctl                   #CentOS 7版本
```
结果如下：

```
kafkacat          RUNNING    pid 17176, uptime 4 days, 16:30:51
#CentOS 6与CentOs 7下结果一致
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

```
（3）supervisor启动失败：
在`/etc/supervisord.conf`文件中查看是否有以下内容，如果没有，请添加。
```
[include]
files = supervisord.d/*.ini
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

