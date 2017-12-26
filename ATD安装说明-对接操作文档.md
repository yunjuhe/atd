# ATD安装说明-对接操作文档

---


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

123.59.102.46 bgp-beijing-beijing-1-123-59-102-46

```
## 2、安装和配置kafka客户端

### 2.1、安装kafka客户端

```
# yum install bsc-kafkacat

```

验证kafka client和server端通信是否正常, 输出有相关topic信息：
```
# yum install supervisor

```
### 2.2、配置kafkacat收集日志

安装supervisor，用supervisor管理kafkacat进程:

```
# yum install supervisor
```

添加supervisor配置文件 `/etc/supervisord.d/kafkacat.ini` (其中/data/nginx_logs/xhw.json.log为web服务器的日志文件，需要您手动指定）：

```
[program:kafkacat]

command=sh -c "tail -F -q

/data/nginx_logs/xhw.json.log|/usr/local/bsc/kafkacat/bin/kafkacat -b bgp-beijing-beijing-1-123-59-102-46:6667 -t juhe-haihang;sleep 10"

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
master: 123.59.102.46
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