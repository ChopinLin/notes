#  kubernetes 运维

## Node 隔离与恢复

当集群硬件升级维护的时候，如果需要让某个 Node 脱离 k8s 集群的管理。可以使用Node 隔离机制。当然也可以恢复Node的调度

- 文件方式

  在Node 的定义文件中，加上 spec.unschedulable: true 属性。然后使用

  ```kubectl replace -f node.yaml``` 修改Node 状态。

- 命令方式

  ```shell
  kubectl patch node <nodeName> -p '{"spec":{"unschedulable":true}}'
  ```

需要注意的是，Node 虽然脱离了k8s 的托管，但是Node 上的Pod 并不会停止运行。需要手动停止。需要恢复调度时，可以将属性值设置为 false 。

另外还可以直接使用 cordon 子命令直接进行隔离与恢复

```shell
kubectl cordon <nodeName>
kubectl uncordon <nodeName>
```

## Node 添加

对于一台新的机器，要加入现有集群，需要机器上安装 容器运行时（docker）、kubectl、kube-proxy 服务。在kubectl 、kube-proxy 服务启动时，要指定 Master Url。kubectl 启动以后就会自动向加群注册当前Node 的信息。



## Label管理

```shell
#增加
kubectl label pod <podName> key=value
#查询 -Lkey 没有空格，按key 查找。可以把pod 换成其他资源
kubectl get pods -Lkey
kubectl get pods -Lkey -n naespace
#修改
kubectl label pod <podName> key=value --overwrite
#删除 key后面加 减号-，表示删除key
kubectl label pod <podName> key-

```

## namespace 与 context

通过 prd.yaml 文件定义一个 namespace

```yaml
apivesion: v1
kind: namespace
metadata:
  name: prd
```

通过 ```kubectl create -f prd.yaml```创建

对于某个namespace，可以为其关联context ,即不同的运行上下文，也可以理解为运行环境。不同运行环境之间创建的资源互不影响，互不可见。

```shell
kubectl config current-context
kubectl config get-contexts
kubectl config set-context
```

## 资源管理



## 排查问题

```shell
#当Pod 启动出问题的时候，查看Pod 的启动事件 Event 可以快速定位问题
#主要关注Events,deascribe 也可以查看其他资源
kubectl describe pod <podName>

#当Pod 只有一个容器时（不算 Pause）,直接查看容器日志
kubectl logs <podName>
#多个容器使用 -c 指定
kubectl logs <podName> -c <containerName>
#与上一个命令等效
docker logs <containerId>

#k8s 服务日志
#如果 用 systemctl 托管了k8s 可以使用 systemctl 命令直接查看
#查看 kube-controller-manager 日志
systemctl status kube-controller-manager -l

```

如果k8s 没有被其他服务托管，可以直接在配置的日志路径查看相关日志。

例如，如果是Pod 启动失败，可以先定位Pod 调度的节点，然后直接登录到节点上，查看 kubelet 的日志。直接通过Pod 名字查询。

如果是扩容或者RC的相关问题，则应该在master 上查看 kube-controller-manager 或者 kube-schduler 日志。直接搜索 资源名查询。

如果是服务访问异常，应该关注防火墙（iptable）或者是 kub-proxy 的服务日志

