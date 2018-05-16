# 安装拦截器文档（适于Redhat/Centos)

---

### 说明：
>  针对安装拦截器，Redhat/Centos和Ubuntu略有不同：
> <br/>
> （1）[通过kafkacat给kafka推送日志][1]<br/>
> （2）[安装拦截器文档（适于Ubuntu)][2]<br/>

## 以下是安装拦截器（适于Redhat/Centos)的操作文档：

## 1. 添加yum repo
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
## 2. 安装拦截器扩展
安装supervisor，用supervisor管理kafkacat进程:

```
$ yum install supervisor
```
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
查看拦截器扩展进程是否启动：

```
$ /etc/init.d/supervisord status   #CentOS 6版本
$ supervisorctl   #CentOS 7版本
```
```
fw-agent            RUNNING    pid 17176, uptime 4 days, 16:30:51
#CentOS 6与CentOs 7下结果一致 
  
```
## 3. 安装salt-minion
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

## 3，常见的问题及解决办法：
 （1） 网络连通性测试，测试与atd机器的salt-maste是否连通：
 
```
 $ telnet ATD1 4506
 $ telnet ATD1 4505
```

（2）如果安装的主机本身已经安装过salt-minion，可能已经有之前配置的认定salt-master，会存在/etc/salt/pki/minion/minion_master.pub导致无法认证新master的问题，可以选择移除或配置双master。

移除不需要的master：
```
$ mv /etc/salt/pki/minion/minion_master.pub /etc/salt/pki/minion/minion_master.pub.b
```

（3）salt配置双master：

配置多Master时，Minion需要知道连接的每个Master的网络地址。需要在Minion的配置文件中进行配置，默认的配置文件是/etc/salt/minion。找到master的配置项，更新需要新增的Master，下面是一个多Master的配置例子：

```
master:
 - master1.example.tld
 - master2.example.tld
```
（4）supervisor启动失败：
在`/etc/supervisord.conf`文件中查看是否有以下内容，如果没有，请添加。
```
[include]
files = supervisord.d/*.ini
```


[1]: https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8ERedhat%26Centos).md
[2]: https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8EUbuntu).md
