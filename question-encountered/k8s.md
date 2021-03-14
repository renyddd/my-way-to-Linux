## 理解 Pod
Pod 扮演的是传统部署环境里的“虚拟机”的角色（进程是以组的形势运行的）。再把容器看作是运行在这个“机器”里的用户程序，凡是调度、网络、存储的事情进本都是 Pod 级别的。因此凡是跟容器的 Namespace 相关的属性，一定也是 Pod 级别的。

若使用例如 deployment 控制器的话，其中 template 字段的内容与一个标准的 Pod 对应的 API 定义丝毫不差。

[极客时间 - 深入剖析 Kubernetes](https://time.geekbang.org/column/article/40583)

## k8s pod 的生命周期
[kubernetes.io](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)

[DZone - Birth of a Pod](https://dzone.com/articles/kubernetes-lifecycle-of-a-pod)

![pic](https://cdn-images-1.medium.com/max/1500/1*WDJmiyarVfcsDp6X1-lLFQ.png)

导致 Pod 诞生的一系列事件：

1. kubectl 或者其他任意的 API 客户端向 API server 提交 Pod spec；
2. API server 将该 Pod object 写入 etcd；一旦写入成功，etcd 将会向 API server 发回认可；
3. API server 反应 etcd 中的状态变化；
4. 所有 k8s 组件都通过 watch 持续监视 API server 相关的变更；
5. kube-scheduler 通过他的 watcher 发觉还有一个新的 Pod 还没有绑定任何节点；
6. kube-scheduler 为该 Pod 分配一个 Node 并且更新 API server；
7. 这个更改将会传给 etcd，API server 还会将这个节点的分配，反映到 Pod 对象中；
8. 每个 node 上都有 kubelet 在监视 API server，因此目标节点监视到有一个新 Pod 被分配过来了；
9. Kubelet 运行容器并且向 API server 更新其状态；
10. API server 将该 Pod 持久化存储至 etcd；
11. 一旦 etcd 反馈写入成功，API server 便将该认可发送给 kubelet，表明这个事件已被接受。

[raft - Understandable Distributed Consensus](http://thesecretlivesofdata.com/)


## kubelet 作用整理
[ref - kubenetes.io](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/)

Kubelet 是每个节点上运行的主要 node agent，并负责向 API Server 注册该节点。
Kubelet 通过一段描述 Pod 的 Json 或 Yaml 的 PodSpec 来工作，kubelet 接受一组 API Server 下发的 PodSpec 并且确保这些 spec 中描述的容器健康的运行。

in-action 整理：
1. 向 API server 创建 Node 资源并注册该节点；
2. 持续监控 API server 是否给该节点分配了 Pod 然后启动 Pod 容器；
3. 监控容器运行，持续报告状态、事件；
4. 运行容器存活探针的组件，并且将负责重启容器。


## k8s 组件
[https://kubernetes.io/zh/docs/concepts/overview/components/](https://kubernetes.io/zh/docs/concepts/overview/components/)

control plane:
- API server：必须由此来与集群交互，用于处理内部和外部请求；
- controlelr-manager：控制器管理器是将多个控制器功能合而为一，逻辑上讲，每个控制器都是一个单独的进程，但它们都被编译到同一个可执行文件，并运行在一个进程中：
  - node controller：负责在节点出现故障时通知和响应；
  - replication controller：负责为系统中的每个副本控制器对象维护正确数量的 pod；
  - endpoints controller：填充 endpoints 对象（即加入 service 与 pod）；
  - 服务账户和令牌控制器 service account & token controllers：为新的命名空间创建默认账户和 api 访问令牌；
- scheduler：负责监视新创建的、为指定运行节点的 pod，并选择节点让 pod 在上运行，同时也会考虑单个 pod 资源需求、硬件约束、亲和性等；
- etcd：分布式、可靠的键值存储，用于存储 k8s 中的关键数据。

node 组件:
- kubelet：一个在集群中每个节点上运行的代理，保证容器都运行在 pod 中，kubelet 接受提供给他的 PodSpecs，并确保这些 PodSpec 中描述的容器处在健康状态；
- kube proxy：集群中每个节点上运行的网络代理，


扩展：
- service: https://kubernetes.io/zh/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
- controller-manager: https://juejin.cn/post/6844904021271003150



## k8s device plugin
todo
[设备插件wiki](https://kubernetes.io/zh/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)

[https://feisky.gitbooks.io/kubernetes/content/plugins/device.html](https://feisky.gitbooks.io/kubernetes/content/plugins/device.html)



device plugin 的设计思路，就很像是 Linux 中一切设备皆文件的思路。

## k8s 拓扑管理
[ref](https://kubernetes.io/zh/docs/tasks/administer-cluster/topology-manager/)
> 使用命令行选项 --topolgy-manager-scope=pod 来启动 kubelet，就可以选择 pod 作用域。
该作用域允许把一个 pod 里的所有容器作为一个分组，分配到一个共同的 NUMA 节点集。也就是，拓扑管理器会把一个 pod 当成一个整体，并且试图把整个 pod（所有容器）分配到一个单个的 NUMA 节点。

> pod 作用域与 single-numa-node 拓扑管理器策略一起使用，对于延时敏感的工作负载，或者对于进行 IPC 的高吞吐量应用程序，都是特别有价值的。结合两个选项，你可以把一个 pod 里的所有容器都放到一个单个的 NUMA 节点，使得该 pod 消除了 NUMA 之间的通信开销。

## k8s 集群的高可用
[多 master 中各组件的选举机制 ref](https://cloud.tencent.com/developer/article/1450346)
node 其实就已经是在做高可用了。

controller-manager 和 scheduler 仅需要运行多份实例即可，这两个服务是有状态的，会通过向 apiserver 中的 endpint 加锁的方式来进行 leader selection，当目前的 leader 实例无法正常工作时，别的实例会拿到锁成为新的 leader。

选举是通过在 kube-system 命名空间中创建一个与程序同名的 endpoint 资源对象来实现，各实例通过 apiserver 去获取 endpoint 状态，通过竞争的方式去抢占指定的 endpoint 资源锁，胜利者为 leader。

```bash
]# kubectl get endpoints -n kube-system  
NAME                      ENDPOINTS                           AGE
kube-controller-manager   <none>                              23d
kube-scheduler            <none>                              23d
```

apiserver 的高可用：使用外部 lb，在 node 配置中也使用域名配置。


扩展：[创建高可用集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/)


## k8s Pod spec 更新冲突
todo














