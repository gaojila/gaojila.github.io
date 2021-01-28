# CKA(2.kubectl logs)


## 题目

```
监控 foobar Pod 的日志，提取 pod 相应的行'error'写入到/logs 文件中
```

## 解题

查看 pod 的日志，是使用 kubelet logs 命令,选项如下:

```bash
Options:
      --all-containers=false: Get all containers' logs in the pod(s).
  -c, --container='': Print the logs of this container
  -f, --follow=false: Specify if the logs should be streamed.
      --ignore-errors=false: If watching / following pod logs, allow for any errors that occur to be
non-fatal
      --insecure-skip-tls-verify-backend=false: Skip verifying the identity of the kubelet that logs
are requested from.  In theory, an attacker could provide invalid log content back. You might want
to use this if your kubelet serving certificates have expired.
      --limit-bytes=0: Maximum bytes of logs to return. Defaults to no limit.
      --max-log-requests=5: Specify maximum number of concurrent logs to follow when using by a
selector. Defaults to 5.
      --pod-running-timeout=20s: The length of time (like 5s, 2m, or 3h, higher than zero) to wait
until at least one pod is running
      --prefix=false: Prefix each log line with the log source (pod name and container name)
  -p, --previous=false: If true, print the logs for the previous instance of the container in a pod
if it exists.
  -l, --selector='': Selector (label query) to filter on.
      --since=0s: Only return logs newer than a relative duration like 5s, 2m, or 3h. Defaults to
all logs. Only one of since-time / since may be used.
      --since-time='': Only return logs after a specific date (RFC3339). Defaults to all logs. Only
one of since-time / since may be used.
      --tail=-1: Lines of recent log file to display. Defaults to -1 with no selector, showing all
log lines otherwise 10, if a selector is provided.
      --timestamps=false: Include timestamps on each line in the log output

Usage:
  kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

对于一般的查看，只需要使用  kubectl logs podname 就可以了

```bash
root@shein-mointor-lab-01:~# kubectl logs node-exporter-pfw4v -n kube-monitorc
```

```bash
-c, --container='': Print the logs of this container
```

在查看 pod 的日志时，如果 pod 中只有一个容器，那么不用这个选项，如果 pod 中有多个容器，我们就需要指定其中一个。

比如我们的 pod kucc 中有 4 个容器，redis，nginx，memcached 和 consul，我们要看 redis 的日志，直接访问 pod 的日志是不可以的，必须加上 -c 指定 redis 容器

```bash
kubectl logs kucc -c redis
```

```bash
-f, --follow=false: Specify if the logs should be streamed.
```

类似 tail -f

```bash
kubectl logs -f kucc -c redis
```

```bash
-l, --selector='': Selector (label query) to filter on.
```

这个和 kubectl get 的-l 是一样的，是选择一类 pod 的输出，一般都是同类的 pod 数量非常多，我们会使用这个选项来追踪错误日志。

```bash
kubectl logs -f -l name=node-exporter -n kube-monitor
```

```bash
--since=0s: Only return logs newer than a relative duration like 5s, 2m, or 3h. Defaults to all logs. Only one of since-time / since may be used.
```

这个可以看一下最近时间的日志，比如我们看 10 秒之内的日志。

```bash
kubectl logs -f -l name=node-exporter -n kube-monitor --since=10s
```

```bash
--since-time='': Only return logs after a specific date (RFC3339). Defaults to all logs. Only one of since-time since may be used.
```

指定时间，但是格式是 ‘2020-02-25T14:33:33Z’，比如

```bash
kubectl logs -f -l name=node-exporter -n kube-monitor --since-time='2020-02-25T14:33:33Z'
```

## 答案

```bash
kubectl logs foobar | grep error > /logs
```

