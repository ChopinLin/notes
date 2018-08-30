## kubernetes(k8s)基本概念

### 集群管理

1. Master

   集群控制节点，通常占用一个独立的服务器，高可用建议用3台。Master 节点需要启用 etcd 服务，用来保存所有资源对象的数据。Master 节点运行的一些关键进程对资源进行管理。

2. Node

   除了 Master 以外的所有节点，都是 Node 。Node 和 Master 一样，可以是物理主机，也可以是虚拟机，是主要的工作负载节点。Node 上运行的工作负载一般来说是容器。当 Node 宕机的时候，其上的负载会被Master 转移到其他 Node 上。Node 节点运行的一些关键进程负责与 Master 协作，负载的创建，管理，启停等工作。

   Node 被正确配置之后，会自动向Master上报机器信息，当 Node 超过一定时间没有上报信息，Master 会认为 Node 不可用，Master 触发 负载迁移的自动流程。 

   

### 资源

1. Pod

   Pod 是kubernetes 最重要最基本的概念。Pod 可以包含多个容器，并且所有的 Pod 都包含一个 Pause 的根容器，这个容器是 kurbernetes 平台的一部分。其他的容器属于用户业务容器。

   Pasue 容器的状态代表了整个 Pod 的状态，也就代表了 Pod 里面所有容器的状态。另外 Pause 容器挂载的Volume 被所有的容器共享。因此 Pause 容器解决了 Pod 存活判断和 容器文件共享问题。

   分为静态 Pod 和 普通 Pod

2. Label

   用于资源选择

3. Replication Controller（RC）

   顾名思义，副本控制器。通过定义 RC ，Kubernetes 实现了 Pod 的数量维持在某一个定义的数值，实现了应用集群的高可用性。减少了运维的工作（监控，故障恢复脚本）。手动删除 RC 时，可以根据需要选择是否同时删除该 RC 创建的实例。注意 Pod 的动态缩放不能选择指定的 Node 进行。

   Kubernetes v1.2 后更名为Replication Set（RS）,支持基于集合的 Label selector。

4. Deployment

   Kubernetes 在 v1.2 后引入的新概念，与 RS 一起服务于 Pod ，更好的解决 Pod 的编排问题。可以看做是 RC 的 一次升级。由于 Pod 的创建，调度，绑定以及在目标 Node 上启动对应的容器需要一定时间。这实际上是一个连续变化的过程。所以 Deployment 被设计为可以随时了解当前 Pod 部署的进度。

5. Horizontal Pod Autoscaler

   HPA 通过分析 RC 所控制的 Pod 的负载变化情况，来确定是否要调整Pod的副本数，用于自动扩缩容。HPA 的特点是满足阈值制动触发，不要要人工干预。目前有两种阈值指标。

   - CPU 使用率（算术平均，与Pod 定义的 Pod Request 有关）
   - 自定义服务指标（TPS,QPS）

6. StatefulSet

   是 Deployment 和 RC 的一个特殊变种。适用于有状态的服务，比如MYSQL，ZooKeeper等复杂中间件。

   - Pod 有稳定，唯一的网络标识
   - 启停顺序是受控制的，操作第n个Pod的时候，前 n-1 个 Pod 状态是确定的
   - 采用持久化存储卷，与Headless Service 配合使用

7. Service

8. Volume

9. Persistent Volume

10. Namespace

11. Annotation



