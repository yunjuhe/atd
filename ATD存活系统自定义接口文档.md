
# 自定义处理方式接口开发文档
------

### 说明
> 接口以GET方式调用，返回json格式数据，数据中只包含存活且可用的机器列表。调用频率1分钟一次，如果接口调用失败，维持原有集群状态不变。

### 注意事项
> 1. 对于将缩容的机器，接口需在机器宕机之前大于10分钟就已对接口的数据进行更新，保证ATD平滑自动缩容
> 2. 对于将扩容的机器，接口需在机器正常开启10分钟之后对接口的数据进行更新，保证ATD平滑自动扩容

### 1. response body数据样例

```json
{
  "code": 0,
  "msg": "success",
  "data": [
  {
    "ip_addr": "192.168.0.1",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_pass": "123456"
  },
  {
    "ip_addr": "192.168.0.2",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_pass": "123456"   
  },
  {
    "ip_addr": "192.168.0.3",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_pass": "123456"
  }
  ]
}
```    

**字段解释：**

|字段名|类型|取值范围|解释|
|------|------|------|------|
|code|int|-|调用结果， 0表示成功，1表示失败|
|msg|string|-|提示信息|
|data|array|-|用于返回存活机器的列表，ATD扩缩容机器需要root登录权限，若登录无需密码，可不填写ssh_pass该key|

### 2. 调用示例

> 说明：<br/>
> 1、在验证时，我们将会调用如上样例的接口请求数据；<br/>
> 2、如果10s内返回结果，并且（1）返回的数据为如上格式；（2）code值为0，则将验证通过。<br/>

```
[root@bs203 ~]# curl http://172.18.1.203:8001/system/alive 

{
  "code": 0,
  "msg": "success",
  "data": [
  {
    "ip_addr": "192.168.0.1",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_password": "123456"
  },
  {
    "ip_addr": "192.168.0.2",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_password": "123456"   
  },
  {
    "ip_addr": "192.168.0.3",
    "ssh_port": 22,
    "ssh_user": "root",
    "ssh_password": "123456"
  }
  ]
}
```
