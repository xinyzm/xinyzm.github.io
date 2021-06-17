---
title: 快速搭建 K8S 集群环境
keywords: docker kubernetes
catalog: true
comments: true
date: 2020-12-24 16:50:00
subtitle: 
header-img: snail-bg.jpg
tags:
- Linux
- Docker
- Kubernetes
categories:
- 运维
---
说明：首先准备三台主机，一台作为 Master 节点，另外两台作为 Node 节点

### 1、设置主机名以及 hosts 文件

```
vi /etc/hostname
k8s-master/k8s-node01/k8s-node02

vi /etc/hosts
# 在文件最后添加如下配置（ip根据实际情况填写）
192.168.11.53 k8s-master
192.168.11.54 k8s-node01
192.168.11.55 k8s-node02
```

### 2、搭建 Docker 环境

```
1）安装 docker
	curl -sSL https://get.daocloud.io/docker | sh

2）启动服务
	systemctl start docker
	
3）查看服务状态
	systemctl status docker
	
4）设置开机自启动
	systemctl enable docker
	
5）docker镜像加速
	curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

### 3、配置阿里云的软件源（[https://opsx.alibaba.com/mirror](https://opsx.alibaba.com/mirror)）

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 4、将 SELinux 设置为 permissive 模式（相当于将其禁用）

```
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### 5、关闭防火墙（也可以关闭指定端口）

```
systemctl stop firewalld.service
systemctl disable firewalld.service
```

### 6、安装相关软件

```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### 7、关闭 Swap

```
# 修改kubelet禁止提示swap警告，最好关闭swap
vi /etc/sysconfig/kubelet

# 如果配置了swap不然提示出错信息
KUBELET_EXTRA_ARGS="--fail-swap-on=false"

# 关闭swap
swapoff -a

# 修改系统文件使得机器bridge模式开启
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### 8、设置开机自启动

```
systemctl enable --now kubelet
```

### 9、初始化集群（Master 节点执行）

```
kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=Swap --apiserver-advertise-address=192.168.11.53 --image-repository registry.aliyuncs.com/google_containers

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 10、将 Node 节点加入集群（注意每个人的token是不一样的，请以自己的为准）

```
kubeadm join 192.168.11.53:6443 --token qhnrm8.oxvjftyhpf9y7w43 \
    --discovery-token-ca-cert-hash sha256:ddd66aab82248e097151c77cde35356139d683b77c38e5e6dadf14804f2fe648 
```

### 11.配置 flannel 网络（Master 节点执行）

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

在执行上述命令时，出现如下错误**The connection to the server raw.githubusercontent.com was refused - did you specify the right host or port?**，错误原因是这个网址必须能够访问国外网站才能正常访问。

## 参考资料

[手把手教你搭建k8s测试环境](https://cloud.tencent.com/developer/article/1458931)

[K8S官网 - 安装 kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

