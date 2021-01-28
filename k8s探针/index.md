# k8s探针


# 配置 readiness 和 liveness 探针

**_(参考 Jimmy Song 大佬的文章学习)_**<br>
Kubelet 使用 liveness probe（存活探针）来确定何时重启容器。例如，当应用程序处于运行状态但无法做进一步操作，liveness 探针将捕获到 deadlock，重启处于该状态下的容器，使应用程序在存在 bug 的情况下依然能够继续运行下去（谁的程序还没几个 bug 呢）。<br>
Kubelet 使用 readiness probe（就绪探针）来确定容器是否已经就绪可以接受流量。只有当 Pod 中的容器都处于就绪状态时 kubelet 才会认定该 Pod 处于就绪状态。该信号的作用是控制哪些 Pod 应该作为 service 的后端。如果 Pod 处于非就绪状态，那么它们将会被从 service 的 load balancer 中移除。<br>

## liveness 探针

### 定义一个 liveness exec

许多长时间运行的应用程序最终会转换到 broken 状态，除非重新启动，否则无法恢复。Kubernetes 提供了 liveness probe 来检测和补救这种情况。<br>
在本次练习将基于 busybox 镜像创建运行一个容器的 Pod。以下是 Pod 的配置文件 live-exec.yaml：<br>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec-pod
  namespace: default
spec:
  containers:
    - name: liveness-exec-container
      image: busybox
      imagePullPolicy: IfNotPresent
      command:
        [
          "sh",
          "-c",
          "touch /tmp/live ; sleep 60 ; rm -rf /tmp/live ; sleep 3600",
        ]
      livenessProbe:
        exec:
          command: ["test", "-e", "/tmp/live:"]
        initialDelaySeconds: 1
        periodSeconds: 3
```

该配置文件给 Pod 配置了一个容器。periodSeconds 规定 kubelet 要每隔 3 秒执行一次 liveness probe。 initialDelaySeconds 告诉 kubelet 在第一次执行 probe 之前要的等待 1 秒钟。探针检测命令是在容器中执行 cat /tmp/live 命令。如果命令执行成功，将返回 0，kubelet 就会认为该容器是活着的并且很健康。如果返回非 0 值，kubelet 就会杀掉这个容器并重启它。<br>
容器启动时，执行该命令：<br>

```bash
/bin/bash -c "touch /tmp/live ; sleep 60 ; rm -rf /tmp/live ; sleep 3600"

```

在容器生命的最初 60 秒内有一个 /tmp/live 文件，在这 60 秒内 cat /tmp/live 命令会返回一个成功的返回码。60 秒后， cat /tmp/live 将返回失败的返回码。<br>
创建 pod：<br>

```bash
kubectl create -f live-exec.yml
```

60 秒后，查看 Pod 已经有重启：<br>

```bash
[root@kube-master01 liveness]# kubectl get pod liveness-exec-pod -w
NAME                READY   STATUS    RESTARTS   AGE
liveness-exec-pod   1/1     Running   2          91s
```

### 定义一个 liveness httpGet

我们还可以使用 HTTP GET 请求作为 liveness probe。下面是一个基于 gcr.io/google_containers/liveness 镜像运行了一个容器的 Pod 的例子 http-liveness.yaml：<br>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-httpget-pod
  namespace: default
spec:
  containers:
    - name: liveness-httpget-container
      image: wangyanglinux/myapp:v1
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 80
      livenessProbe:
        httpGet:
          port: 80
          path: /index.html
        initialDelaySeconds: 1
        periodSeconds: 3
        timeoutSeconds: 10
```

### 定义一个 liveness tcpSocket

第三种 liveness probe 使用 TCP Socket。 使用此配置，kubelet 将尝试在指定端口上打开容器的套接字。 如果可以建立连接，容器被认为是健康的，如果不能就认为是失败的。<br>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp-pod
  namespace: default
spec:
  containers:
    - name: liveness-tcp-container
      image: wangyanglinux/myapp:v1
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 80
      livenessProbe:
        tcpSocket:
          port: 80
        initialDelaySeconds: 1
        periodSeconds: 3
        timeoutSeconds: 10
```

### 使用命名端口

可以使用命名的 ContainerPort 作为 HTTP 或 TCP liveness 检查：<br>

```yml
ports:
  - name: liveness-port
    containerPort: 8080
    hostPort: 8080

livenessProbe:
  httpGet:
  path: /healthz
  port: liveness-port
```

## 定义 readiness 探针

有时，应用程序暂时无法对外部流量提供服务。 例如，应用程序可能需要在启动期间加载大量数据或配置文件。 在这种情况下，你不想杀死应用程序，但你也不想发送请求。 Kubernetes 提供了 readiness probe 来检测和减轻这些情况。 Pod 中的容器可以报告自己还没有准备，不能处理 Kubernetes 服务发送过来的流量。<br>
Readiness probe 的配置跟 liveness probe 很像。唯一的不同是使用 readinessProbe 而不是 livenessProbe。<br>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-httpget-pod
  namespace: default
spec:
  containers:
    - name: readiness-httpget-container
      image: wangyanglinux/myapp:v1
      imagePullPolicy: IfNotPresent
      readinessProbe:
        httpGet:
          port: 80
          path: /index1.html
        initialDelaySeconds: 1
        periodSeconds: 3
```

Readiness probe 的 HTTP 和 TCP 的探测器配置跟 liveness probe 一样。<br>
Readiness 和 livenss probe 可以并行用于同一容器。 使用两者可以确保流量无法到达未准备好的容器，并且容器在失败时重新启动。<br>

## 配置 Probe

Probe 中有很多精确和详细的配置，通过它们你能准确的控制 liveness 和 readiness 检查：<br>

- initialDelaySeconds：容器启动后第一次执行探测是需要等待多少秒。
- periodSeconds：执行探测的频率。默认是 10 秒，最小 1 秒。
- timeoutSeconds：探测超时时间。默认 1 秒，最小 1 秒。
- successThreshold：探测失败后，最少连续探测成功多少次才被认定为成功。默认是 1。对于 liveness 必须是 1。最小值是 1。
- failureThreshold：探测成功后，最少连续探测失败多少次才被认定为失败。默认是 3。最小值是 1。

HTTP probe 中可以给 httpGet 设置其他配置项：<br>

- host：连接的主机名，默认连接到 pod 的 IP。你可能想在 http header 中设置"Host"而不是使用 IP。
- scheme：连接使用的 schema，默认 HTTP。
- path: 访问的 HTTP server 的 path。
- httpHeaders：自定义请求的 header。HTTP 运行重复的 header。
- port：访问的容器的端口名字或者端口号。端口号必须介于 1 和 65535 之间。

对于 HTTP 探测器，kubelet 向指定的路径和端口发送 HTTP 请求以执行检查。 Kubelet 将 probe 发送到容器的 IP 地址，除非地址被 httpGet 中的可选 host 字段覆盖。 在大多数情况下，你不想设置主机字段。 有一种情况下你可以设置它。 假设容器在 127.0.0.1 上侦听，并且 Pod 的 hostNetwork 字段为 true。 然后，在 httpGet 下的 host 应该设置为 127.0.0.1。 如果你的 pod 依赖于虚拟主机，这可能是更常见的情况，你不应该是用 host，而是应该在 httpHeaders 中设置 Host 头。

