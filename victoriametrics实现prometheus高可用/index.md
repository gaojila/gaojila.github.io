# VictoriaMetrics实现Prometheus高可用


## 背景
非容器下prometheus高可用体系调研
| 参数             | 联邦                                                         | Thanos                                                                   | VictoriaMetrics                                                                                   |
|------------------|--------------------------------------------------------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| 长期存储         | 小于一个月                                                   | 永久（对象存储）                                                         | 俩个月（可用本地盘存储）                                                                          |
| grafana配置      | 联邦节点，有oom风险                                          | query，长期存储读storegateway，短期读本都盘（指标过多未拆分时有oom风险） | vmselect，不使用prometheus，使用vmagent采集数据，比prometheus内存消耗更小，数据查询读vmselect |
| prometheus多副本 | 依赖亲和或standby机制，一般只有一个副本提供数据，service亲和 | 支持去重（副本重启数据不会断）                                           | 支持去重（副本重启数据不会断）                                                                    |
| 部署             | 简单                                                         | 复杂                                                                     | 简单                                                                                              |
| 资源使用         | 多                                                           | 最多                                                                     | 较多                                                                                              |

## VictoraMetrics 架构

![架构图](https://raw.githubusercontent.com/gaojila/images/master/VictoriaMetrics实现Prometheus高可用/victoria.png)

## vmagent

### 为什么选择 vmagent？

1. 可用作prometheus的替代品，替代Prometheus抓取node_exporter等目标
2. 相比Prometheus消耗更少的cpu、io、内存
3. 支持Prometheus配置文件
4. 抓取大量指标时支持自动哈希
5. 可以将收集的指标同时复制到多个远程存储系统

### vmagent 选项

- -promscrape.config 带有 Prometheus 配置文件的路径
- -remoteWrite.url 对于 VictoriaMetrics 等远程存储端点，-remoteWrite.url可以多次指定该参数以将数据并发复制到任意数量的远程存储系统
- -http.connTimeout 连接超时时间
- -promscrape.maxScrapeSize 接收指标的最大大小
- -promscrape.suppressDuplicateScrapeTargetErrors 抑制抓取错误Target
- -promscrape.cluster.membersCount 设置hash节点个数
- -promscrape.cluster.replicationFactor 设置节点副本个数
- -promscrape.cluster.memberNum 当前节点标记
- -promscrape.streamParse 以流式方式抓取目标，这对于导出大量指标的目标可能很有用

### vmagent 部署

```
/opt/VictoriaMetrics/bin/vmagent -promscrape.config=/opt/prometheus.yml -remoteWrite.url=http://vminsert.gaojila.com/insert/0/prometheus/api/v1/write -http.connTimeout=1000ms -promscrape.maxScrapeSize=250MB -promscrape.suppressDuplicateScrapeTargetErrors -promscrape.cluster.membersCount=3 -promscrape.cluster.replicationFactor=2 -promscrape.cluster.memberNum=0 -promscrape.streamParse
```

### vmagent 扩容

vmagent 的扩容比较简单，只需要将新机器加入到集群中，更改所有vmagent的启动配置的-promscrape.cluster.membersCount和添加新主机的标记-promscrape.cluster.memberNum

## vminsert

vminsert是Victoria集群中的数据写入节点，起本身不存储数据只起到转发作用

### vminsert 选项

- -storageNode 指定数据存储节点
- -replicationFactor 指定副本数

### vminsert 部署

```
/opt/vminsert-prod -storageNode=192.168.1.2:8400 -storageNode=192.168.1.3:8400 -storageNode=192.168.1.4:8400 -replicationFactor=2
```
### vminsert 扩容

vminsert 扩容比较简单，它是无状态的，只需要将新机器加入到nginx中即可

## vmstorge

vmstorge 是集群中的存储节点，用于存储vminsert转发过来的数据和用于vmselect查询数据

### vmstorge 选项

- -search.maxUniqueTimeseries 查询的最大超时时间

### vmstorge 部署

```
/opt/vmstorage-prod -search.maxUniqueTimeseries=3000000
```
### vmstorge 扩容

vmstorge 扩容简单，只需要添加新机器，把vmstorage加入到vminsert和vmselect集群中

## vmselect

vmselect 集群中的查询节点，主要用于查询和去重合并

### vmselect 选项

- -storageNode 指定数据存储节点
- -dedup.minScrapeInterval 删除相应时间内的重复数据
- -search.logSlowQueryDuration 设置慢查时间
- -search.maxQueryDuration 查询执行的最长时间
- -search.maxQueueDuration 查询请求等待的最大时间
### vmselect 部署

```
/opt/vmselect-prod -storageNode=192.168.1.2:8401 -storageNode=192.168.1.3:8401 -storageNode=192.168.1.4:8401 -dedup.minScrapeInterval=13s -search.logSlowQueryDuration=15s -search.maxQueryDuration=50s -search.maxQueueDuration=50s
```

### vmselect 扩容

vmselect 扩容比较简单，它是无状态的，只需要将新机器加入到nginx中即可

