用kubeadm 安装时，用 docker-ce 20.10 (k8s 最后支持的版本 )，k8s 版本为 1.23

```sh
yum list docker-ce --showduplicates | grep "20.10"
```

minikube start --driver=docker

![image-20250421005642157](C:\Users\95883\AppData\Roaming\Typora\typora-user-images\image-20250421005642157.png)

### 核心概念

- Pod
  - 最小部署单元
  - 一组容器的集合
  - 内部容器共享网络
  - 默认启动 pause 根容器，用来共享网络和磁盘
  - DaemonSet，每个节点会运行一个这样的Pod，适合 日志收集，监控，分布式存储
  - Cron Job
  - 静态 pod，不用 kubelet 管理，运行在 master 节点
    - etcd，为 apiserver 提供服务
    - apiserver 管理 k8s 资源，只依赖 etcd
    - controller-manager 只依赖 apiserver
    - kube-scheduler 只依赖 apiserver
- Controller
  - 创建 Pod
  - StatefulSet 
  - 无状态
  - 一次性任务
  - 定时任务
  - Deployment 滚动给更新，版本回退
  - ReplicaSet 由 Deployment 管理
  - DaemonSet 守护进程
- Service
  - 定义一组 Pod 的访问规则
  - server cluster 逻辑概念
- Ingress 七层代理，k8s 默认只支持四层代理
- RBAC
- Helm
- CoreDNS
- ETCD
- Dashboard
- Federation 集群联邦，管理多个 k8s 集群
- ConfigMap 通过共享容器卷来实现文件共享和更新
- Secret 管理密钥，提供控制器拉去镜像时所需密钥



### 环境搭建

- kubeadm

  - master 节点 kubeadm init
  - worker 节点 kubeadmin join master-ip:port
  - 禁止 swap 分区

  ``` shell
  # 关闭防火墙
  systemctl stop firewalld
  systemctl disable firewalld
  
  # 关闭 SELinux
  setenforce 0
  
  # 关闭 swap 分区
  swapoff -a
  
  yum install kubelet kubeadm kubectl
  
  kubectl get pod,svc
  
  ```

  

- 二进制安装，繁琐



### 节点类型

- master
  - API server 所有服务的统一入口，提供 k8s 的 Open API, 数据存到 ETCD
    - 对接 kubectl
    - 对接 WebUI
  - scheduler 调度，选择合适的 worker 部署应用
  - controller manager 管理资源的控制器
  - etcd
- worker
  - kubelet 通过 CRI 管理 Docker 容器，与 master 通信
  - kube-proxy 提供 pod 间网络代理(控制 iptables/ipvs 规则)，支持负载均衡



### 不同的 IP

- NodeIP：Node 节点的 IP 地址，即物理机(虚拟机)的 IP 地址。

  是物理机的IP（或虚拟机IP），代表的是物理节点或者虚拟节点的网络。每个Service都会在Node节点上开通一个端口，外部可以通过 http://NodeIP:NodePort 即可访问 Service 里的 Pod 提供的服务。

- PodIP：Pod 的 IP 地址，即 docker 容器的 IP 地址，也是虚拟 IP 地址。

  是每个 Pod 的 IP 地址，Docker Engine根据 docker 网桥的 IP 地址段进行分配的，通常是一个虚拟的二层网络，Pod的ip经常变化，且在k8s集群外无法访问。特点如下：
  
  1. 同Service下的pod可以直接根据PodIP相互通信。
  
  2. 不同Service下的pod在集群间pod通信要借助于 cluster ip。
  
  3. pod和集群外通信，要借助于node ip。
  
- ClusterIP：k8s 虚拟的 Service 的 IP 地址，也是虚拟 IP 地址。

  Service 的 IP 地址，此为虚拟 IP 地址，只是出现在service的规则当中，外部网络无法 ping 通，只有kubernetes集群内部访问使用。特点如下：

  1. Cluster IP仅仅作用于Kubernetes Service这个对象，并由Kubernetes管理和分配P地址 Cluster。
  2. IP无法被ping，他没有一个“实体网络对象”来响应 Cluster IP只能结合Service。
  3. Port组成一个具体的通信端口，单独的Cluster IP不具备通信的基础，并且他们属于Kubernetes集群这样一个封闭的空间。
  4. 在不同Service下的pod节点在集群间相互访问可以通过Cluster IP。

  <img src="https://mmbiz.qpic.cn/mmbiz_png/XhfhfNAicnn0qSic6Hjqbhh5oAtDWAa1Ta1xibqicEnSwIusGsI1K0ePk58k75wnKePBfxnF7LaQe6diaD22O0usiaOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" style="zoom:80%;" />

    >① 代表外部通过公有云的 LoadBalancer 负载均衡服务访问集群内部服务流程
    >
    >② 代表外部用户直接访问集群内部 Service 的 ClusterIP 访问集群内部服务流程
    >
    >③ 代表集群内部不同 Service 之间的 Pod 服务访问流程
    >
    >④ 代表集群内部同一个 Service 中 Pod 服务之间访问流程



### Ingress Controller

Ingress Controller用于解析Ingress的转发规则，本质上也是一种 Service，需要运行在特定 Pod 中，每个Node中一个即可。

Ingress Controller通过API Server获取Ingress资源的变化，并动态地更新Nginx配置文件，实现HTTP(S)的负载均衡及路由转发。

Ingress Controller收到请求，匹配Ingress转发规则转发到后端Service所对应的Pod，由Pod处理请求。Kubernetes中Service、Ingress与Ingress Controller有着以下关系：

- Service是后端真实服务的抽象，一个Service可以代表多个相同的后端服务。
- Ingress是反向代理规则，用来规定HTTP、HTTPS请求应该被转发到哪个Service所对应的Pod上。
- Ingress Controller是一个反向代理程序，负责解析Ingress的反向代理规则。如果Ingress有增删改的变动，Ingress Controller会及时更新自己相应的转发规则，当Ingress Controller收到请求后就会根据这些规则将请求转发到对应Service的Pod上。

Node -> Service -> nginx [Annotations - Ingress-Nginx Controller](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)

![](https://mmbiz.qpic.cn/mmbiz_png/XhfhfNAicnn0qSic6Hjqbhh5oAtDWAa1TaMaYb85ia5CvmdBkHObcBmwvE845DQx3KribN9Ylznk94XgYdbbpByMQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### KubeSphere





### 日常操作和管理

```shell
# 查看集群配置
kubectl config view

# 查看所有 Pod
kubectl get pod -A

# 查看所有 Node，Sevice, Pod，RC
kubectl get -A node,svc,pod,rc

# 查看所有命名空间
kubectl get pods --all-namespaces

# 命令选项
kubectl options

# 创建 pod
kubectl apply -f pod.yaml


```



#### nginx-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-busybox-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
    - name: busybox
      image: busybox:latest
      imagePullPolicy: IfNotPresent
      command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
      volumeMounts:
        - name: shared-logs
          mountPath: /nginx-logs
  volumes:
    - name: shared-logs
      emptyDir: {}

```

```shell
# 进入容器
kubectl exec -it nginx-busybox-pod -c busybox -- sh

# 通过 busy-box 访问 nginx
wget -qO- localhost:80

# 查看 pod 状态
kubectl describe -f .\test.yml
```



#### ingree-nginx

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-consul
  namespace: default
  annotations: 
    kubernets.io/ingress.class: "nginx"
spec:
  rules:
  - host: consul.my.com.local 
    http:
      paths:
      - path: 
        backend:
          serviceName: consul
          servicePort: 8500

```



#### consul

```yaml
apiVersion: extensions/v1beta1
kind: StatefulSet
metadata:
  name: consul
spec:
  serviceName: consul
  replicas: 3
  template: 
    metadata:
      labels:
        app: consul
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - consul
              topologyKey: kubernetes.io/hostname
      terminationGracePeriodSeconds: 10
      containers:
      - name: consul
        image: consul:latest
        args:
             - "agent"
             - "-server"
             - "-bootstrap-expect=3"
             - "-ui"
             - "-data-dir=/consul/data"
             - "-bind=0.0.0.0"
             - "-client=0.0.0.0"
             - "-advertise=$(PODIP)"
             - "-retry-join=consul-0.consul.$(NAMESPACE).svc.cluster.local"
             - "-retry-join=consul-1.consul.$(NAMESPACE).svc.cluster.local"
             - "-retry-join=consul-2.consul.$(NAMESPACE).svc.cluster.local"
             - "-domain=cluster.local"
             - "-disable-host-node-id"
        volumeMounts:
            - name: data
              mountPath: /consul/data
        env:
            - name: PODIP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
        ports:
            - containerPort: 8500
              name: ui-port
            - containerPort: 8400
              name: alt-port
            - containerPort: 53
              name: udp-port
            - containerPort: 8443
              name: https-port
            - containerPort: 8080
              name: http-port
            - containerPort: 8301
              name: serflan
            - containerPort: 8302
              name: serfwan
            - containerPort: 8600
              name: consuldns
            - containerPort: 8300
              name: server
      volumes:
        - name: data
          hostPath:
            path: /home/data

```

