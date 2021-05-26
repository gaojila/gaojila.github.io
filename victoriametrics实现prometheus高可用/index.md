# VictoriaMetrics实现Prometheus高可用


## 背景
非容器下prometheus高可用体系调研
| 参数             | 联邦                                                         | Thanos                                                                   | VictoraMetrics                                                                                |
|------------------|--------------------------------------------------------------|--------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| 长期存储         | 小于一个月                                                   | 永久（对象存储）                                                         | 俩个月（可用本地盘存储）                                                                      |
| grafana配置      | 联邦节点，有oom风险                                          | query，长期存储读storegateway，短期读本都盘（指标过多未拆分时有oom风险） | vmselect，不使用prometheus，使用vmagent采集数据，比prometheus内存消耗更小，数据查询读vmselect |
| prometheus多副本 | 依赖亲和或standby机制，一般只有一个副本提供数据，service亲和 | 支持去重（副本重启数据不会断）                                           | 支持去重（副本重启数据不会断）                                                                |
| 部署             | 简单                                                         | 复杂                                                                     | 简单                                                                                          |
| 资源使用         | 多                                                           | 最多                                                                     | 较多                                                                                          |

## VictoraMetrics 架构
![架构图](https://raw.githubusercontent.com/gaojila/images/master/VictoriaMetrics实现Prometheus高可用/victoria.png)

