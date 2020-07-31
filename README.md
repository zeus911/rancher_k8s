# Rancher Kubernetes Study

### 常用部署解决方案
- Kops        生产级别
- Kubespray   生产级别
- Kubeadm    
- Rancher     生产级别

Kops和Kubespary在国外用的比较多，没有处理中国的网络问题，没法使用。

kubeadm是Kubernetes官方提供的k8s部署工具，不过不支持HA，且支持的docker版本、K8S版本也有限，因此无法作为生产级安装程序。

Rancher 是SUSE旗下  非常牛逼的K8s一体化解决方案 为DevOps赋能

### Rancher
- 功能
    - Kubernetes 集群
    - 应用商店
    - 项目管理  Rancher中一个集群多个namespace 和多个 访问控制权限 组成,允许用户以组为单位，一次管理多个命名空
    - CI/CD
    - 内置Istio
    - 小强一般的容灾恢复能力
    - 可视化UI

### Kubernetes 概念
- Docker
    - 运行时系统标准 
- Kubernetes
    - 容器集群管理标准
    - 部署，运维，弹性伸缩，自动化运维
- Kubernetes 集群
    - etcd
        - Rancher Server 数据集群状态存储,K8s集群状态保存
        - 配置三个 etcd 节点就能满足小型集群的需求，五个 etcd 节点能满足大型集群的需求
    - controlplane
        - Kubernetes API server、scheduler 和 controller mananger
        - 无状态
    - worker
        - Kubelet： 监控节点状态的 Agent，确保您的容器处于健康状态。
        - 工作负载： 承载应用和其他类型的部署的容器和 Pod。
- Helm
    - Helm 是 Kubernetes 的软件包管理工具

### Kubernetes 名词解释 相关概念
- 集群(Cluster)
    - 同一子网中一个或多个服务器， 为容器提供计算资源池
- 节点(Node)
    - 主机 一台服务器  ， 容器运行在节点上，结点上运行这Agent 代理程序kebelet, 用于管理容器实例 节点数量可以伸缩
- 实例(Pod)
    - 是K8s部署应用的最小基本单元 一个 Pod 封装多个应用容器（也可以只有一个容器）、存储资源、一个独立的网络 IP 以及管理控制容器运行方式的策略选项。
- 容器(Container)
    - 一个通过 Docker 镜像创建的运行实例，一个节点可运行多个容器。容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。
- 工作负载（Workload）
    - 工作负载即 Kubernetes 对一组 Pod 的抽象模型，用于描述业务的运行载体，包括 Deployment、Statefulset、Daemonset、Job、CronJob 等多种类型。
- 无状态工作负载（Deployment）
    - 即 Kubernetes 中的“Deployments”，无状态工作负载支持弹性伸缩与滚动升级，适用于实例完全独立、功能相同的场景，如：nginx、wordpress 等。
- 有状态工作负载（StatefulSet）
    - 即 Kubernetes 中的“StatefulSets”，有状态工作负载支持实例有序部署和删除，支持持久化存储，适用于实例间存在互访的场景，如 ETCD、mysql-HA 等即 Kubernetes 中的“StatefulSets”，有状态工作负载支持实例有序部署和删除，支持持久化存储，适用于实例间存在互访的场景，如 ETCD、mysql-HA 等
- 守护进程集（DaemonSet）
    - 即 Kubernetes 中的“DaemonSet”，守护进程集确保全部（或者某些）节点都运行一个 Pod 实例，支持实例动态添加到新节点，适用于实例在每个节点上都需要运行的场景，如 ceph、fluentd、Prometheus Node Exporter 等。
- 普通任务（Job）
    - 即 Kubernetes 中的“Job”，普通任务是一次性运行的短任务，部署完成后即可执行。使用场景为在创建工作负载前，执行普通任务，将镜像上传至镜像仓库。
- 定时任务（CornJob）
    - 即 kubernetes 中的“CronJob”，定时任务是按照指定时间周期运行的短任务。使用场景为在某个固定时间点，为所有运行中的节点做时间同步。
- 编排（Orchestration）
    - 编排模板包含了一组容器服务的定义和其相互关联，可以用于多容器应用和虚机应用的部署和管理。
- 服务（Service）
    - 服务定义了实例及访问实例的途径，如单个稳定的 IP 地址和相应的 DNS 名称。
- 应用服务网格（Istio）
    - Istio 是一个提供连接、保护、控制以及观测功能的开放平台。

# 使用Vagrant + VirtualBox 部署本地集群
注意 请阅读Vagrant 当前兼容 VirtualBox版本 不然 无法调度
这里安装Vagrant(2.2.9) VirtualBox6.0
``` 
# 添加virtualbox 源
wget https://download.virtualbox.org/virtualbox/6.0.24/virtualbox-6.0_6.0.24-139119~Ubuntu~bionic_amd64.deb
wget https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.deb

sudo dpkg -i virtualbox-6.0_6.0.24-139119~Ubuntu~bionic_amd64.deb
sudo dpkg -i vagrant_2.2.9_x86_64.deb
```
### Vagrant 快速入门
- os市场 https://app.vagrantup.com/boxes/search
- 镜像操作
    - `vagrant box add bento/ubuntu-20.04`   (类比 docker pull xxxx)
    - `vagrant box add os_name /xxx/xxx/dic_path` 导入本地box
    - `vagrant box list` (docker images)
    - `vagrant box remove box-name`  (docker images rm xxx)
    - `vagrant package --base box_name --output ./ubuntu_amd64.box` (docker build)
- 初始化 & 启动
    - `mkdir new_project` 创建工作目录 (未来所有os开销文件都会放到这里)
    - `vagrant init ubuntu20` 初始化os
    - `vagrant up` 启动虚拟机
    - `vagrant status` 查看虚拟机运行状态
    - `vagrant ssh` ssh 进入虚拟机
    - `vagrant halt` 关闭虚拟机
    - `vagrant reload` 重启 加载配置文件
    - `vagrant suspend` 挂起
    - `vagrant destroy` 销毁虚拟机
- 快照
    - `vagrant snapshot save [vmname] [快照名称]` 创建快照
    - `vagrant snapshot list` 快照列表
    - `vagrant snapshot restore [vmname] [快照名称]`  还原到指定快照
    - `vagrant snapshot delete [快照名称]` 删除快照
- VagrantFile 常用配置
``` 
# 虚拟机中 linux 的 hostname
config.vm.hostname = "ubuntu1604-lamp"

# 网络设置1 - 公有（public_network）网络（允许局域网中其他机器访问）
config.vm.network "public_network", ip: "192.168.1.223"

# 网络设置2 - 私有（private_network）网络（只允许主机访问虚拟机）
# 此时我们可以使用指定的 IP 加 端口号进行访问，比如使用 192.168.127.11:81 即可访问虚拟机里的 81 端口
config.vm.network "private_network", ip: "192.168.127.11"

# 网络设置3 - 将端口映射到宿主机
# 在宿主机使用 127.0.0.1:8080 即可访问虚拟机里的 80 端口
config.vm.network "forwarded_port", guest: 80, host: 8080

# 共享目录
# 注意：这里的 owner 和 group，与你搭建的 LAMP 环境运行用户一致（在 phpinfo() 页面中搜索 “user”）
config.vm.synced_folder "/Users/用户名/www", "/var/www/html", create:true, owner:"www-data", group:"www-data"

# VirtualBox 虚拟机配置（内存、CPU、显示名称等）
config.vm.provider "virtualbox" do |vb|
#   # Display the VirtualBox GUI when booting the machine
#   vb.gui = true
#
#   # Customize the amount of memory on the VM:
    vb.memory = "512"
    vb.name = "ubuntu1604-lamp"
end
```
- 配置cpu 内存
``` 
config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2
end
```
##### 我们这里使用到的
``` 
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu20"
  config.vm.hostname = "rancher-k8s"
  config.vm.network "public_network", ip: "192.168.1.223"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "8224"
    vb.cpus = 4
  end
end
```
##### 如果安装过旧的box 可能会出现一下问题
- https://github.com/hashicorp/vagrant/issues/8687
- 解决方案
  ``` 
  sudo apt-get autoremove virtualbox-dkms
  sudo apt-get install build-essential linux-headers-`uname -r` dkms virtualbox-dkms
  ```
### 前置操作
- 换阿里源
- install docker
- install rancher `sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher`
安装完 他会自己搭建一套K3S集群 + etcd
### K8s CNI 网络插件对比
- 最流行的插件
    - Flannel
    - Calico
    - Canal
    - Weave
参考文档: https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave/

### 生产级安装
- Master 需要至少3个节点才能保障服务的高可用

| 部署规模	|集群	|节点	|vCPUs	|内存|
| ---- | ---- | ---- | ---- | --- |
|小	|最多 150 个	|最多 1500 个	|2	|8 GB|
|中	|最多 300 个	|最多 3000 个	|4	|16 GB|
|大	|最多 500 个	|最多 5000 个	|8	|32 GB|
|特大	|最多 1000 个|	最多 10,000 个	|16	|64 GB|
|超大	|最多 2000 个|	最多 20,000 个	|32	|128 GB|

### Rancher 安装
- Docker 安装Rancher 这个是开箱即的 但是未来Rancher很难迁移到K8s上(适合开发or测试环境)
- RKE 通过RKE安装K8s  在K8s基础上安装Rancher
- K3S 通过K3S安装K8s  在K3S基础上安装Rancher  (The certified Kubernetes distribution built for IoT & Edge computing 轻量级)

#### 演示通过K3s 搭建K8s
- 1. 创建一台虚拟机 
使用到的配置
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu20"
  config.vm.hostname = "k3s-k8s"
  config.vm.network "public_network", ip: "192.168.88.181"
  #config.vm.network "public_network"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "8224"
    vb.cpus = 4
  end
end
```
- 2. install k3s
    - 注意k3s数据存 这里演示在mysql上
``` 
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -s - server \
--datastore-endpoint="mysql://root:root@tcp(192.168.88.19:3306)/k3s"
```
检测是否安装成功
``` 
sudo k3s kubectl get nodes
```
检测容器运行状况
``` 
sudo k3s kubectl get pods --all-namespaces
```
生成文件访问凭证位置 `/etc/rancher/k3s/k3s.yaml`  这个就是kubeconfig


#### 演示通过RKE 搭建K8s
- install kubectl
``` 
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client
```
- install docker  (`curl -sSL https://raw.githubusercontent.com/dollarkillerx/MyNodeBook/master/docker/ubuntu20_install_docker.sh | sudo bash`)
- 调整防火墙
``` 
     firewall-cmd --permanent --add-port=22/tcp
     firewall-cmd --permanent --add-port=80/tcp
     firewall-cmd --permanent --add-port=443/tcp

     firewall-cmd --permanent --add-port=30000-32767/tcp
     firewall-cmd --permanent --add-port=30000-32767/udp

     firewall-cmd --reload
```
- 同步时间
``` 
ntpdate time1.aliyun.com
```
- 创建集群配置文件 cluster.yml
``` 
rke config --name cluster.yml
```
我们在这里使用的  rke config --name  cluster.yml 会向导生成配置文件   (这里填写具体的服务连接方式)
``` 
nodes:  # 这里配置服务器节点
  - address: 165.227.114.63
    internal_address: 172.16.22.12
    user: ubuntu
    role: [controlplane, worker, etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

# 当使用外部 TLS 终止，并且使用 ingress-nginx v0.22或以上版本时，必须。
ingress:
  provider: nginx
  options:
    use-forwarded-headers: "true"
```
启动 `rke up --config ./rancher-cluster.yml`
会在当前目录生成kubeconfig文件
查看当前集群状态
`kubectl --kubeconfig kube_config_rancher-cluster.yml get nodes`
查看当前Pod运行状态
`kubectl --kubeconfig kube_config_cluster.yml get pods --all-namespaces`
当前服务文件
- rancher-cluster.yml: RKE 集群配置文件。
- kube_config_rancher-cluster.yml: 集群的Kubeconfig 文件，此文件包含用于访问集群的凭据。
- rancher-cluster.rkestate: Kubernetes 集群状态文件，此文件包含用于完全访问集群的凭据。

#### K8S部署方案小结
- K3s 部署K8s最为简单
- RKE 部署K8s 还行 完整的K8S

#### Rancher 部署 以K3s部署为例
- 推荐使用helm 安装
##### Helm 快速入门 (k8s包管理工具)
- install
``` 
https://github.com/helm/helm/releases
wget xxx 
chmod +x xxx
sudo mv xxx /usr/local/bin
```
- 使用
    - search
        - `helm search hub wordpress`
    - 添加存储库
        - `helm repo add brigade https://brigadecore.github.io/charts # helm repo add image_name image_addr`
    - install
        - `helm install happy-panda stable/mariadb`
        - `helm status happy-panda`
- 使用helm install rancher
    - 添加rancher 源
        - `helm repo add rancher-stable http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable`
    - 为rancher 创建 Namespace
        - `kubectl create namespace cattle-system`
    - 配置ssl 保障访问安全性
        - Rancher 自签证书   `ingress.tls.source=rancher`  default  比较安全你使用公共的证书会保留你的域名地址
        - Let's Encrypt     `ingress.tls.source=letsEncrypt`
        - 自己证书           `ingress.tls.source=secret`
    - install cert-manager 创建证书管理中心  (Rancher Let's Encrypt 需要) 
``` 
        kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.yaml
        
        export KUBECONFIG=/etc/rancher/k3s/k3s.yaml # helm 会用到

        # 为cert-manager 创建命名空间
        kubectl create namespace cert-manager
      
        # 添加 Ketstack Helm 仓库
        helm repo add jetstack https://charts.jetstack.io
      
        # 本地更新 Helm Chart 窗口缓存
        helm repo update
      
        # 安装 cert-manager Helm chart
        helm install \
         cert-manager jetstack/cert-manager \
         --namespace cert-manager \
         --version v0.16.0

        # 查看状态
        kubectl get pods --namespace cert-manager
        # 可能拉取不下来images 替换Helm镜像  Azure镜像
        helm repo add stable https://mirror.azure.cn/kubernetes/charts/
        helm repo add incubator https://mirror.azure.cn/kubernetes/charts-incubator/
        # 查看helm源
        helm repo list
        # 删除默认源
        helm repo remove stable

        # 删除POD
        kubectl delete pod $POD_NAME --force --grace-period=0
        # 强制删除NAMESPACE
        kubectl delete namespace $NAMESPACENAME_NAME --force --grace-period=0

        # 查看pod详情
        kubectl describe pod --namespace $NAMESPACENAME_NAME $POD_NAME
        # 查看日志
        kubectl logs $POD_NAME
        # 在Pod中执行命令
        kubectl exec $POD_NAME env
        # 启动容器中的bash
        kubectl exec -ti $POD_NAME bash
```
