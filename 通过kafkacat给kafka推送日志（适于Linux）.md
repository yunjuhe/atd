# 通过kafkacat给kafka推送日志（适于Linux）

---

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

## 以下是通过kafkacat给kafka推送日志的操作文档（适于Linux）：
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

[bsc-juhe-salt]

name=juhe

baseurl=http://bsc-juhe:6WY07AXDsI=@mirrors.juhe.baishancloud.com/repo/salt/el$releasever/$basearch/

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

## 3、安装拦截器、salt和ipset

安装拦截器扩展

```
$ yum install bsc-fw-agent-extension
```

添加supervisor配置文件`/etc/supervisord.d/reset.ini`：
```
[program:fw-agent]

command=/usr/local/bsc/fw-agent-extension/bin/fw-agent 0 cc

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
$ /etc/init.d/supervisord status   #CentOS 6版本
$ supervisorctl   #CentOS 7版本
```
```
kafkacat            RUNNING    pid 17176, uptime 4 days, 16:30:51

fw-agent            RUNNING    pid 17176, uptime 4 days, 16:30:51
  #CentOS 6与CentOs 7下结果一致 
  
```

安装salt minion
```
$ yum install salt-minion
```

修改/etc/salt/minion配置文件，添加如下信息：
```
master: 10.143.119.104
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
如果安装的主机本身已经安装过salt-minion，可能已经有之前配置的认定salt-master，会存在/etc/salt/pki/minion/minion_master.pub导致无法认证新master的问题，可以选择移除或配置双master。

移除不需要的master：
```
# mv /etc/salt/pki/minion/minion_master.pub /etc/salt/pki/minion/minion_master.pub.b
```

配置双master：

配置多Master时，Minion需要知道连接的每个Master的网络地址。需要在Minion的配置文件中进行配置，默认的配置文件是/etc/salt/minion。找到master的配置项，更新需要新增的Master，下面是一个多Master的配置例子：

```
master:
 - master1.example.tld
 - master2.example.tld
 
```



  [1]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87kafkacat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [2]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87filebeat%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [3]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87rsyslog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [4]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87logstash%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8ELinux%EF%BC%89.md
  [5]: https://github.com/yunjuhe/atd/blob/master/%E9%80%9A%E8%BF%87nxlog%E7%BB%99kafka%E6%8E%A8%E9%80%81%E6%97%A5%E5%BF%97%EF%BC%88%E9%80%82%E4%BA%8EWindows%EF%BC%89.md
