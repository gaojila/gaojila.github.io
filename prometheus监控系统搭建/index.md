# Prometheus 监控系统搭建


# 前言

- zabbix 对于运维人员来说无法完全做到自动化,内部业务即将 docker 化,故有将 zabbix 替换为 prometheus 的想法
- 这篇文档将会更新很久
- [参照此文档未完](https://songjiayang.gitbooks.io/prometheus/)
- [参考此文档](https://yunlzheng.gitbook.io/prometheus-book/)

# Prometheus 搭建

## supervisor 守护进程方式

1. github 下载最新二进制包

```
# 使用的包时github的最新稳定版包
wget https://github.com/prometheus/prometheus/releases/download/v2.17.1/prometheus-2.17.1.linux-amd64.tar.gz
```

2. 将下载的二进制包解压到你想要放到的目录下

3. supervisor 守护进程配置

## docker 方式

# Node_exporter 搭建

# Alertmanage 搭建

# Grafana 搭建

# Prometheus 常用内置函数

# Prometheus 自定义监控

# Alertmanage 告警路由

# Prometheus 告警规则

# Grafana 配置告警

# Grafana 画图

