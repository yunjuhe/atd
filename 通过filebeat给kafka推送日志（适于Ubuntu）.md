# 通过filebeat给kafka推送日志（适于Ubuntu）

---

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
## 3、配置filebeat（标红色的/tmp/1.log应修改为要推送日志的路径）
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



