# k8s 网络

## k8s 网络模型

IP-per-Pod 模型。从宏观上看，每一个Pod可以看成一个独立的“虚拟机”或者“物理机”，Pod 中的容器相当于 机器中的进程。IP-per-Pod 模型 有隐含的几层含义。

- 同一个Pod内的容器共享一个网络命名空间。
- Pod 内容器看到自己的Ip和端口与其他Pod看到的一样。

k8s 对集群的网络要求

- 所有容器都可以不用NAT方式与其他容器通信
- 所有节点都可以不用NAT方式与其他容器通信，反之亦然
- 容器的地址和其他应用看到的地址一样

搭建私有集群的时候需要通过一些开源的网络组件实现 k8s 要求的网络环境。

## Docker 网络基础







