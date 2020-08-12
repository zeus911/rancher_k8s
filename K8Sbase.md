# K8s Base 
### 基础命令
- `kubectl`
    - `kubectl version` 
    - `kubectl cluster-info` 查看集群状态
    - `kubectl get nodes`    查看集群中的机器
    
#### Spec & Status (规格和状态)
每一个K8s对象都包含两个嵌套的对象字段 它们控制着对象的配置: 对象spec和对象status

我们必须提供spec它描述(对象所期望的状态) 从而让其匹配和你所期望的状态
```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # 副本
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```
run `kubectl create -f docs/user-guide/nginx-deployment.yaml --record`
- apiVersion ——指定Kubernetes API的版本
- kind ——你想创建什么类型的对象
- metadata ——有助于唯一标识对象的数据，包括name 字符串，UID和可选的namespace
    
    
#### Label & Selector (标签和选择器)
Service 匹配一组 Pod 是使用 标签(Label)和选择器(Selector), 它们是允许对 Kubernetes 中的对象进行逻辑操作的一种分组原语。标签(Label)是附加在对象上的键/值对，可以以多种方式使用:
- 指定用于开发，测试和生产的对象
- 嵌入版本标签
- 使用 Label 将对象进行分类 
   
    
### Kubernetes 部署
需要创建Kubernetes Deployment配置

Deployment 会指挥K8s创建实例， 创建Deployment后 Kubernetes master 将应用程序实例 调度到集群中各个节点中

创建程序实列后 Kubernetes Deployment 控制器会持续监视这些实例 ，会监控这个实例 提供了一种自我修复机制来解决机器故障维护

```  
kubectl create namespace dev  # 创建一个命名空间
kubectl create deployment bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --namespace=dev  # 拿取一个镜像 部署到namespace dev下
kubectl get -n dev deployments  # 查看部署
kubectl get -n dev pods         # 查看dev 下面的pods
```

#### Pods
Pod 是 Kubernetes 抽象出来的，表示一组一个或多个应用程序容器（如 Docker 或 rkt ），以及这些容器的一些共享资源。这些资源包括:
- 共享存储，当作卷
- 网络，作为唯一的集群 IP 地址
- 有关每个容器如何运行的信息，例如容器映像版本或要使用的特定端口。
- Kubelet，负责 Kubernetes 主节点和工作节点之间通信的过程; 它管理 Pod 和机器上运行的容器。
``` 
kubectl get - 列出资源
kubectl describe 显示有关资源的详细信息
kubectl logs 打印pod和其中容器日志
kubectl exec 在pod中的容器执行命令

kubectl exec -it $POD_NAME bash
```

#### Kubernetes Service 总览
Kubernetes Pod 是转瞬即逝的。 Pod 实际上拥有 生命周期。 当一个工作 Node 挂掉后, 在 Node 上运行的 Pod 也会消亡。 ReplicaSet 会自动地通过创建新的 Pod 驱动集群回到目标状态，以保证应用程序正常运行。
##### 内部网络
- ClusterIP (default) 在集群内部IP公开Service 只能内部集群访问
- NodePort 使用NAT在集群中每个选定的Node的相同的端口上公开Service 使用`Node
IP>: <NodePort> 从集群外部访问Service
- LoadBalancer 在当前云中创建负载均衡器, 并为Service 分配一个固定外部IP
- ExteralName 通过返回带有该CNAME 记录,使用任意名称 (由spec 中`externalName`指定) 公开Server 不能使用代理 需要kube-dns高级版本
``` 
kubectl get pods

kubectl get services # 获取集群当中的服务

kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080  # 暴露端口

kubectl get pods -l label-name       # 通过label 查看 pod
kubectl get services -l label-name   # 通过services查看 services

kubectl label pod $POD_NAME app=v1   # 添加新标签

删除服务
kubectl delete service -l app=v1
```

#### 弹性伸缩
``` 
查看副本数
kubectl get rs
对当前服务进行横向扩展
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```
#### 滚动更新
滚动更新 允许通过使用新的实例逐步更新 Pod 实例，零停机进行 Deployment 更新。新的 Pod 将在具有可用资源的节点上进行调度。
