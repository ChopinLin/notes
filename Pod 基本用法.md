# Pod 

[TOC]

## Pod定义

### 定义



### 静态 Pod



## Pod 容器共享 Volume



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

### Job

### Cronjob

### 自定义

## Init Container

## Pod 升级和回滚

## Pod 扩容和缩容



# Service

