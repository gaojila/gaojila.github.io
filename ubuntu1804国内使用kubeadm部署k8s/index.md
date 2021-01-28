# ubuntu1804国内使用kubeadm部署k8s


# 安装前准备

## kvm 准备 3 台虚拟机

- kube-master
- kube-node1
- kube-node2

## 关闭防火墙

```bash
sudo ufw disable
```

## 关闭 swap

- 临时关闭

```bash
sudo swapoff -a
```

- 永久关闭

```
/etc/fstab 注释swap行
```

# 安装 docker

## 卸载旧 docker

```bash
sudo apt-get remove docker docker-engine docker.io
```

彻底删除 docker-ce 和 docker-cli

```bash
sudo apt-get autoremove docker-ce
```

## 安装依赖

```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

## 添加 docker 的 gpg key

为了使用 https 访问，使用阿里云加速

```bash
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

## 设置 docker 镜像源

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

更新源

```bash
sudo apt update && sudo apt-get update
```

## 安装 docker 18.06

查看 docker-ce 版本

```bash
apt-cache madison docker-ce
```

安装指定版本

```bash
sudo apt-get install -y docker-ce=18.06.3~ce~3-0~ubuntu
```

## 默认启动

```bash
sudo systemctl enable docker && sudo systemctl start docker
```

## 修改 docker 启动参数，进行加速

```bash
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://1nj0zren.mirror.aliyuncs.com",
        "https://docker.mirrors.ustc.edu.cn",
        "http://f1361db2.m.daocloud.io",
        "https://registry.docker-cn.com"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 安装 kubernets

## 更新 apt 源

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

sudo curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<-'EOF'
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update
```

## 安装 kubectl kubeadm kubelet

在 master 和 node 上都安装

```bash
sudo apt-get install -y kubelet=1.17.3-00 kubeadm=1.17.3-00 kubectl=1.17.3-00
sudo apt-mark hold kubelet=1.17.3-00 kubeadm=1.17.3-00 kubectl=1.17.3-00
```

## 启动 kubelet

```bash
sudo systemctl enable kubelet && sudo systemctl start kubelet
```

## 初始化集群(master)

在国内有些众所周知的原因我们无法 kubeadm 初始化是无法连接到谷歌仓库的
在 master 上执行下面脚本后再初始化

```bash
root@kube-master:~# cat k8s.sh
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
docker pull coredns/coredns:1.6.5

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.3 k8s.gcr.io/kube-apiserver:v1.17.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3 k8s.gcr.io/kube-controller-manager:v1.17.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.3  k8s.gcr.io/kube-scheduler:v1.17.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3  k8s.gcr.io/kube-proxy:v1.17.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1  k8s.gcr.io/pause:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0  k8s.gcr.io/etcd:3.4.3-0
docker tag coredns/coredns:1.6.5  k8s.gcr.io/coredns:1.6.5

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
docker rmi coredns/coredns:1.6.5
```

开始初始化

```bash
root@kube-master:~# kubeadm init --kubernetes-version v1.17.3 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
```

保留初始化最后的 token(在添加 node 时使用)

```bash
kubeadm join 192.168.1.6.554:6443 --token ekw68p.rzmiyxxviu1pqu80 --discovery-token-ca-cert-hash sha256:45234a6478890244c7ddedb49b88c4aac5a745c277c7221efbcf7d622788f798
```

## 安装网络插件(flannel)

下载配置文件

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

更改配置文件

```diff
root@kube-master:/var/tmp/0206# diff -u kube-flannel.yml ~/kube-flannel.yml
--- kube-flannel.yml    2020-02-06 14:05:30.240084193 +0000
+++ /root/kube-flannel.yml      2020-02-06 08:05:43.654959409 +0000
@@ -169,7 +169,7 @@
       serviceAccountName: flannel
       initContainers:
       - name: install-cni
-        image: quay.io/coreos/flannel:v0.11.0-amd64
+        image: lizhenliang/flannel:v0.11.0-amd64
         command:
         - cp
         args:
@@ -183,7 +183,7 @@
           mountPath: /etc/kube-flannel/
       containers:
       - name: kube-flannel
-        image: quay.io/coreos/flannel:v0.11.0-amd64
+        image: lizhenliang/flannel:v0.11.0-amd64
         command:
         - /opt/bin/flanneld
         args:
```

安装网络插件

```bash
kubectl apply -f kube-flannel.yml
```

检查 master 状态(ready 为成功)

```bash
root@kube-master:~# kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
kube-master   Ready    master   6h12m   v1.17.3
```

## 加入 node

node 执行下面脚本

```bash
root@kube-node-1:~# cat k8s-node.sh
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3  k8s.gcr.io/kube-proxy:v1.17.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1  k8s.gcr.io/pause:3.1

docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
```

## node 加入集群

```bash
kubeadm join 192.168.1.6.554:6443 --token ekw68p.rzmiyxxviu1pqu80 --discovery-token-ca-cert-hash sha256:45234a6478890244c7ddedb49b88c4aac5a745c277c7221efbcf7d622788f798
```

## master 上检查集群状态

```bash
root@kube-master:~# kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
kube-master   Ready    master   6h12m   v1.17.3
kube-node-1   Ready    <none>   5h48m   v1.17.3
kube-node-2   Ready    <none>   5h44m   v1.17.3
root@kube-master:~# kubectl get pods -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-2qq7p              1/1     Running   1          6h18m
coredns-576cbf47c7-7xl9z              1/1     Running   1          6h18m
etcd-kube-master                      1/1     Running   1          6h9m
kube-apiserver-kube-master            1/1     Running   1          6h9m
kube-controller-manager-kube-master   1/1     Running   1          6h9m
kube-flannel-ds-amd64-fzhft           1/1     Running   0          5h55m
kube-flannel-ds-amd64-p8cpp           1/1     Running   1          6h10m
kube-flannel-ds-amd64-v4688           1/1     Running   1          5h51m
kube-proxy-5dnzv                      1/1     Running   1          6h18m
kube-proxy-hzksv                      1/1     Running   1          5h51m
kube-proxy-rtwtc                      1/1     Running   0          5h55m
kube-scheduler-kube-master            1/1     Running   1          6h9m
```

