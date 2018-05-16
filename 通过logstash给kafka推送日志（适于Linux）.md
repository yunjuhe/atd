# 通过logstash给kafka推送日志（适于Linux）

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

## 以下是通过logstash给kafka推送日志的操作文档（适于Linux）：

# 1.下载logstash-5.6:

- 配置yum源，编辑`/etc/yum.repos.d/logstash-5.x.repo`文件：

```
[logstash-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

- 编辑完成后清除一下缓存:

        yum clean all
        
- 安装:

        yum install logstash-5.6.4-1.noarch

# 2.绑定kafka-server机器的hostname到ip的映射：
- 绑定kafka server,对应的主机名和ip如下，那么就需要在日志服务器上绑定这些hosts，在`/etc/hosts`文件中追加：

        vim /etc/hosts
```
{{ip}} {{hostname}}
```
- 例： 

        vim /etc/hosts
```
ATD1 172.18.1.2
ATD2 172.18.1.3
ATD3 172.18.1.4
```

# 3.编辑logstash的配置文件:
- 编辑文件`/etc/logstash/conf.d/file-kafka.conf` :

```
input {
    file {
        path => "/tmp/*.log"
        discover_interval => 1
        start_position => "end"
        sincedb_path => "/dev/null"
    }
}

output{
    kafka {
            codec => plain {
                format => "%{message}"
            }
            bootstrap_servers => "kafka1:6667"
            topic_id => "juhe-test"
            compression_type => "snappy"
    }
}
```

- 注意：
1）paths为本地文件的路径，支持多个文件的导入，支持通配符，支持文件句柄释放后新的同名文件的句柄引入。
2）bootstrap_servers为kafka-server的地址，格式为”ATD机器hostname:kafk-server的端口号”。
3）以上两个参数都可以设置多个，例：

        path => [“/tmp/*.log”, ”/tmp/2.log.b”]
        bootstrap_servers => [“kafka1:6667”, “kafka2:6667”, “kafka3:6667”]

- 完成logstash配置文件的编辑操作后，启动logstash：

        /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/file-kafka.conf

- 如果因为日志量过大而出现日志积压等情况，可以给logstash加`pipeline.workers`和 `pipeline.batch.size`，参数为在命令行中添加”`-w`”及”`-b`”，具体参考 `/usr/share/logstash/bin/logstash --help` 输出的结果。





- 参考官方文档：
https://www.elastic.co/guide/en/logstash/5.6/configuration.html


  [1]: http://1.com
  [2]: http://2.com
  [3]: http://3.com
  [4]: http://4.com
  [5]: http://5.com

