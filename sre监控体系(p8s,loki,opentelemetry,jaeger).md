[toc]

# 思路

通过 采集和存储前后端全端全链路的指标、事件、日志、性能, 生成告警通知, 主动发现异常并解决



充分利用云厂商提供的监控

关闭不能使用的功能， 提高告警灵敏度



链接仓库:	https://github.com/shilintan/monitor-research

# 指标

链接仓库:	https://github.com/shilintan/monitor-research/tree/main/metrics/base

## 指标采集

按照网络流入顺序采集指标

​	slb, nat, ecs, k8s, k8s master, k8s woker,k8s pod, cillium

​	nginx, jvm, 中间件指标(mysql, redis, rabbitmq, es), 应用中间件指标(seata, xxljob, nacos)

​	服务接口自动定时查询(httpGet)生成指标



### ecs

cpu使用率、iowait(20ms)

memory使用率

disk使用率、io耗时(20ms)

### mysql

slow log > 100

连接使用率 < 80%, 代码上出现不释放连接的问题, 一般情况下不会被用到很多, 可以通过配置文件调整大小, 但是也受到操作系统资源情况限制

buffer pool read 读取率 < 99%, 防止数据没有被内存缓存到而从磁盘加载导致慢SQL

buffer pool 利用率 < 5%, 有大量不带索引条件查询的SQL被查询, 而且缓存配置有问题需要调整

脏页比例 > 75%, 超过75%后如果有写入压力则会导致写入卡顿, 解决方式: 优化磁盘、分库分表分实例解决、提高redo log文件大小(innodb_log_file_size)



其他



### redis

slow log > 100

拒绝连接数量 > 100, 解决: 客户端定时检查redis连接, 且检查时间间隔要小于服务端配置

阻塞key数量 > 100, 解决: 优化网络(内核优化、分散集群、backlog调大、降低网络延迟)、提高网络配置(升配), 查看有哪些大key并优化大key, 根据业务类型划分到多个redis实例, 降低fork频率, 并发压力过大, 输出缓冲区大小调高(或者不限制)、使用redis-cluster替代redis、优化内核(文件描述符、进程)

key命中率 < 10%, 出现原因: 并发锁判空、缓存清理, 处理: 提高内存, 数据预热, 过期时间上加随机值以防止同时过期, 判断层不放在redis上做, 代码异常(代码死循环)



### rabbitmq

消息堆积量 > 500, 解决: 看系统是否有异常、rabbitmq负载是否过高、消费队列提高

消费者数量 < 1, 解决: 看消费者服务为什么宕机了, 滚动升级



### elasticsearch

jvm使用率 > 80%, 升级配置, 配置索引



### mongodb

连接数量 < 1, 看服务为什么宕机了



### seata





### xxljob



## 指标存储

至少存储一个月的数据, 必要时使用远程存储(VictoriaMetrics)

## 指标告警通知

80%饱和度、阈值、状态



# 日志

链接仓库:	https://github.com/shilintan/monitor-research/tree/main/tracing

写日志文件、采集日志文件、异常日志告警



服务集成日志框架, 按日志级别打印日志

日志中要包含traceid



服务日志

```
{scrape_job="service",app=~"service-.*"}|= "Exception"!="ServiceException"
```

中间件日志

​	seata

```
{app="seata-server",namespace="prod",scrape_job="kubernetes-pods"}|="Exception"
```



# 事件

链接仓库:	https://github.com/shilintan/monitor-research/tree/main/event/k8s



采集event, event告警

​	k8s event

​		服务自检

​			springboot-actuator

​				中间件连接检查

​				连接池可用率检查





# 性能

链接仓库:	https://github.com/shilintan/monitor-research/tree/main/tracing



客户端嵌入式采集agent, 采集性能, 对性能进行告警

​	java opentelemetry



通过日志文件计算性能

​	nginx access log	

​	mysql slowlog

​	redis slowlog

​	rabbitmq 堆积量

​	mongodb slowlog

​	es slowlog



# 告警

关闭不能使用的功能， 提高告警灵敏度



# 事件管理







# 问题

