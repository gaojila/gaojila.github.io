# CKA(1.kubectl get)


## 题目

```
使用 name 排序列出所有的 PV，把输出内容存储到/opt/文件中 使用 kubectl own 对输出进行排序，并且不再进一步操作它
```

## 解题

kubectl 是用来管理 k8s 的命令行工具之一，他主要是调用 KUBECONFIG 文件，并且加载其中的信息，向 kube-apiserver 发起请求。

一般格式

```bash
kubectl [command] [type] [name] [tags]
```

[command]: 是说指定要对资源执行的操作，下面是所有的命令，可以执行所有的k8s操作。

```bash
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create        Create a resource from a file or from stdin.
  expose        Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
  run           Run a particular image on the cluster
  set           Set specific features on objects

Basic Commands (Intermediate):
  explain       Documentation of resources
  get           Display one or many resources
  edit          Edit a resource on the server
  delete        Delete resources by filenames, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale     Auto-scale a Deployment, ReplicaSet, or ReplicationController

Cluster Management Commands:
  certificate   Modify certificate resources.
  cluster-info  Display cluster info
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        Mark node as unschedulable
  uncordon      Mark node as schedulable
  drain         Drain node in preparation for maintenance
  taint         Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe      Show details of a specific resource or group of resources
  logs          Print the logs for a container in a pod
  attach        Attach to a running container
  exec          Execute a command in a container
  port-forward  Forward one or more local ports to a pod
  proxy         Run a proxy to the Kubernetes API server
  cp            Copy files and directories to and from containers.
  auth          Inspect authorization

Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         Apply a configuration to a resource by filename or stdin
  patch         Update field(s) of a resource using strategic merge patch
```

[type]是指资源的类型，比如 pod，service，deployment

```bash
kubectl get pods
kubectl get deploy
```

需要注意的是，如果不加[name]选项，就是 get 所有的，而 pods 和 pod 目前的意思是一样的。

一个使用 kubeamd 新创建的集群，刚刚创建完成的时候，会有两个 namespaces，一个是 kube-system，用来存放 kube-apiserver 等系统用到的 pod，一个是 default，作为默认名称空间，如果创建资源的时候不指定名称空间，都会在这个空间里面存放。

```bash
root@shein-mointor-lab-01:~# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2d1h
root@shein-mointor-lab-01:~# kubectl get pod -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE
calico-kube-controllers-76d4774d89-6tg9v       1/1     Running   2          38d
calico-node-65tsz                              1/1     Running   1          38d
calico-node-gzh5l                              1/1     Running   2          38d
coredns-7ff77c879f-ql8dh                       1/1     Running   2          38d
coredns-7ff77c879f-zp8ms                       1/1     Running   2          38d
eip-nfs-kube-monitor-5f98475889-5tv7v          1/1     Running   2          3d12h
eip-nfs-kube-ops-554dbf6855-wwll7              1/1     Running   0          2d11h
etcd-shein-mointor-lab-01                      1/1     Running   3          38d
kube-apiserver-shein-mointor-lab-01            1/1     Running   2          38d
kube-controller-manager-shein-mointor-lab-01   1/1     Running   2          38d
kube-proxy-mpbdh                               1/1     Running   2          38d
kube-proxy-swj5c                               1/1     Running   1          38d
kube-scheduler-shein-mointor-lab-01            1/1     Running   3          38d
kuboard-5454b89cb9-bz4s7                       1/1     Running   0          25d
metrics-server-7f96bbcc66-zr276                1/1     Running   2          3d12h
root@shein-mointor-lab-01:~#
```

[flags]选项也是琳琅满目，可以使用 kubectl get -h 来列出所有的选项，我们来对常用的进行说明。

```bash
root@shein-mointor-lab-01:~# kubectl get -h
Display one or many resources

 Prints a table of the most important information about the specified resources. You can filter the
list using a label selector and the --selector flag. If the desired resource type is namespaced you
will only see results in your current namespace unless you pass --all-namespaces.

 Uninitialized objects are not shown unless --include-uninitialized is passed.

 By specifying the output as 'template' and providing a Go template as the value of the --template
flag, you can filter the attributes of the fetched resources.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # List all pods in ps output format.
  kubectl get pods

  # List all pods in ps output format with more information (such as node name).
  kubectl get pods -o wide

  # List a single replication controller with specified NAME in ps output format.
  kubectl get replicationcontroller web

  # List deployments in JSON output format, in the "v1" version of the "apps" API group:
  kubectl get deployments.v1.apps -o json

  # List a single pod in JSON output format.
  kubectl get -o json pod web-pod-13je7

  # List a pod identified by type and name specified in "pod.yaml" in JSON output format.
  kubectl get -f pod.yaml -o json

  # List resources from a directory with kustomization.yaml - e.g. dir/kustomization.yaml.
  kubectl get -k dir/

  # Return only the phase value of the specified pod.
  kubectl get -o template pod/web-pod-13je7 --template={{.status.phase}}

  # List resource information in custom columns.
  kubectl get pod test-pod -o
custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

  # List all replication controllers and services together in ps output format.
  .......................................省略
```

```bash
-A, --all-namespaces=false: If present, list the requested object(s) across all namespaces. Namespace in current context is ignored even if specified with --namespace.
```

默认的 get 命令是只列出 context 中指定的名称空间中的资源的，而 context 中的默认名称空间是 default，所以如果想把所有名称空间中的指定资源列出来，需要使用-A 选项。在以前的版本中是使用--all-namespaces 选项。目前依然好用，但是说明文档中已经没有了。

```bash
root@shein-mointor-lab-01:~# kubectl get pods -A
NAMESPACE       NAME                                           READY   STATUS    RESTARTS   AGE
default         nginx                                          1/1     Running   0          2d2h
ingress-nginx   ingress-nginx-controller-69fb496d7d-f9xjz      1/1     Running   1          3d12h
kube-monitor    node-exporter-pfw4v                            1/1     Running   0          3d10h
kube-monitor    node-exporter-zh2p6                            1/1     Running   0          3d10h
kube-monitor    prometheus-6978748887-4fxxv                    1/1     Running   1          3d12h
kube-ops        jenkins2-6fdbbc868d-lc46q                      1/1     Running   0          2d11h
kube-system     calico-kube-controllers-76d4774d89-6tg9v       1/1     Running   2          38d
kube-system     calico-node-65tsz                              1/1     Running   1          38d
kube-system     calico-node-gzh5l                              1/1     Running   2          38d
kube-system     coredns-7ff77c879f-ql8dh                       1/1     Running   2          38d
kube-system     coredns-7ff77c879f-zp8ms                       1/1     Running   2          38d
kube-system     eip-nfs-kube-monitor-5f98475889-5tv7v          1/1     Running   2          3d12h
kube-system     eip-nfs-kube-ops-554dbf6855-wwll7              1/1     Running   0          2d11h
kube-system     etcd-shein-mointor-lab-01                      1/1     Running   3          38d
kube-system     kube-apiserver-shein-mointor-lab-01            1/1     Running   2          38d
kube-system     kube-controller-manager-shein-mointor-lab-01   1/1     Running   2          38d
kube-system     kube-proxy-mpbdh                               1/1     Running   2          38d
kube-system     kube-proxy-swj5c                               1/1     Running   1          38d
kube-system     kube-scheduler-shein-mointor-lab-01            1/1     Running   3          38d
kube-system     kuboard-5454b89cb9-bz4s7                       1/1     Running   0          25d
kube-system     metrics-server-7f96bbcc66-zr276                1/1     Running   2          3d12h
```

```bash
--field-selector='': Selector (field query) to filter on, supports '=', '==', and '!='.(e.g. --field-selector key1=value1,key2=value2). The server only supports a limited number of field queries per type.
```

这个选项是选择特定字段的资源，字段就是用 kubectl get pod 加上-o yaml 选项看到的字段，比如

```bash
root@shein-mointor-lab-01:~# kubectl get pods nginx -n default -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 10.122.136.206/32
    cni.projectcalico.org/podIPs: 10.122.136.206/32
  creationTimestamp: "2020-07-07T11:08:42Z"
  labels:
    disk: ssd
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:disk: {}
      f:spec:
        f:containers:
          k:{"name":"nginx"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
..........................................................省略
```

而我们就可以使用这个命令来指定，只 get 特定名称的资源

```bash
root@shein-mointor-lab-01:~# kubectl get pod --field-selector metadata.name=nginx -n default
\NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2d2h
```

```bash
-L, --label-columns=[]: Accepts a comma separated list of labels that are going to be presented as columns. Names are case-sensitive. You can also use multiple flag options like -L label1 -L label2...
```

这个是指定在输出结果的时候，加上指定的列，比如加上 -L tier

```bash
root@shein-mointor-lab-01:~# kubectl get pods -L tier -n kube-system
NAME                                           READY   STATUS    RESTARTS   AGE     TIER
calico-kube-controllers-76d4774d89-6tg9v       1/1     Running   2          38d
calico-node-65tsz                              1/1     Running   1          38d
calico-node-gzh5l                              1/1     Running   2          38d
coredns-7ff77c879f-ql8dh                       1/1     Running   2          38d
coredns-7ff77c879f-zp8ms                       1/1     Running   2          38d
eip-nfs-kube-monitor-5f98475889-5tv7v          1/1     Running   2          3d12h
eip-nfs-kube-ops-554dbf6855-wwll7              1/1     Running   0          2d12h
etcd-shein-mointor-lab-01                      1/1     Running   3          38d     control-plane
kube-apiserver-shein-mointor-lab-01            1/1     Running   2          38d     control-plane
kube-controller-manager-shein-mointor-lab-01   1/1     Running   2          38d     control-plane
kube-proxy-mpbdh                               1/1     Running   2          38d
kube-proxy-swj5c                               1/1     Running   1          38d
kube-scheduler-shein-mointor-lab-01            1/1     Running   3          38d     control-plane
kuboard-5454b89cb9-bz4s7                       1/1     Running   0          25d
metrics-server-7f96bbcc66-zr276                1/1     Running   2          3d12h
```

```bash
--no-headers=false: When using the default or custom-column output format, don't print headers (default printheaders).
```

这个就是说在输出的时候不要列的名称，方便我们使用 awk 进行截取。

```bash
root@shein-mointor-lab-01:~# kubectl get pods -A --no-headers
default         nginx                                          1/1   Running   0     2d2h
ingress-nginx   ingress-nginx-controller-69fb496d7d-f9xjz      1/1   Running   1     3d12h
kube-monitor    node-exporter-pfw4v                            1/1   Running   0     3d10h
kube-monitor    node-exporter-zh2p6                            1/1   Running   0     3d10h
kube-monitor    prometheus-6978748887-4fxxv                    1/1   Running   1     3d12h
kube-ops        jenkins2-6fdbbc868d-lc46q                      1/1   Running   0     2d12h
kube-system     calico-kube-controllers-76d4774d89-6tg9v       1/1   Running   2     38d
kube-system     calico-node-65tsz                              1/1   Running   1     38d
kube-system     calico-node-gzh5l                              1/1   Running   2     38d
kube-system     coredns-7ff77c879f-ql8dh                       1/1   Running   2     38d
kube-system     coredns-7ff77c879f-zp8ms                       1/1   Running   2     38d
kube-system     eip-nfs-kube-monitor-5f98475889-5tv7v          1/1   Running   2     3d12h
kube-system     eip-nfs-kube-ops-554dbf6855-wwll7              1/1   Running   0     2d12h
kube-system     etcd-shein-mointor-lab-01                      1/1   Running   3     38d
kube-system     kube-apiserver-shein-mointor-lab-01            1/1   Running   2     38d
kube-system     kube-controller-manager-shein-mointor-lab-01   1/1   Running   2     38d
kube-system     kube-proxy-mpbdh                               1/1   Running   2     38d
kube-system     kube-proxy-swj5c                               1/1   Running   1     38d
kube-system     kube-scheduler-shein-mointor-lab-01            1/1   Running   3     38d
kube-system     kuboard-5454b89cb9-bz4s7                       1/1   Running   0     25d
kube-system     metrics-server-7f96bbcc66-zr276                1/1   Running   2     3d12h
```

```bash
-o, --output='': Output format. One of: json|yaml|wide|name|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...
```

这个是说输出的格式，我们最常用的就是 -o yaml 和-o wide

```bash
root@shein-mointor-lab-01:~# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.10.0.1    <none>        443/TCP   35d   <none>
```

```bash
--output-watch-events=false: Output watch event objects when --watch or --watch-only is used. Existing objects are output as initial ADDED events.
```

这个是观察某一个资源的 event 情况。比如：

```bash
root@shein-mointor-lab-01:~# kubectl get pods nginx --watch
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2d2h
```

```bash
-w, --watch=false: After listing/getting the requested object, watch for changes. Uninitialized objects are excluded

if no object name is provided.
```

这个和 tail -f 是一个原理，只要有事件变化就可以追踪，用来监控 pod 的运行情况。

```bash
root@shein-mointor-lab-01:~# kubectl get pods nginx -w
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2d2h
```

```bash
-l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2)
```

主要是用来用 label 进行过滤的。例如：

```bash
root@shein-mointor-lab-01:~# kubectl get pods -A -l name=node-exporter
NAMESPACE      NAME                  READY   STATUS    RESTARTS   AGE
kube-monitor   node-exporter-pfw4v   1/1     Running   0          3d11h
kube-monitor   node-exporter-zh2p6   1/1     Running   0          3d11h
```

```bash
--show-labels=false: When printing, show all labels as the last column (default hide labels column)
```

而这个选项就是把所有的标签显示出来。

```bash
root@shein-mointor-lab-01:~# kubectl get pods -A --show-labels
NAMESPACE       NAME                                           READY   STATUS    RESTARTS   AGE     LABELS
default         nginx                                          1/1     Running   0          2d2h    disk=ssd
ingress-nginx   ingress-nginx-controller-69fb496d7d-f9xjz      1/1     Running   1          3d12h   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,pod-template-hash=69fb496d7d
kube-monitor    node-exporter-pfw4v                            1/1     Running   0          3d11h   controller-revision-hash=5656df5854,name=node-exporter,pod-template-generation=1
kube-monitor    node-exporter-zh2p6                            1/1     Running   0          3d11h   controller-revision-hash=5656df5854,name=node-exporter,pod-template-generation=1
kube-monitor    prometheus-6978748887-4fxxv                    1/1     Running   1          3d12h   app=prometheus,pod-template-hash=6978748887
kube-ops        jenkins2-6fdbbc868d-lc46q                      1/1     Running   0          2d12h   app=jenkins2,pod-template-hash=6fdbbc868d
kube-system     calico-kube-controllers-76d4774d89-6tg9v       1/1     Running   2          38d     k8s-app=calico-kube-controllers,pod-template-hash=76d4774d89
kube-system     calico-node-65tsz                              1/1     Running   1          38d     controller-revision-hash=56c64ccfb5,k8s-app=calico-node,pod-template-generation=1
kube-system     calico-node-gzh5l                              1/1     Running   2          38d     controller-revision-hash=56c64ccfb5,k8s-app=calico-node,pod-template-generation=1
kube-system     coredns-7ff77c879f-ql8dh                       1/1     Running   2          38d     k8s-app=kube-dns,pod-template-hash=7ff77c879f
```

```bash
--sort-by='': If non-empty, sort list types using this field specification.  The field specification is expressed as a JSONPath expression (e.g. '{.metadata.name}'). The field in the API resource specified by this JSONPath expression

must be an integer or a string.
```

题目中出现的问题，这个是指定按照某列进行排序。比如：

```bash
root@shein-mointor-lab-01:~# kubectl get pv --sort-by=.metadata.name
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                              STORAGECLASS                   REASON   AGE
nfs-pv-kube-monitor                        100Gi      RWX            Retain           Bound    kube-system/nfs-pvc-kube-monitor   nfs-storageclass-provisioner            6d10h
nfs-pv-kube-ops                            100Gi      RWX            Retain           Bound    kube-system/nfs-pvc-kube-ops       nfs-storageclass-provisioner            2d12h
pvc-40cb9bc4-c533-4851-885e-7bc501474254   20Gi       RWX            Retain           Bound    kube-ops/opspvc                    kube-ops                                2d12h
pvc-d77b1164-2edd-4025-b691-8907db70cdaf   20Gi       RWX            Delete           Bound    kube-monitor/monitor               kube-monitor                            6d10h
```

其实不加 sort-by 默认就是名字排序，但是考试的时候可能会用别的字段排序，一定看好题目。比如，按照创建时间排序。

```bash
root@shein-mointor-lab-01:~# kubectl get pv --sort-by=.metadata.creationTimestamp
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                              STORAGECLASS                   REASON   AGE
nfs-pv-kube-monitor                        100Gi      RWX            Retain           Bound    kube-system/nfs-pvc-kube-monitor   nfs-storageclass-provisioner            6d10h
pvc-d77b1164-2edd-4025-b691-8907db70cdaf   20Gi       RWX            Delete           Bound    kube-monitor/monitor               kube-monitor                            6d10h
nfs-pv-kube-ops                            100Gi      RWX            Retain           Bound    kube-system/nfs-pvc-kube-ops       nfs-storageclass-provisioner            2d12h
pvc-40cb9bc4-c533-4851-885e-7bc501474254   20Gi       RWX            Retain           Bound    kube-ops/opspvc                    kube-ops                                2d12h
```

## 答案

```bash
root@shein-mointor-lab-01:~# kubectl get pv --sort-by=.metadata.name
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
nfs-pv-kube-monitor 100Gi RWX Retain Bound kube-system/nfs-pvc-kube-monitor nfs-storageclass-provisioner 6d10h
nfs-pv-kube-ops 100Gi RWX Retain Bound kube-system/nfs-pvc-kube-ops nfs-storageclass-provisioner 2d12h
pvc-40cb9bc4-c533-4851-885e-7bc501474254 20Gi RWX Retain Bound kube-ops/opspvc kube-ops 2d12h
pvc-d77b1164-2edd-4025-b691-8907db70cdaf 20Gi RWX Delete Bound kube-monitor/monitor kube-monitor 6d10h
```

