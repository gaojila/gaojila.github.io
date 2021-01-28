# k8s资源控制器


# ReplicaSet

## 什么是 ReplicaSet？

ReplicaSet 是下一代复本控制器。ReplicaSet 和 Replication Controller 之间的唯一区别是现在的选择器支持。Replication Controller 只支持基于等式的 selector（env=dev 或 environment!=qa），但 ReplicaSet 还支持新的，基于集合的 selector（version in (v1.0, v2.0)或 env notin (dev, qa)）。在试用时官方推荐 ReplicaSet。

大多数 kubectl 支持 Replication Controller 的命令也支持 ReplicaSets。rolling-update 命令有一个例外 。如果您想要滚动更新功能，请考虑使用 Deployments。此外， rolling-update 命令是必须的，而 Deployments 是声明式的，因此我们建议通过 rollout 命令使用 Deployments。

虽然 ReplicaSets 可以独立使用，但是今天它主要被 Deployments 作为协调 pod 创建，删除和更新的机制。当您使用 Deployments 时，您不必担心管理他们创建的 ReplicaSets。Deployments 拥有并管理其 ReplicaSets。

## 何时使用 ReplicaSet？

ReplicaSet 可确保指定数量的 pod“replicas”在任何设定的时间运行。然而，Deployments 是一个更高层次的概念，它管理 ReplicaSets，并提供对 pod 的声明性更新以及许多其他的功能。因此，我们建议您使用 Deployments 而不是直接使用 ReplicaSets，除非您需要自定义更新编排或根本不需要更新。

这实际上意味着您可能永远不需要操作 ReplicaSet 对象：直接使用 Deployments 并在规范部分定义应用程序。

## 演示

- 建立 rs.yml 文档

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 3 #有三个模板
  selector: #标签选择器
    matchLabels:
      tier: frontend
  template: #模板
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
        - name: myapp
          image: nginx
          env:
            - name: GET_HOSTS_FROM
              value: dns
          ports:
            - containerPort: 80
```

- 创建 pod

```bash
kubectl create -f rs.yml
```

- 查看 rs

```
[root@apiserver ~]# kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       21s
```

- 查看 pod

```
[root@apiserver ~]# kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
frontend-87wtq   1/1     Running   0          23s
frontend-r8kjt   1/1     Running   0          23s
frontend-w7n98   1/1     Running   0          23s
```

- 查看 rs 标签

```bash
[root@apiserver ~]# kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
frontend-87wtq   1/1     Running   0          2m39s   tier=frontend
frontend-r8kjt   1/1     Running   0          2m39s   tier=frontend
frontend-w7n98   1/1     Running   0          2m39s   tier=frontend
```

- 这时我们尝试修改标签：将 tier 改为 frontend1

```bash
[root@apiserver ~]# kubectl label pod frontend-87wtq tier=frontend1
error: 'tier' already has a value (frontend), and --overwrite is false
```

```bash
[root@apiserver ~]# kubectl label pod frontend-87wtq tier=frontend1 --overwrite=true
pod/frontend-87wtq labeled
[root@apiserver ~]# kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE     LABELS
frontend-87wtq   1/1     Running   0          5m49s   tier=frontend1
frontend-r8kjt   1/1     Running   0          5m49s   tier=frontend
frontend-w7n98   1/1     Running   0          5m49s   tier=frontend
frontend-wh4xx   1/1     Running   0          2s      tier=frontend
```

```bash
[root@apiserver ~]# kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       24m
```

这时我们会发现 rs 的数量没有变，pod 的数量多了一个，原因是 rs replicas: 3 是以标签 tier=frontend 为准，当标签修改后，他会自动新建一个和原来标签一样的 pod，当我们删掉 rs 之后，还会存留 tier=frontend1 的 pod

# deployment

Deployment 为 Pod 和 Replica Set（下一代 Replication Controller）提供声明式更新。

你只需要在 Deployment 中描述你想要的目标状态是什么，Deployment controller 就会帮你将 Pod 和 Replica Set 的实际状态改变到你的目标状态。你可以定义一个全新的 Deployment，也可以创建一个新的替换旧的 Deployment。

一个典型的用例如下：

1. 使用 Deployment 来创建 ReplicaSet。ReplicaSet 在后台创建 pod。检查启动状态，看它是成功还是失败。
2. 然后，通过更新 Deployment 的 PodTemplateSpec 字段来声明 Pod 的新状态。这会创建一个新的 ReplicaSet，Deployment 会按照控制的速率将 pod 从旧的 ReplicaSet 移动到新的 ReplicaSet 中。
3. 如果当前状态不稳定，回滚到之前的 Deployment revision。每次回滚都会更新 Deployment 的 revision。
4. 扩容 Deployment 以满足更高的负载。
5. 暂停 Deployment 来应用 PodTemplateSpec 的多个修复，然后恢复上线。
6. 根据 Deployment 的状态判断上线是否 hang 住了。
7. 清除旧的不必要的 ReplicaSet。

![](https://img2018.cnblogs.com/i-beta/1829785/201912/1829785-20191220160443664-101482876.png)

## 演示

- 创建 deployment.yml 文档

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
  spec:
    containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
```

- 创建 deployment

```bash
[root@apiserver ~]# kubectl apply -f deployment.yaml --record #record 版本记录
deployment.extensions/nginx-deployment created
```

- 查看 deplyment

```bash
[root@apiserver ~]# kubectl get deployments.
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           12s
```

- 查看 rs

```bash
[root@apiserver ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-86bddccdd7   3         3         3       96s
```

- 查看 pod

```bash
[root@apiserver ~]# kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-86bddccdd7-5s8ls   1/1     Running   0          19s
nginx-deployment-86bddccdd7-9frh8   1/1     Running   0          19s
nginx-deployment-86bddccdd7-wvcdx   1/1     Running   0          19s
[root@apiserver ~]# kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP                NODE                    NOMINATED NODE   READINESS GATES
nginx-deployment-86bddccdd7-5s8ls   1/1     Running   0          3m2s   192.168.102.179   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-9frh8   1/1     Running   0          3m2s   192.168.102.178   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-wvcdx   1/1     Running   0          3m2s   192.168.102.177   localhost.localdomain   <none>           <none>
```

- 查看程序是否正常运行(能访问页面即为正常)

```html
[root@apiserver ~]# curl 192.168.102.179

<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.
    </p>

    <p>
      For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br />
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
```

- 扩容：扩容到原来的 10 倍

```bash
[root@apiserver ~]# kubectl scale deployment nginx-deployment --replicas=10
deployment.extensions/nginx-deployment scaled
```

- 查看 pod

```bash
[root@apiserver ~]# kubectl  get pod  -o wide
NAME                                READY   STATUS              RESTARTS   AGE    IP                NODE                    NOMINATED NODE   READINESS GATES
nginx-deployment-86bddccdd7-5s8ls   1/1     Running             0          135m   192.168.102.179   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-9frh8   1/1     Running             0          135m   192.168.102.178   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-db44r   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-gh2s4   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-mmvc8   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-sqzn7   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-tb7k5   1/1     Running             0          3s     192.168.102.180   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-vtk8g   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-wvcdx   1/1     Running             0          135m   192.168.102.177   localhost.localdomain   <none>           <none>
nginx-deployment-86bddccdd7-z62js   0/1     ContainerCreating   0          3s     <none>            localhost.localdomain   <none>           <none>
```

- 对镜像进行修改更新

```bash
[root@apiserver ~]# kubectl set image deployment/nginx-deployment nginx=linuxserver/nginx
deployment.extensions/nginx-deployment image updated
```

- 查看 pod

```bash
[root@apiserver ~]# kubectl  get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP                NODE                    NOMINATED NODE   READINESS GATES
nginx-deployment-d46468c78-6855p   1/1     Running   0          32s   192.168.102.135   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-lmnw7   1/1     Running   0          24s   192.168.102.138   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-lxhvq   1/1     Running   0          34s   192.168.102.132   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-mg7j7   1/1     Running   0          31s   192.168.102.136   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-mwcdt   1/1     Running   0          23s   192.168.102.143   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-nw4gv   1/1     Running   0          28s   192.168.102.137   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-qdzt7   1/1     Running   0          16s   192.168.102.145   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-qqmhs   1/1     Running   0          34s   192.168.102.131   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-r59lc   1/1     Running   0          27s   192.168.102.134   localhost.localdomain   <none>           <none>
nginx-deployment-d46468c78-r7572   1/1     Running   0          17s   192.168.102.144   localhost.localdomain   <none>           <none>
```

- 查看能否访问

```html
[root@apiserver ~]# curl 192.168.102.135
<html>
  <head>
    <title>Welcome to our server</title>
    <style>
      body {
        font-family: Helvetica, Arial, sans-serif;
      }
      .message {
        width: 330px;
        padding: 20px 40px;
        margin: 0 auto;
        background-color: #f9f9f9;
        border: 1px solid #ddd;
      }
      center {
        margin: 40px 0;
      }
      h1 {
        font-size: 18px;
        line-height: 26px;
      }
      p {
        font-size: 12px;
      }
    </style>
  </head>
  <body>
    <div class="message">
      <h1>Welcome to our server</h1>
      <p>The website is currently being setup under this address.</p>
      <p>
        For help and support, please contact:
        <a href="me@example.com">me@example.com</a>
      </p>
    </div>
  </body>
</html>
```

- 查看 rs（因为试了几次，出现了 4 个 rs）

```bash
[root@apiserver ~]# kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-8665475fd6   0         0         0       27m
nginx-deployment-867d9bcb4c   0         0         0       18m
nginx-deployment-86bddccdd7   0         0         0       171m
nginx-deployment-d46468c78    10        10        10      3m15s
```

- 查看 deployment

```bash
[root@apiserver ~]# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   10/10   10           10          171m
```

- 进行回滚

```bash
[root@apiserver ~]# kubectl rollout undo deployment/nginx-deployment
deployment.extensions/nginx-deployment rolled back
```

- 查看回滚后 pod

```bash
[root@apiserver ~]# kubectl get pod -o wide
NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
nginx-deployment-86bddccdd7-2mmfx 1/1 Running 0 81s 192.168.102.147 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-5hr2z 1/1 Running 0 81s 192.168.102.146 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-5xsrz 1/1 Running 0 79s 192.168.102.148 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-7rf94 1/1 Running 0 68s 192.168.102.155 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-bjmfv 1/1 Running 0 78s 192.168.102.149 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-pw79g 1/1 Running 0 67s 192.168.102.156 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-r9hs5 1/1 Running 0 75s 192.168.102.151 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-snjgw 1/1 Running 0 73s 192.168.102.152 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-sw8tk 1/1 Running 0 77s 192.168.102.150 localhost.localdomain <none> <none>
nginx-deployment-86bddccdd7-tcrsq 1/1 Running 0 73s 192.168.102.153 localhost.localdomain <none> <none>
```

- 查看回滚后版本

```html
[root@apiserver ~]# curl 192.168.102.147

<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.
    </p>

    <p>
      For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br />
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
```

## 回滚的其它命令

- 查看回滚状态

```bash
[root@apiserver ~]# kubectl  rollout status deployment nginx-deployment
deployment "nginx-deployment" successfully rolled out
```

- 查看回滚历史

```bash
[root@apiserver ~]# kubectl  rollout history deployment nginx-deployment
deployment.extensions/nginx-deployment
REVISION  CHANGE-CAUSE
2         kubectl apply --filename=deployment.yaml --record=true
4         kubectl apply --filename=deployment.yaml --record=true
6         kubectl apply --filename=deployment.yaml --record=true
7         kubectl apply --filename=deployment.yaml --record=true
```

- 回滚到某一确定的历史版本

```bash
[root@apiserver ~]# kubectl rollout undo deployment nginx-deployment --to-revision=2
deployment.extensions/nginx-deployment rolled back
```

