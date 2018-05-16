# 安装拦截器文档（适于Ubuntu)

---
### 说明：
>  针对安装拦截器，Redhat/Centos和Ubuntu略有不同：
> <br/>
> （1）[安装拦截器（适于Redhat/Centos)操作文档][1]<br/>
> （2）[安装拦截器（适于Ubuntu)操作文档][2]<br/>


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

 [1]: https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8ERedhat&Centos%29.md
 [2]: https://github.com/yunjuhe/atd/blob/master/%E5%AE%89%E8%A3%85%E6%8B%A6%E6%88%AA%E5%99%A8%E6%96%87%E6%A1%A3%EF%BC%88%E9%80%82%E4%BA%8EUbuntu%29.md


