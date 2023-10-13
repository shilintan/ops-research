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

## 云中间件

云mysql

云redis

云rabbitmq

云mongodb

云elasticsearch

cdn (内容缓存)

oss (对象文件存储)

## 中间件

minio

## 服务器

ecs

## 服务器集群

k8s

# 研发流程

优先本地调试, 开发环境, 测试环境, 生产环境

使用域名而不是IP访问

# 上线标准

## 安全性

### 网络

网络环境隔离(vpn(openvpn)), ecs无外网(使用slb和nat), 外网端口绑定到专门的ecs的端口上

### 账号

服务器只能使用证书文件登陆

k8s使用证书认证

中间件删除默认账号, 修改默认账号密码

每个中间件使用独立的账号密码, 调试人员使用专门人员的专门权限的账号密码

java 服务使用env文件剥离配置文件, 使用 jasypt 加密包, 配置文件中使用 ENC(加密后的字符串)

#### 应用

关闭服务的调试功能

使用DDOS盾和应用防火墙

静态代码扫描、依赖库cve扫描

### 维护

云账号独立到人

​	在多人重叠的情况下抽出角色

主账号不创建access key和secret, 每个服务使用专门的robot账号分配权限及创建access key和secret

使用jumpserver而不是开放ssh和账号密码

使用连接工具而不是直接连接原始中间件, 例如yearning

## 审计

云操作审计

SQL审计

用户操作审计

## 稳定性

### 维护

使用指令

使用脚本

使用容器化、k8s

gitops

terraform

git flow机制

### 部署

多可用区



内核优化、关闭swap

#### 中间件

磁盘类型必须为ssd

架构类型必须为集群(主从)

主从同步机制为半同步复制, 有一个从节点数据跟主节点数据一致



##### mysql

网关、读写分离(会话一致性)、主从

确定需求的cpu, memory, disk(size, iops)

##### redis

锁和缓存数据分离

主从, cluster可选

缓存清理策略

​	锁: 不清理(noeviction)

​	缓存数据: 最近最少( volatile-lru)

设置密码

锁的副本数量为3, min-slaves-to-write为1, min-slaves-max-lag为2, 否则存在主redis宕机引起部分锁丢失问题

| 配置项                   | 值        | 作用                                                  |
| ------------------------ | --------- | ----------------------------------------------------- |
| repl-diskless-sync       | no        | 防止在网络卡顿的情况导致主从复制数据丢失              |
| repl-diskless-sync-delay | 5         | 等待上一次同步传输完成的时间                          |
| repl-diskless-load       | disabled  | 从节点磁盘加载全量数据                                |
| repl-disable-tcp-nodelay | no        | 不合并带宽, 减少延迟, 避免丢失数据                    |
| repl-backlog-size        | 104857600 | 主从复制积压缓冲区, 类似mysql binlog/wal 文件大小限制 |
| repl-backlog-ttl         | 0         | 主节点不清理积压                                      |







##### rabbitmq

##### elasticsearch

##### mongodb

##### seata

多副本部署

##### xxljob

多副本部署

##### nacos

使用集群模式，配置集群数量，配置集群整体连接uri

#### k8s

通过公网上传到线上

通过修改`/etc/hosts`配置本地IP指向镜像仓库, 减少资源消耗、减少失误、加速镜像拉取

#### 应用

独立性, 不同类型服务在不同的资源池中

使用合适规格服务器

​	硬盘优于内存优于cpu, 中间件切记使用ssd

​	java服务使用跑在8c32g上

nginx 需要多台小而多的cpu

java需要大内存

时区设置

### 稳定变更

#### mysql-sql

使用无锁变更SQL, 例如yearning, 防止执行SQL(alert, update)导致数据库无法使用(锁表)



### 动态扩容

#### 服务动态扩容

hpa, vpa

基于cpu, memory, 自定义指标

### 流量管理

队列削峰填谷

限流机制

​	集群限流

​	节点限流

灰度环境

​	注意: mq隔离

​	第一量:

​		域名方式, 额外提供一个灰度域名指向灰度环境命名空间

​		指定用户方式, 网关判断用户ID为灰度流向灰度环境命名空间

​	全量:

​		缓慢提高部署到新版本的流量比



### 服务集群化

分布在多台服务器上, 至少3个副本

可下线、可扩容、可清理

自动断网、自动重启机制 (探针健康检查)

​	springboot actuator

### 动态扩缩容

通过cpu/memory/自定义指标流量 对服务副本进行扩容

### 客户端

客户端重试、熔断、fallback(熔断推断)

日志打印规范

​	远程调用

​	系统异常

### 配置

内核优化

配置文件调整为生产级别

​	关闭调试功能

​	提高连接池大小

​	jvm参数

​	提升生产级别感知, 服务注册发现

​	确定所有配置参数

​	降低额外流量影响

​		例如: 日志同步写改为异步写

### 监控

前端和后端的指标、事件、日志、性能



采集指标, 顺着网络进行采集, 记录指标, 至少保留一个月, 对指标的饱和度(80%)和状态进行告警

​	slb, nat, ecs, k8s, k8s master, k8s woker,k8s pod, cillium

​	nginx, jvm, 中间件指标(mysql, redis, rabbitmq, es), 应用中间件指标(seata, xxljob, nacos)

​	服务接口自动定时查询(httpGet)生成指标

采集event, event告警

​	k8s event

记录日志文件、采集日志文件、对日志文件进行告警

采集性能, 客户端嵌入式agent, 对性能进行告警

​	nginx access log

​	java opentracing

​	mysql slowlog

​	redis slowlog

​	rabbitmq 堆积量

​	mongodb slowlog

​	es slowlog

充分利用云厂商提供的监控指标

#### 通知

钉钉

短信

手机号码

### 优化

mysql

es

mongodb

redis

### 故障排查

java arthas排查包

前端 wireshark抓包

看监控

### 测试

代码单元测试

接口测试

集成测试

压力测试

中间件性能测试

故障测试

​	下线上线、扩容、备份还原

### 备份

#### mysql

每天定时备份, 保留一周

开启binlog, 保留一周

## 费用优化

对应服务使用合适的服务器规格

使用资源包而不是按量， 尽可能包年而不是包月

云厂商有优化活动(85折)时集中采购

测试环境可以放到本地环境做

### 数据管理

定期归档

​	高频存储、标准存储、冷存储、深度存储

### 容量管理

计算未来一段时间可以承载的数据量

提前分库分表分实例



### 处理量管理

web服务副本数、k8s集群大小、java服务节点池大小

中间件能够处理的并发量

统计最近使用人数及频次(例如一月), 对比可能的总量, 提前计算冗余量并提前扩容

动态扩容机制



## 技术储备

使用新技术时需要研究底层是怎么实现的, 会不会有什么问题

看官方文档, 看组件架构, 关注出了问题怎么解决

# 运维推进思路

轻重缓急

做急切的影响主线的

做长期重要的



核心是通过流水线减少运维工作, 通过监控减少事故, 资源隔离, 测试环境验证再上线, 新事情先使用临时方案再慢慢解决

一旦使用了就要考虑到出问题了怎么处理,  有哪里问题, 怎么暂存备份(进度)

使用稳定版本而不是latest, 使用测试过的版本而不是新的高版本, 小版本影响不大

尽可能使用统一体系而不是零碎的小的东西

多写文档



# 迁移思路

## 网络打通

## 数据迁移

### 文件卷

挂载数据目录, 生成备份压缩文件, 存储备份文件

挂载备份文件, 解压文件, 启动服务

### 实时数据

#### mysql/redis/mongodb

阿里云dts
