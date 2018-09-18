# docker command

```shell
#运行jenkins 镜像
docker run --name lxpjenkins -d -e user=docker -v /home/docker/jenkins:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts

docker run --name lxpjenkins -d -e user=docker -v /mnt/sda1/docker-u
ser/jenkins:/var/jenkins_home -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts

#获取容器/镜像的元数据。
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mymysql

#官方文档中说attach后可以通过CTRL-C来detach，但是如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。
#好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。
docker attach --sig-proxy=false mynginx

docker images
docker rm $(docker ps -a -q)

# 在容器中执行
docker exec -i -t  lxpjenkins /bin/bash

#通过运行容器生成镜像 -p 生成镜像时暂停容器,不会提交映射volume 的内容
docker commit -a author -p -m "commit msg" lxpjenkins

#--rm 容器退出后会删除，所以不设置名字也无所谓
docker run --rm -it -p 8080:8080 -p 50000:50000 lxpjenkins /bin/bash

docker build -t image:2.2 . -f Dockerfile
docker save -o xxx image:2.2
docker load < xxx
docker tag image:2.2 repo/image:2.2

```



# kubernates

```shell
#kubectl
#创建资源
kubectl create -f my-ser.yaml
#创建目录下所有 yanml文件定义资源
kubectl create -f <dir>

#查看pods/rc/service 列表
kubectl get nodes，pods，rc，service，deployment
#输出更详细的信息，包括 pod ip 和所在node ip
kubectl get pods -o wide
#详细yaml配置
kubectl get pods -o yaml
#查看所有rs deploy svc pod
kubectl get all --all-namespaces=true  
#更新
kubectl replace pod.yaml


#查看xx的详细信息
kubectl describe pod <pod-name>
kubectl describe node <node-name>

#执行pod内指定容器命令
kubectl exec -it <pod> -c <container> /bin/bash

#pod 调度
# 给 node 打标签
kubectl label nodes <node-name> <label-key>=<label-value>
kubectl label nodes 10.89.32.15 name=cat


# deployment部分 使用deployment升级会生成新的rc 和 pod
kubectl edit deploy/nginx #修改deploy的yaml文件 会触发升级
kubectl set image deploy/nginx nginx=nginx:1.9.1 #修改deploy的镜像 直接修改会触发滚动升级
kubectl scale deploy/nginx --replicas 5 #扩容或者缩容量
kubectl rollout status deployments nginx #查看更新状态
kubectl rollout history deploy/nginx  #查看历史版本列表
kubectl rollout history deploy/nginx --revision=x #查看具体历史版本
kubectl rollout undo deploy/nginx #回滚到上一个部署版本
kubectl rollout undo deploy/nginx --to-revision=x #回滚到某一个版本号
kubectl rollout pause deploy/nginx #暂停更新操作 配置修改完再进行恢复  可以避免因为频繁改动配置导致频繁升级
kubectl rollout resume deploy/nginx #恢复更新操作

#用kubectl rolling-update 对rc进行滚动升级
kubectl rolling-update redis-master -f redis-master-controller-v2.yaml
kubectl rolling-update redis-master --image=redis-master:2.0 #直接修改镜像进行升级
kubectl get xxx -o yaml #获取资源对应的yaml文件
kubectl replace -f xxx.yaml  #通过替换yaml文件进行更新
```





```shell
docker run -d -p 5000:5000 --restart always --name registry registry:2

docker build -t lsf:2.2 . -f Dockerfile
docker save -o xxx lsf:2.2
docker load < xxx
docker tag lsf:2.2 localhost:5000/lsf:2.2

sudo docker login --username=100003956454 ccr.ccs.tencentyun.com
password mucfc1234
docker push localhost:5000/lsf:2.2
docker pull localhost:5000/lsf:2.2

docker run --name lsf -d -v /app/ces/lsf-core/logs:/app/ces/lsf-core/logs -v /app/ces/lsf-messaging/logs:/app/ces/lsf-messaging/logs -p 29000:29000 -p 29010:29010 lsf:2.2 /app/ces/bin/start.sh

docker ps -a|grep lsf-messaging

docker inspect lsf-messaging

docker stop lsf-messaging

docker rm lsf-messaging

docker run -i -t xxxxx /bin/bash
```

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-storage

https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/

```yaml
#grace shutdown https://pracucci.com/graceful-shutdown-of-kubernetes-pods.html
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
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
        lifecycle:
          preStop:
            exec:
              # SIGTERM triggers a quick exit; gracefully terminate instead
              command: ["/usr/sbin/nginx","-s","quit"]
      terminationGracePeriodSeconds: 60   

```


```yaml
#statefulSet tencent example
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1beta2
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
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
          name: web
        volumeMounts:
        - name: storage
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: storage
      annotations:
        volume.beta.kubernetes.io/storage-class: cbs
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
```