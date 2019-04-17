# 使用Rook安装Ceph存储

本文介绍了如何在minikube搭建的k8s上如何部署ceph存储。首先介绍了minikube的安装方法，然后使用minikube来创建k8s伪分布式集群。然后在上面使用Rook来部署ceph存储集群。

![Rook架构图](https://ws1.sinaimg.cn/large/006gLaqLly1g25f7nmftyj31kw0w0dm8.jpg)

![Rook架构图](https://ws1.sinaimg.cn/large/006gLaqLly1g25fa3j0wsj31sg0wk0yb.jpg)

什么是Rook？
> Rook是一款云原生环境下的开源分布式存储编排系统，目前已进入CNCF孵化。Rook将分布式存储软件转变为自我管理，自我缩放和自我修复的存储服务。它通过自动化部署，引导、配置、供应、扩展、升级、迁移、灾难恢复、监控和资源管理来实现。 Rook使用基础的云原生容器管理、调度和编排平台提供的功能来履行其职责。
>
> Rook利用扩展点深入融入云原生环境，为调度、生命周期管理、资源管理、安全性、监控和用户体验提供无缝体验。
>
> Ceph是一个分布式存储系统，提供文件、数据块和对象存储，可以部署在大型生产集群中。

![ceph架构图](https://ws1.sinaimg.cn/large/006gLaqLly1g25fd6p5sbj30d90hfjuo.jpg)

![ceph架构图](https://ws1.sinaimg.cn/large/006gLaqLly1g25fdnnwjcj30j90dm0sw.jpg)

## 安装MiniKube

### 安装VirtualBox

Step 1: 更新 Ubuntu
安装 VirtualBox 之前， 首先运行以下命令更新 Ubuntu server。

```shell
sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove
```

Step 2: 安装需要的 Linux Headers
更新系统之后， 运行以下命令安装需要的 Ubuntu linux headers。

```shell
sudo apt-get -y install gcc make linux-headers-$(uname -r) dkms
```

Step 3: 添加 VirtualBox Repository and key
安装完上面需要的安装包之后， 运行以下命令安装 VirtualBox repository key。

```shell
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

接下来运行下面的命令添加 VirtualBox repository 到您的系统中。

```shell
sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list'
```

Step 4: 安装 VirtualBox
添加 repository and key 之后， 运行以下命令安装 VirtualBox 5.1。

```shell
sudo apt-get update
sudo apt-get install virtualbox-5.2
```

验证 VirtualBox 是否成功安装，运行以下命令。

```shell
VBoxManage -v
```

Step 5: 安装 VirtualBox Extension Pack
每次安装 VirtualBox 请确保 Extension Pack 也已成功安装， The pack 开启 VRDP (Virtual Remote Desktop Protocol) 和其他提升。

运行以下命令进行安装

```shell
curl -O http://download.virtualbox.org/virtualbox/5.2.0/Oracle_VM_VirtualBox_Extension_Pack-5.2.0-118431.vbox-extpack
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.2.0-118431.vbox-extpack
```

运行以下命令查看 extension pack 是否已经安装成功。

```shell
VBoxManage list extpacks
```

最终结果应该像下面这样。

```shell
Successfully installed "Oracle VM VirtualBox Extension Pack".
richard@ubuntu1704:~$ VBoxManage list extpacks
Extension Packs: 1
Pack no. 0:   Oracle VM VirtualBox Extension Pack
Version:      5.2.0
Revision:     118431
Edition:
Description:  USB 2.0 and USB 3.0 Host Controller, Host Webcam, VirtualBox RDP, PXE ROM, Disk Encryption, NVMe.
VRDE Module:  VBoxVRDP
Usable:       true
Why unusable:
```

### 安装Minikube

```shell
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.0.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

```shell
minikube start --registry-mirror=https://registry.docker-cn.com

minikube dashboard
```

## 安装Rook

```shell
git clone https://github.com/rook/rook/
cd cluster/examples/kubernetes/ceph

kubectl create -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml
```

> **注释**

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    # For the latest ceph images, see https://hub.docker.com/r/ceph/ceph/tags
    image: ceph/ceph:v13.2.5-20190319
  dataDirHostPath: /var/lib/rook
  mon:
    count: 3
    allowMultiplePerNode: false
  dashboard:
    enabled: true
  storage:
    useAllNodes: true
    useAllDevices: true
```

使用minikube安装将cluster.yaml修改为：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    # For the latest ceph images, see https://hub.docker.com/r/ceph/ceph/tags
    image: ceph/ceph:v13.2.5-20190319
  dataDirHostPath: /data/rook
  mon:
    count: 3
    allowMultiplePerNode: false
  dashboard:
    enabled: true
  storage:
    useAllNodes: true
    useAllDevices: true
```

使用下面的命令部署工具pod：

```shell
kubectl apply -f rook-tools.yaml
```

这是一个独立的pod，没有使用其他高级的controller来管理，我们将它部署在rook-system的namespace下。

```shell
kubectl -n rook exec -it rook-tools bash
```

使用下面的命令查看rook集群状态。

```shell
$ rookctl status
OVERALL STATUS: OK

USAGE:
TOTAL       USED       DATA      AVAILABLE
37.95 GiB   1.50 GiB   0 B       36.45 GiB

MONITORS:
NAME             ADDRESS                IN QUORUM   STATUS
rook-ceph-mon0   10.254.162.99:6790/0   true        UNKNOWN

MGRs:
NAME             STATUS
rook-ceph-mgr0   Active

OSDs:
TOTAL     UP        IN        FULL      NEAR FULL
1         1         1         false     false

PLACEMENT GROUPS (0 total):
STATE     COUNT
none

$ ceph df
GLOBAL:
    SIZE       AVAIL      RAW USED     %RAW USED
    38861M     37323M        1537M          3.96
POOLS:
    NAME     ID     USED     %USED     MAX AVAIL     OBJECTS
```

```shell
kubectl create -f storageclass.yaml

kubectl create -f mysql.yaml
kubectl create -f wordpress.yaml
```

```shell
$ kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
mysql-pv-claim   Bound     pvc-95402dbc-efc0-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
wp-pv-claim      Bound     pvc-39e43169-efc1-11e6-bc9a-0cc47a3459ee   20Gi       RWO           1m
```

## 添加监控



## 参考文献

1. <https://my.oschina.net/u/2306127/blog/2051740>

2. <https://rook.github.io/docs/rook/master/ceph-quickstart.html>

3. <https://yq.aliyun.com/articles/221687>

4. <https://github.com/AliyunContainerService/minikube>

5. <https://websiteforstudents.com/installing-virtualbox-5-2-ubuntu-17-04-17-10/>

6. <https://jimmysong.io/kubernetes-handbook/practice/rook.html>

7. <http://docs.ceph.com/docs/mimic/architecture/>

8. <https://subscription.packtpub.com/book/virtualization_and_cloud/9781783985623/3/ch03lvl1sec27/ceph-storage-architecture>