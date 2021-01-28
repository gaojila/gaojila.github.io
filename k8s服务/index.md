# k8s-service


## Service 的类型

- **ClusterIP：** 默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP。
- **NodePort：** 在 ClusterIP 的基础上为 service 在每台机器撒谎那个绑定一个端口，这样就可通过 NodePort 来访问服务。
- **LoadBalancer：** 在 NodePort 的基础上，借助 cloud provider 创建一个外部负载均衡器，并将请求转发到 NodePort。
- **ExternalName：** 把集群外部的服务引入到集群内部来，在集群内部直接使用，没有任何类型代理被创建。

