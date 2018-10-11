# Pod 

[TOC]

## Pod定义

### 定义

```yaml
# yaml格式的pod定义文件完整内容：
apiVersion: v1        　　#必选，版本号，例如v1
kind: Pod       　　　　　　#必选，Pod
metadata:       　　　　　　#必选，元数据
  name: string        　　#必选，Pod名称
  namespace: string     　　#必选，Pod所属的命名空间
  labels:       　　　　　　#自定义标签
    - name: string      　#自定义标签名字
  annotations:        　　#自定义注释列表
    - name: string
spec:         　　　　　　　#必选，Pod中容器的详细定义
  containers:       　　　　#必选，Pod中容器列表
  - name: string      　　#必选，容器名称
    image: string     　　#必选，容器的镜像名称
    imagePullPolicy: [Always | Never | IfNotPresent]  #获取镜像的策略 Alawys表示下载镜像 IfnotPresent表示优先使用本地镜像，否则下载镜像，Nerver表示仅使用本地镜像
    command: [string]     　　#容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      　　 #容器的启动命令参数列表
    workingDir: string      #容器的工作目录
    volumeMounts:     　　　　#挂载到容器内部的存储卷配置
    - name: string      　　　#引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string     #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean     #是否为只读模式
    ports:        　　　　　　#需要暴露的端口库号列表
    - name: string      　　　#端口号名称
      containerPort: int    #容器需要监听的端口号
      hostPort: int     　　 #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string      #端口协议，支持TCP和UDP，默认TCP
    env:        　　　　　　#容器运行前需设置的环境变量列表
    - name: string      　　#环境变量名称
      value: string     　　#环境变量的值
    resources:        　　#资源限制和请求的设置
      limits:       　　　　#资源限制的设置
        cpu: string     　　#Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string      #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests:       　　#资源请求的设置
        cpu: string     　　#Cpu请求，容器启动的初始可用数量
        memory: string      #内存清楚，容器启动的初始可用数量
    livenessProbe:      　　#对Pod内个容器健康检查的设置，当探测无响应几次后将自动重启该容器，检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
      exec:       　　　　　　#对Pod容器内检查方式设置为exec方式
        command: [string]   #exec方式需要制定的命令或脚本
      httpGet:        　　　　#对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:      　　　　　　#对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0   #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0    　　#对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0     　　#对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
    restartPolicy: [Always | Never | OnFailure] #Pod的重启策略，Always表示一旦不管以何种方式终止运行，kubelet都将重启，OnFailure表示只有Pod以非0退出码退出才重启，Nerver表示不再重启该Pod
    nodeSelector: obeject   　　#设置NodeSelector表示将该Pod调度到包含这个label的node上，以key：value的格式指定
    imagePullSecrets:     　　　　#Pull镜像时使用的secret名称，以key：secretkey格式指定
    - name: string
    hostNetwork: false      　　#是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
    volumes:        　　　　　　#在该pod上定义共享存储卷列表
    - name: string     　　 　　#共享存储卷名称 （volumes类型有很多种）
      emptyDir: {}      　　　　#类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      hostPath: string      　　#类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
        path: string      　　#Pod所在宿主机的目录，将被用于同期中mount的目录
      secret:       　　　　　　#类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
        scretname: string  
        items:     
        - key: string
          path: string
      configMap:      　　　　#类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
        name: string
        items:
        - key: string
          path: string 
```

- Pod包含的容器要保持在前台运行。（可以使用 supervisor 辅助或者参考开源镜像实现）
- Pod 包含一个或者多个容器，其中所有Pod 都会有一个 Pause容器，代表了Pod 的生命周期。
- Pod 里的容器可以通过 localhost  互相访问。 

### 静态 Pod

由 kubelet 管理的 仅存于特定Node 上的Pod 。不能通过Yaml文件管理。总是由kubelet 创建，并在kubelet 所在的Node 上运行。有两种创建方式。

- 配置文件

  在kubelet 启动时，添加参数 –config=path。 kubelet 会定期扫描该目录下的 `.json` `.yaml`文件。创建Pod。

- HTTP 方式

  --manifest-url 指定URL，kubelet 会定期获取并以`.json` `.yaml`文件格式解析。

## Pod 容器共享 Volume

spec.containers.volumeMounts.name 指定挂在相同的Volume。则同一Pod内不同容器使用共享该 Volume。

apec.volumes 指定volume 类型。

## Pod 配置管理

### ConfigMap 资源

- 生成为容器内的环境变量
- 设置容器启动命令的启动参数（设为环境变量）
- 以 Volume 的形式挂在为容器内的文件或目录

### ConfigMap 创建

- yaml 方式

- kubectl 命令方式

  --form-file

  --form-literal

## Pod使用ConfigMap

- valueForm

  configMapKeyRef

- envFrom

  configMapRef

- volumes

  configMap

## 容器内获取 Pod  的信息

容器定义处于 Pod 下级，k8sDownward API 获取容器所在 Pod的信息。

包括 metadata.name,metadata.namespace 等

- 环境变量：用于单个变量，将 Pod 信息注入 Container 内
- Volume 挂载将数组类信息挂载到容器内

## Pod生命周期检测、重启策略、健康检查

### Pod 状态值

- Pending
- Running
- Succeeded
- Failed
- Unknown

### 重启策略

- Always
- Onfailure
- Never

不同控制器对 Pod 有不同的重启策略要求。

### Pod健康检查的两种方式

- LivenessProbe 探针

  判断是否 Running，主要有三种实现

  ``` yaml
  #ExecAction 执行命令，返回 0 认为存活,否则杀掉重启。
  livenessProbe: 
    exec: 
      command: 
        -  cat
        - /tmp/health
    initialDelaySeconds: 30 #启动容器后首次执行健康检查的等待时间
    timeoutSeconds: 1 #健康检查的超时时间，超时认为容器不可用，重启
  
  #TcpSocketAction 能够建立Tcp连接，健康
  livenessProbe: 
    tcpSocket: 
      port: 80 
    initialDelaySeconds: 30
    timeoutSeconds: 1
  
  #HttpGetAction 返回200 小于 400 健康
  livenessProbe: 
    httpGet: 
      path: /_status/healthz 
      port: 80
    initialDelaySeconds: 30
    timeoutSeconds: 1    
  ```

- ReadinessProbe 探针

  判断是否Ready



## Pod 调度

根据调度策略完成 Pod 的部署。由kubrnetes Master 的 Schedular 服务进程负责实现 Pod 的调度。通常我们无法知道 Pod 被调度到了哪个 Node 上。k8s 也提供了一些调度策略的设置，可以让用户完成对Pod 的精准调度。

### Deployment/RC 全自动调度

利用 Deployment/RC 创建 ReplicaSet。**ReplicaSet与Deployment/RC 关系对比 java实例与类**



### NodeSelector 定向调度(可能弃用)

首先需要对Node 设置标签，Pod 通过 nodeSelector 属性匹配相应的标签将 Pod 调度到对应的Node 上。



### Affinitity 调度（灵活性比较强大的调度设置）

有两个维度，分为 Node Affinity 和 Pod Affinity。

可以有优先级，当没有满足优先需求的情况下，会退而求其次。

可以根据节点上 已经在运行的 Pod 来调度，而不是只根据节点标签，可以实现Pod之间的互斥。



#### Node Affinity

目前有两种亲和性的表达。

- RequireDuringSchedulingIgnoredDuringExecution

  必须满足指定的规则，NodeSelector 定向调度的替代者。相当于强约束。

- PreferDuringSchedulingIgnoredDuringExecution

  强调优先满足指定规则，尝试调度，但是不强求。属于软约束。多个优先级可以设置权重，以自定义匹配的先后顺序。

IgnoredDuringExecution，意思是在Pod已经调度到某节点后，如果节点的 lable 被修改后。忽略节点label的变化。 可以在Pod的定义上增加以上属性，实现亲和性调度。

规则设置的注意事项：

当 nodeSelector 和   nodeAffinity 同时出现时，必须都满足。

nodeAffinity 中 若指定了多个 nodeSelectorTerms，则满足一个即可。

nodeSelectorTerms 若有多个 matchExpressions 则必须都满足。通过operator 属性的 NOtIn,DoesNotExist 来实现互斥。

#### Pod Affinity

不仅根据Node来调度，还可以根据Node上的Pod来调度。这种规则的一个简单描述：如果在一个具有 X 标签的Node上运行了一个或者多个符合条件 Y 的 Pod，那么被调度的 Pod 应该运行（拒绝运行，互斥）在这个Node上。Pod 亲和性也有两种表达

- RequireDuringSchedulingIgnoredDuringExecution
- PreferDuringSchedulingIgnoredDuringExecution

Node 选择可以用 k8s 内置的Node 标签 。下文是一些例子，分别从主机，机架，区域维度来选择。可以使用自定义的标签。

```yaml
topologyKey： kubernetes.io/hostname 
topologyKey： failure-domai.beta.kubernetes.io/zone 
topologyKey： failure-domai.beta.kubernetes.io/region 
```

规则设置的注意事项：

不同于NodeAffinity 亲和与互斥都用 nodeaffinity 中，同过operator 区分。pod互斥与亲和在不同属性中。

pod 亲和调度用 podAffinity 属性，pod 互斥调度用 podantiAfinity 属性。

### Taints 与 Tolerations

理解 Taint 需要与NodeAffinity 进行对比。NodeAffinity 是在Pod上定义的属性，使得Pod被调度到某个Node上。而Taint 是在Node上定义的属性。让 Node 拒绝 Pod 的运行（就算正在Node上运行后新加了 Taint，也会被驱逐）。

Taint 需要与 Tolerations 配合使用。

在Node上申明了一个或者多个Taint 后，如果Pod 不使用 Tolerations 去容忍相应的 Taint,Pod

将无法在相应的Node 上运行。Tolerations 是Pod 的属性，相当于申明了 Taint 的Node 节点的通行证。

 常见的用例：

- 让某些特殊节点只给特定的应用使用
- 定义Pod 驱逐行为，应对节点故障。

### DaemonSet

用于管理每个Node上仅运行一份Pod的副本实例。除了试用默认的调度策略之外，还可以使用 NodeSelector 或者 NodeAffinity来指定满足条件的 Node 进行调度。

试用这些需求的应用：

- 每个 Node 上运行一个 GlusterFs 或者 Ceph 的存储的Daemon 进程
- 每个 Node 上运行一个 日志采集程序 Logstah 或者 Fluented
- 每个 Node 上运行一个 性能监控程序，采集Node的运行性能数据，例如 Prometheus ，collectd 等

### Job

k8s 从1.2版本开始支持批处理类型的应用，可以通过kubernetes Job资源对象来定义并启动一个批处理任务。批处理任务通常并行（或串行）启动多个计算进程去处理一批工作项（work item），处理完后，整个批处理任务结束。

批处理按任务实现方式不同分为以下几种模式：

- **Job Template Expansion模式** 
  **一个Job对象对应一个待处理的Work item**，有几个Work item就产生几个独立的 Job，适用于Work item数量少，每个Work item要处理的数据量比较大的场景。例如有10个文件（Work item）,每个文件（Work item）为100G。

  写一个模板的 job.yaml，根据参数不同，生成不同的 job.yaml 执行不同的逻辑。这种模式跟下 Job 与 Pod 没有对应关系

- **Queue with Pod Per Work Item** 
  采用一个任务队列存放Work item，一个 Job 对象作为消费者去完成这些Work item，其中 Job 会启动N个Pod，**每个Pod对应一个Work item。**

  在消息队列存放 work Item。然后将编写好的处理程序（worker）打包成镜像并定义成为Job中的work Pod。work Pod 从消息队列中拉取 任务执行，处理完成后进程结束。Job 启动下一个Pod 去拉取下一个作业。任意时刻Pod的数量固定。

- **Queue with Variable Pod Count** 
  采用一个任务队列存放Work item，一个Job对象作为消费者去完成这些Work item，其中Job会启动N个Pod，**每个Pod对应一个Work item。但Pod的数量是可变的**。

  在消息队列存放 work Item。然后将编写好的处理程序（worker）打包成镜像并定义成为Job中的work Pod。work Pod 从消息队列中拉取 任务执行。与 上一种模式的区别主要在于，work Pod 在执行完work之后会继续去拉取work item。并不会退出。

考虑到批处理的并行的问题。k8s 将Job分为三种类型。通过并行Pod 的正常结束数量判断 Job 是否完成。

- Non-parallel Jobs 

  通常一个Job只启动一个Pod,除非Pod异常才会重启该Pod,一旦此Pod正常结束，Job将结束。 

- Parallel Jobs with a fixed completion count

  并行Job会启动多个Pod，此时需要设定Job的.spec.completions参数为一个正数，当正常结束的Pod数量达到该值则Job结束。 

- Parallel Jobs with a work queue

  任务队列方式的并行 Job 需要一个独立的Queue，Work item都在一个Queue中存放，不能设置 Job的.spec.completions参数。

  此时Job的特性：

  - 每个Pod能独立判断和决定是否还有任务项需要处理
  - 如果某个Pod正常结束，则 Job 不会再启动新的 Pod
  - 如果一个Pod 成功结束，则此时应该不存在其他 Pod 还在干活的情况，它们应该都处于即将结束、退出的状态
  - 如果所有的 Pod 都结束了，且至少一个 Pod 成功结束，则整个Job算是成功结束

### Cronjob

类似Linux CronTab 的 定时任务

k8s v1.5以上支持。在API Server 启动进程添加以下配置，并重启。才能支持CronJob.

–runtime-config=batch/v2alpha1=true

schedule 属性定义Cron 表达式。（与Linux 表达式有区别，第一位是 Minute 而不是 Second）

### 自定义

- 用任何语言实现一个调度器（shell 也可以）并启动。

- 在Pod 的 spec: 下添加 schedulerName: 属性。

  ```yaml
  apiVersion: v1
    kind: Pod
    metadata:
    xxx: xx
    spec:
      schedulerName: my-scheduler
      containers:
        xxx
  ```

  

## Init Container

应用场景：在应用启动前需要进行的初始化操作都可以放到 Init Container 容器中执行。

- 基于环境变量或者配置模板生成配置文件
- 下载依赖包或者预配置
- 等待其他主件的正确运行

与一般的 Containers 配置不同，用 initContainers 配置 。

Init Container 与其他的容器本质上是一样的。但是有一些区别

- 先于应用容器运行，仅运行一次就结束。可以有多个Init Container，当且仅当 前一个执行成功执行下一个。
- 资源限制的设置不同
- 不可以设置 readinessProbe 探针

## Pod 升级和回滚

以 Deployment 为例子。

### 升级

```shell
#修改Deployment set 或者 edit
kubectl set image deployment/<deployment-name>
kubectl edit deployment/<deployment-name>
#一旦 deployment 发生了修改，会自动触发Deployment所运行的Pod的升级。
#查看升级状态
kubectl rollout status deployment/<deployment-name>
#查看详细的事件信息（了解如何升级）
kubectl describe deployment/<deployment-name>
```

一些参数：

- spec.strategy.type=Recreate

  杀死所有的Pod，重新创建

- spec.strategy.type=RollingUpdate

  默认策略，滚动逐个跟新，并根据配置配置控制滚动更新过程Pod数量

- spec.strategy.RollingUpdate.maxUnavailable

  控制滚动跟更新时 Pod 不可用的上限数量，可以是百分比或者数量。

- spec.strategy.RollingUpdate.maxSurge

  控制滚动跟更新时新旧 Pod 总量超过期望副本的最大值，可以是百分比或者数量。

多重更新（Rollover：上一次没更新完成，又再次触发更新），每一次更新都会创建一个新的ReplicaSet。上一次更新的操作会停止，已经更新的Pod会杀掉。并创建新的Pod。不会等到上一次更新结束再开始本次更新。



### 回滚

```shell
#查看 Deployment 部署历史,--revision=N 指定查看某个版本
kubectl rollout history deployment/<deployment-name> --revision=N

#停止当前的更新,回滚到上一个版本
kubectl rollout undo deployment/<deployment-name>
#回滚到指定版本
kubectl rollout undo deployment/<deployment-name> --to-revision=N

#由于Deployment 修改就会触发更新，可以关闭部署操作，等完成一次复杂修改之后，再开启更新
kubectl rollout pause deployment/<deployment-name>
kubectl edit deployment/<deployment-name> xxx
kubectl edit deployment/<deployment-name> xxx
...
kubectl rollout resume deployment/<deployment-name>
```



RC 的滚动升级通过 kubectl rolling-update 命令实现。逐渐被RS 和 Deployment 所取代。

DaemonSet的更新升级策略：OnDelete、RollingUpdate、Paritioned。 



## Pod 扩容和缩容

有手动触发和自动触发两种。

```shell
# 命令手动触发。N 比当前Pod副本数小，缩容。大，扩容。
kubectl scale deployment <deployment-name> --replicas N
```

HPA 自动扩容

1. MasterKube-controller-manager 服务要加上参数 --horizontal-pod-autoscaler-sync-period 定义时长（默认 30 s）,来周期性的检测Pod CPU使用率，并在满足条件的时候对 RC/Deployment 进行扩缩容。
2. Pod 的CPU 使用率来源于Heapster 组件。所以依赖Heapster 组件。
3. 扩缩容的 RC/Deployment 必须定义 resources.requests.cpu 的值。

- 命令创建 HPA

  ```shell
  kubectl autoscale deployment <name> --min=a --max=b --cpu-percent=1~100
  ```

- 通过 yaml 文件创建HPA

  ```yaml
  apiVersion: autoscaling/v2beta1
  kind: HorizontalPodAutoscaler
  metadata:
    name: podinfo
  spec:
    scaleTargetRef:
      apiVersion: extensions/v1beta1
      kind: Deployment
      name: podinfo
    minReplicas: 2
    maxReplicas: 10
    metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 80
    - type: Resource
      resource:
        name: memory
        targetAverageValue: 200Mi
  ```

# Service

由于Pod的IP不可靠，而且对于分布式应用来说，需要一个负载均衡来实现请求分发。Service是为解决这个问题而设计出的一个核心资源。

Service 为一组相同功能的容器应用提供一个统一的入口地址，并且将请求负载分发到后端的各个应用容器上。主要的内容包括 Service 的负载均衡、外网访问、DNS 服务、Ingress 7层路由机制等。

## 定义

```yaml
apiVersion: v1　　　　　　　　　　　　//必须
kind: Service　　　　　　　　　　　　 //必须
matadata:　　　　　　　　　　　　　　 //必须，元数据
  name: string　　　　　　　　　　　　//必须，service的名称
  namespace: string　　　　　　　　　//非必须，指定的namespace名称，与rc在同一个namespace即可
  labels:　　　　　　　　　　　　　　　//非必须，service的标签
    - name: string
  annotations:　　　　　　　　　　　　　//非必须，service的注解属性信息
    - name: string
spec:　　　　　　　　　　　　　　　　　　//必须，详细描述
  selector: []　　　　　　　　　　　　  //必须，lable selector设置，选择具有执行lable标签的pod作为管理范文
  type: string　　　　　　　　　　　　　//必须，service的类型，指定service的访问方式，默认为cluster ip，用于k8s内部的pod访问， node上的kube-proxy通过设置 iptables 转发。
  # nodePort，使用宿主机的端口，使能够访问各 node 的外部客户通过node的ip地址和端口就能访问服务。
  # loadbalancer： 使用外接负载均衡完成服务到负载的分发，需要在spec.status.loadBalancer字段指定负载均衡器的ip，并同时定义nodePort和clusterIP
  clusterIP: string　　　　　　　　　　//非必须，虚拟服务器ip地址
  sessionAffinity: string　　　　　　 //非必须，是否支持session，可选值为cluster ip
  ports:　　　　　　　　　　　　　　　　//端口　　　　　　　
  - name: string　　　　　　　　　　　　//端口名称　　
    protocol: string　　　　　　　　　　//协议类型
    port: int　　　　　　　　　　　　　　//服务监听的端口号，service clusterip的端口
    targetPort: int　　　　　　　　　　 //需要转发到后端pod的端口
    nodePort: int　　　　　　　　　　　　//container port 映射到宿主机的端口
  status:　　　　　　　　　　　　　　　　 //当type 为loadBalancer时，说这负载均衡的地址
    loadBalancer:
      ingress:　　　　　　　　　　　　　　//外部负载均衡
        ip: string　　　　　　　　　　   //负载均衡器的ip地址
        hostname: string　　　　　　　　 //外部负载均衡的主机名
```

1. 通过命令快速创建

   ```shell
   # k8s 通过 kubectl expose 命令快速创建 service
   kubectl expose resource <service-name>
   kubectl expose rc webapp
   #查看k8s集群服务
   kubectl get svc
   ```

2. 通过yaml文件创建

   ```shell
   #通过 yaml 定义文件创建
   kubectl create -f webapp-svc.yaml
   ```

目前的两种负载均衡策略

- RoundRobin

  轮询模式，**默认**

- SessionAffinity

  基于客户端 IP 地址的会话保持模式。同一个客户端的请求发往同一个后端服务。**非默认**，需设置 service.spec.sessionAffinity=ClientIp 启用该策略。

##默认Service

##外部访问Service

##Headless Service

## 服务域名解析

## Ingress 7层路由



