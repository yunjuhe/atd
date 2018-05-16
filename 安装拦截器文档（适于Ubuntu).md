# utuntu安装拦截器

标签（空格分隔）： 未分类

---

## 1、安装软件
- 官方文档：http://mirrors.nju.edu.cn/saltstack/2016.3.html

- 在文件/etc/apt/sources.list.d/saltstack.list中添加：
```
deb http://repo.saltstack.com/apt/ubuntu/16.04/amd64/2016.3 xenial main
```
- 安装：
```
# apt-get install salt-minion
# apt-get install ipset
```
## 2、修改配置
编辑`/etc/salt/minion`文件
```
master: 172.16.16.3
id: 本机的ip
```

## 3、启动
```
# sudo /etc/init.d/salt-minion start
```



