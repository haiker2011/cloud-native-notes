# 使用Rook安装Ceph存储

本文介绍了如何在minikube搭建的k8s上如何部署ceph存储。首先介绍了minikube的安装方法，然后使用minikube来创建k8s伪分布式集群。然后在上面使用Rook来部署ceph存储集群。

## 安装MiniKube

### 安装VirtualBox

Step 1: Update Ubuntu
Before installing VirtualBox, run the commands below to update Ubuntu server.

```shell
sudo apt-get update && sudo apt-get dist-upgrade && sudo apt-get autoremove
```

Step 2: Install Required Linux Headers
Now that your system is updated, run the commands below to install required Ubuntu linux headers.

```shell
sudo apt-get -y install gcc make linux-headers-$(uname -r) dkms
```

Step 3: Add VirtualBox Repository and key
After installing the required package above, run the commands below to install VirtualBox repository key.

```shell
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

Next run the commands below to add VirtualBox repository to your system.

```shell
sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian $(lsb_release -sc) contrib" >> /etc/apt/sources.list'
```

Step 4: Install VirtualBox
After adding the repository and key, run the commands below to install VirtualBox 5.1. At the time of this writing the latest version of the software was 5.1. If there are newer versions available, please replace the one below with the current latest.

```shell
sudo apt-get update
sudo apt-get install virtualbox-5.2
```

To verify if VirtualBox is installed, run the commands below.

```shell
VBoxManage -v
```

Step 5: Install VirtualBox Extension Pack
Everytime you install VirtualBox make sure to install the extension pack as well. The pack enables VRDP (Virtual Remote Desktop Protocol) and many other enhancements.

To install it, run the commands below

```shell
curl -O http://download.virtualbox.org/virtualbox/5.2.0/Oracle_VM_VirtualBox_Extension_Pack-5.2.0-118431.vbox-extpack
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.2.0-118431.vbox-extpack
```

Agree to the terms and install.

Run the commands below to view the extension pack installed.

```shell
VBoxManage list extpacks
```

The results should look like the one below:

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

1. https://my.oschina.net/u/2306127/blog/2051740

2. https://rook.github.io/docs/rook/master/ceph-quickstart.html

3. https://yq.aliyun.com/articles/221687

4. https://github.com/AliyunContainerService/minikube

5. https://websiteforstudents.com/installing-virtualbox-5-2-ubuntu-17-04-17-10/
