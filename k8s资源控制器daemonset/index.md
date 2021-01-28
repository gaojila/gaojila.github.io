# k8s资源控制器DaemonSet


## 什么是 DaemonSet

DaemonSet 确保全部（或者一些）Node 上运行一个 Pod 的副本，当有 Node 加入集群时，也会为他们新增一个 Pod，当有 Node 从集群移除时，这些 Pod 也会被回收，删除 DaemonSet 将会删除它所创建的所有 Pod。<br>
使用 DaemonSet 的一些典型用法：<br>

- 运行集群存储 daemon,例如在每个 Node 上运行`glusterd`、`ceph`。
- 在每个 Node 上运行日志收集 daemon，例如`fluentd`、`logstash`。
- 在每个 Node 上运行监控 daemon，例如`Prometheus Node Exporter`、`colletcd`、`Datadog`、`New Relic`、`Ganglia`、`gmond`。

**For Expamle**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-example
  labels:
    app: daemonset
spec:
  selector:
    matchLabels:
      name: daemonset-example
  template:
    metadata:
      labels:
        name: daemonset-example
    spec:
      containers:
        - name: daemonset-example
          image: wangyanglinux/myapp:v1
```

