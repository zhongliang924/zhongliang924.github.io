---
layout: wiki
title: k8s安装和配置
cate1: k8s
---

Kubernetes 是由谷歌开发的一个开源系统，用于在集群内运行和管理以容器微服务为基础的应用。使用 Kubernetes 需要确保可以从 Kubernetes 集群外部访问在 Kubernetes 内创建的服务。以下是在 `Ubuntu` 主机上安装 K8s 的步骤：

## 主控节点和工作节点都需要执行的操作

**关闭防火墙**

```shell
ufw disable
```

**关闭selinux**

```shell
sudo apt install selinux-utils
setenforce 0
```

**禁止swap分区**

```shell
swapoff -a
sudo vim /etc/fstab		注释掉swap一行
```

**桥接的IPV4流量传递到 iptables 的链**

```shell
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

**配置k8s资源**

```shell
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt-get update
```

**安装nfs**

```shell
apt-get install nfs-common
```

**安装kubeadm(初始化cluster)，kubelet(启动pod)和kubectl(k8s命令工具)**

```shell
apt install -y kubelet=1.21.3-00 kubeadm=1.21.3-00 kubectl=1.21.3-00
```

**设置开机启动并启动kubelet**

```shell
systemctl enable kubelet && systemctl start kubelet
```

## 主控节点需要执行的操作

**Master节点执行初始化配置**

使用 ``kubeadm init`` 命令启动引导一个 Kubernetes 主节点：

```shell
kubeadm init \
  --apiserver-advertise-address=10.24.83.22 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.3 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all
```

参数说明：

```shell
--apiserver-advertise-address=10.24.83.40 \       #修改为自己master ip
--image-repository registry.aliyuncs.com/google_containers \   #设置阿里镜像仓库
--kubernetes-version v1.21.3 \         	#指定k8s版本
--service-cidr=10.96.0.0/12 \   			#指定service  ip网段
--pod-network-cidr=10.244.0.0/16 \		#指定pod ip网段
```

**master节点拷贝认证文件**

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**安装网络插件**

安装 flannel 网络插件：[k8s安装网络插件-flannel_k8s安装flannel_匿称s的博客-CSDN博客](https://blog.csdn.net/bh451326803/article/details/125263918)

执行：

```shell
kubectl apply -f kube-flannel.yml
```

## 工作节点需要执行的操作

**Node 节点加入集群**

在 master 节点执行完 ``kubeadm init`` 之后，出现信息

![image-20230404200323062](/images/wiki/image-20230404200323062.png)

在工作节点执行 ``kubeadm join``，启动引导一个 Kubernetes 工作节点并且将其加入到集群

```shell
kubeadm join 10.24.83.22:6443 ...
```

这条命令是有有效期的，需要的时候，可以执行以下命令获取

```shell
kubeadm token create --print-join-command
```

可以查看集群的基本状况：

```shell
kubectl get nodes
```

![image-20230404200903123](/images/wiki/image-20230404200903123.png)

其中master节点为主机节点，IP为10.24.83.40，node02节点为虚拟机节点，IP为192.168.24.129，可以发现所有的node均已Ready。
