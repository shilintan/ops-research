# 案例场景

下面列举了一套系统需要用到的东西

ram(账号管理)

​	key/secret

## 网络

dns (域名管理)

ssl (证书)

eip (公网IP)

slb (4层负载均衡)

nat (出口网关)

vpc (局域网)

应用防火墙



## 中间件

云mysql

云redis

云rabbitmq

云mongodb

云es

cdn (内容缓存)

oss (对象文件存储)



## 服务器

ecs



## 服务器集群

k8s



# 上线标准

## 安全性

网络环境隔离(vpn(openvpn)), ecs无外网(使用slb和nat), 外网端口绑定到专门的ecs的端口上

服务器只能使用证书文件登陆

删除默认账号, 修改默认账号密码

关闭服务的调试功能

k8s使用证书认证



使用DDOS盾和应用防火墙

静态代码扫描、依赖库cve扫描

### 维护

云账号独立到人

​	在多人重叠的情况下抽出角色

主账号不创建access key和secret, 每个服务使用专门的robot账号分配权限及创建access key和secret

## 稳定性

### 维护

使用指令

使用脚本

使用容器化、k8s

gitops

terraform

git flow机制

### 测试

代码单元测试

接口测试

集成测试

压力测试



### 部署

独立性, 不同类型服务在不同的资源池中

使用低规格服务器, 8c32g

### 流量管理

队列填谷削峰

灰度环境

​	域名级别、部分用户级别

​		mq隔离



### 服务集群化

分布在多台服务器上, 至少3个副本

可扩容、可清理

 自动断网、自动重启机制 (探针健康检查)

​	springboot actuator

限流机制

​	集群限流

​	节点限流

### 客户端

客户端重试、熔断、fallback(熔断推断)



### 配置

内核优化

配置文件调整为生产级别

​	关闭调试功能

​	提高连接池大小

​	jvm参数

​	提升生产级别感知, 服务注册发现

​	确定所有配置参数

​	降低额外流量影响

### 监控

前端监控状态、日志、性能、事件

采集指标, 顺着网络进行采集, 记录指标, 至少保留一个月, 对指标的饱和度(80%)和状态进行告警

​	slb, nat, ecs, k8s, k8s master, k8s woker,k8s pod

​	nginx, jvm, 中间件指标(mysql, redis, rabbitmq, es), 应用中间件指标(seata, xxljob, nacos)

采集event, event告警	

​	k8s event

记录日志文件、采集日志文件、对日志文件进行告警

采集性能, 客户端嵌入式agent, 对性能进行告警

​	nginx access log

​	java opentracing

​	mysql slowlog

​	redis slowlog

​	rabbitmq 堆积量



服务接口自动定时查询(httpGet), 告警 

#### 通知

钉钉

短信

手机号码

### 故障排查

java arthas排查包

前端 wireshark抓包

看监控
