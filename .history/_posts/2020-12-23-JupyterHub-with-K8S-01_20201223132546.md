---
layout: post
title: 项目管理实战——基于真实项目案例的项目管理策略
subtitle: 实际的项目管理技巧与方法，很多东西是PMBOK、高项考试里不会出现的。
image: /img/sml-imgs/svcc.jpeg
bigimg: 
  -  "/img/big-imgs/qinghai1.jpeg" : "青海湖，2014年"
tags: [Jupyter, Jupyter Notebooks, JupyterHub, K8S, kubernetes]
comments: true
---

# JupyterHub-with-K8S-01: k8s Deploy

## 1. Ubuntu环境准备

用virtualbox准备五个虚拟机。  


|name|ip|
|:------:|:---------:|
|k8s-master|10.8.60.240|
|k8s-node-01|10.8.60.244|
|k8s-node-02|10.8.60.252|
|k8s-node-03|10.8.60.253|
|nfs-svr|10.8.60.243|


对于virtualbox网络配置不熟悉的，可以参考：[virtualbox 虚拟机组网](https://www.jianshu.com/p/e6684182471b).


开启`SSH server`、`oh my zsh`：

```shell
sudo apt-get install -y openssh-server zsh vim curl git wget net-tools
sudo ps -e | grep ssh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# inside ~/.zshrc
plugins=(zsh-autosuggestions zsh-syntax-highlighting)
```

改为固定IP，避免DHCP抽风后获得的IP地址变化：

```shell
sudo vi /etc/network/interfaces
```

编辑如下内容（按照具体情况修改）：

```bash
auto lo
iface lo inet loopback

dns-nameservers 172.16.11.11

auto enp0s3
iface enp0s3 inet static
address 10.8.60.199
netmask 255.255.252.0
gateway 10.8.60.1
```

**但是对于18.04 貌似这样改DNS不对，可以SSH，但是不能上网，可以在用界面手动配置，具体怎么解决有空再说。**

### 安装NFS server与客户端，方便后续挂载PV

基本步骤：

```shell
kubectl create -f deploy/rbac.yaml  # 创建ServiceAccount以及规则
kubectl create -f deploy/class.yaml  # 创建StorageClass
kubectl create -f deploy/deployment.yaml  # 部署nfs-client-provisioner
kubectl create -f deploy/test-claim.yaml  # 创建PVC
kubectl create -f deploy/test-pod.yaml  # 测试pod可以访问NFS做为后端存储的PVC
```

> **注意**
> 最好检查一下`/etc/resolv.conf`，如果发现 nameserver 是回环地址，后面 pod 状态有可能会是 `CrashLoopBackOff`，需要改成正常的 `DNS` 地址：

```conf
nameserver 127.0.1.1
```

### Before you begin,确认环境

> 非常重要，以后可能出的稀奇古怪的问题，都可能是环境问题导致的，所以一行一行确认下环境没有坏处。

- One or more machines running one of:
  - Ubuntu 16.04+
  - Debian 9+
  - CentOS 7
  - Red Hat Enterprise Linux (RHEL) 7
  - Fedora 25+
  - HypriotOS v1.0.1+
  Flatcar Container Linux (tested with 2512.3.0)
- 2 GB or more of RAM per machine (any less will leave - little room for your apps)
- 2 CPUs or more
- Full network connectivity between all machines in the - cluster (public or private network is fine)
- Unique hostname, MAC address, and product_uuid for every - node. See here for more details.
- Certain ports are open on your machines. See here for more - details.
- `Swap` **disabled**. You **MUST** disable swap in order for the - kubelet to work properly.

### 1.1 问题处理

```shell
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

**解决方案**

1、重设root密码

```shell
sudo passwd root
```

2、提权运行`apt-get`命令

```shell
su
```

## 2. Installing kubeadm

This page shows how to [install the kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) toolbox. For information how to create a cluster with kubeadm once you have performed this installation process, see the [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) page.The Chinese version is[安装 kubeadm](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).

### 2.1 环境检查

#### 确认MAC和product_uuid的唯一性

Verify the MAC address and product_uuid are unique for every node.

```shell
ifconfig -a    # 查看MAC
sudo cat /sys/class/dmi/id/product_uuid # 查看product_uuid
```

#### Check network adapters

#### Letting iptables see bridged traffic

#### 配置防火墙

Check required ports.
在本地内网环境测试为了方便, 直接关闭了防火墙. 若安全要求较高, 可以参考官方文档放行必要端口。

```shell
systemctl stop firewalld   # 关闭服务
systemctl disable firewalld    # 禁用服务
```

#### 禁用SELinux

官方文档：[coredns pods have CrashLoopBackOff or Error state](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#coredns-pods-have-crashloopbackoff-or-error-state).  
修改`/etc/selinux/config`, 设置`SELINUX=disabled`.  
**重启机器**。  

```shell
sudo apt install -y policycoreutils
k8s@k8s-master:~$ sestatus    # 查看SELinux状态
SELinux status:                 disabled
```

#### 禁用交换分区

Swap disabled. You MUST disable swap in order for the kubelet to work properly.

临时禁用：`sudo swapoff -a`

编辑`/etc/fstab`, 将`swap`注释掉。

```shell
root@k8s-master:/home/k8s# vim /etc/fstab

#/swapfile                                 none            swap    sw              0       0
```

**重启机器**,检查`swap`:

```shell
k8s@k8s-master:~$ sudo free -m
              total        used        free      shared  buff/cache   available
Mem:           1987         874         228          31         884         924
Swap:             0           0           0
```

#### 配置 sysctl iptables

`shell`需要切换到`root`，`sudo`貌似权限也不够。

```shell
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 2.2 安装 runtime

#### Docker

Use the following commands to install Docker on your system:

```shell
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2
```

```shell
# Add Docker's official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
```

```shell
# Add the Docker apt repository:
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

> **注意：**此处有坑，官方文档安装的 docker 版本是19.03.11 ，但是貌似用ubuntu 16的时候出现了“failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"”的问题，可以用“pt-cache madison docker-ce”查看具体的版本，修改为：“docker-ce=5:19.03.12~3-0~ubuntu”

```shell
# Install Docker CE
apt-get update && apt-get install -y \
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)
```

```shell
# Set up the Docker daemon
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

不得不服墙，如果使用建议使用国内镜像[docker-registry-cn-mirror-test](https://github.com/docker-practice/docker-registry-cn-mirror-test/actions)，可以在`daemon.json`中增加：

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "storage-driver": "overlay2"
}
```

其中https://xxxxxxxx.mirror.aliyuncs.com为阿里云镜像加速地址, xxxxxxxx需要替换为自己账户中的地址。

```shell
mkdir -p /etc/systemd/system/docker.service.d
```

```shell
# Restart Docker
systemctl daemon-reload
systemctl restart docker
```

If you want the docker service to start on boot, run the following command:

```shell
sudo systemctl enable docker
```

Refer to the [official Docker installation guides](https://docs.docker.com/engine/install/) for more information.

### 2.3 安装Kubernetes： kubeadm、kubelet 和 kubectl

您需要在每台机器上安装以下的软件包：

  - `kubeadm`：用来初始化集群的指令。
  - `kubelet`：在集群中的每个节点上用来启动 pod 和容器等。
  - `kubectl`：用来与集群通信的命令行工具。

#### 准备工作

由于官方源地址国内不可用，只有`shell`走代理(代理IP地址为宿主机IP，且虚拟机配置了NAT网卡)或者更换为阿里源。

```shell
export https_proxy=http://192.168.31.180:7890 http_proxy=http://192.168.31.180:7890 all_proxy=socks5://192.168.31.180:7890

curl cip.cc    #确认代理是否生效
```

添加阿里源：由于国内网络原因, 官方文档中的地址不可用, 可以替换为阿里云镜像地址, 执行以下代码即可:

```shell
# 添加并信任APT证书
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

# 添加源地址
add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"
```

#### 安装Kubernetes

[k8s 安装helm, minio, nfs, MetalLB, ingress-nginx, cert-manager, k8s-dashboard, met](https://juejin.im/post/6844904169304768520)

```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
# 使用阿里源
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF 
# 使用中国科学技术大学源
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

##### 报错处理

即使走代理`sudo apt-get update`中可能也会遇到如下报错：

```shell
Get:6 http://cn.archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Fetched 252 kB in 43s (5,826 B/s)
Reading package lists... Done
W: Failed to fetch https://apt.kubernetes.io/dists/kubernetes-xenial/InRelease  Could not connect to packages.cloud.google.com:443 (216.58.200.46), connection timed out
W: Some index files failed to download. They have been ignored, or old ones used instead.
```

可以尝试：

```shell
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

但是也基本没用，只有换源能解决办法，为什么挂了代理还不行，先不研究了。

## 3. 初始化控制平面节点

**注意**

`--control-plane-endpoint`是创建高可用集群的（3台master节点，3台worker节点），所以：标志应该被设置成负载均衡器的地址或 DNS 和端口。

`--pod-network-cidr` 地址的选择和使用的网络插件有关，不同的网络插件有不同的默认地址，为了避免后面的麻烦，还是老老实实用网络插件官方的地址为最优选择。

官方文档：[Installing a Pod network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)中有这样的注释：

>**Note**: Currently **Calico** is the only CNI plugin that the kubeadm project performs e2e tests against. If you find an issue related to a CNI plugin you should log a ticket in its respective issue tracker instead of the kubeadm or kubernetes issue trackers.

所以，似乎选择[`Calico`](https://www.projectcalico.org/)似乎比较合理了，[`quickstart`](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)是这样写的：

```shell
# Initialize the master using the following command.
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

## 3.1 初始化

一切都确认了之后可以运行：

```shell
sudo kubeadm init --dry-run
```

做一下检查，安装出了问题可以用`kubeadm reset`清理。

```shell
sudo kubeadm init \
    --pod-network-cidr=192.168.0.0/16 \
    --apiserver-advertise-address=10.8.60.240 \
    --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'

# 如果自动生成的yaml文件版本不对可以增加：
#      --kubernetes-version=1.18.0 
```

貌似墙还是很强大的，可以参考下面修改yaml文件，预先把镜像拉下来：

```shell
sudo kubeadm config images pull --config kubeadm-init.yaml
```


也可以生成init的yaml文件，便于学习和理解。

```shell
kubeadm config print init-defaults > kubeadm-init.yaml
```

按照命令行的参数，修改`yaml`文件中对应的内容，然后初始化：

```shell
kubeadm init --config kubeadm-init.yaml
```

如果一切顺利，可以看到如下输出，最后部分`kubeadm join`开头的部分需要`copy`下来，后续worker加入集群是需要这个`token`的。  
如果你不慎遗失了该命令，可以在master节点上使用：

```shell
kubeadm token create --print-join-command
```

命令来重新生成一条。


```shell
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.8.60.240:6443 --token ueko10.v0aljk1x0y2dhc11 \
    --discovery-token-ca-cert-hash sha256:db67a3af7a9a2dd381f47aef846e2bc7fca8e6523415b9221ae0210ea1eeeef1
```

### 3.2 配置 kubectl 工具

要使kubectl适用于您的非root用户，请运行以下命令，这些命令也是kubeadm init输出的一部分：

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

执行完成后并不会刷新出什么信息，可以通过下面两条命令测试 kubectl是否可用：

```shell
# 查看已加入的节点
kubectl get nodes
# 查看集群状态
kubectl get cs
```

大概率会返回如下带错误信息的内容：

```shell
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```

参考[kubeadm安装k8s 组件controller-manager 和scheduler状态 Unhealthy](https://blog.csdn.net/hehj369986957/article/details/107855829)。

`kubectl get pod -A`会发现对应的pod状态正常。  
使用 `sudo netstat -tlunp` 可以确认没有启用`10252`和`10251`端口。  
使用 `ss -ant | grep 10252` 没有返回，说明端口没有监听。  

检查kube-scheduler和kube-controller-manager组件配置是否禁用了非安全端口:

```shell
vim /etc/kubernetes/manifests/kube-scheduler.yaml
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

修改其中的：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    - --port=0
    image: k8s.gcr.io/kube-scheduler:v1.19.0
```

去掉`- --port=0` 。

然后`sudo systemctl restart kubelet`再检查`kubectl get cs`会发现集群状态正常了。

### 3.3 部署 Calico 网络

[Quickstart for Calico on Kubernetes](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)

> **注意**：许多Linux发行版都包含NetworkManager。
> 默认情况下，NetworkManager不允许Calico管理界面。如果您的节点具有NetworkManager，请 在安装Calico之前完成[防止NetworkManager控制Calico接口](https://docs.projectcalico.org/maintenance/troubleshoot/troubleshooting#configure-networkmanager)中的步骤 。

在以下位置创建以下配置文件，`/etc/NetworkManager/conf.d/calico.conf`以防止NetworkManager干扰接口：

```shell
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico

```

1. 安装Tigera Calico运算符和自定义资源定义。
   
    ```shell
   kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
   ```

2. 通过创建必要的自定义资源来安装Calico。有关此清单中可用配置选项的更多信息，请参见[安装参考](https://docs.projectcalico.org/reference/installation/api)。
   

    ```shell
    kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
    ```

   > **注意**：在创建此清单之前，请阅读其内容并确保其设置适合您的环境。例如，您可能需要更改默认IP池CIDR以匹配您的Pod网络CIDR。
3. 使用以下命令确认所有Pod正在运行。
   
    ```shell
   watch kubectl get pods -n calico-system
   ```

    等待直到：each pod has the STATUS of Running，时间有点儿久，确认好后，再进行下一步。
   
    >**注意**：Tigera运算符将资源安装在`calico-system`名称空间中。其他安装方法可能改用`kube-system`名称空间。
4. 控制平面节点隔离,移除母版上的taints，以便您可以在其上调度pod。  
   默认情况下，出于安全原因，您的群集不会在控制平面节点上调度Pod。如果您希望能够在控制平面节点上计划Pod，例如用于开发的单机Kubernetes集群，请运行:

    ```shell
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```

    应该返回类似下面的内容：

    ```shell
    node/k8s-master untainted
    ```

5. 使用以下命令确认集群中现在有一个节点。
   
   ```shell
    kubectl get nodes -o wide
   ```

    应该返回类似下面的内容：

    ```shell
    NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
    k8s-master   Ready    master   43m   v1.19.0   10.8.60.199   <none>        Ubuntu 18.04.5 LTS   5.4.0-42-generic   docker://19.3.11
    ```

恭喜你！现在，您有了带有Calico的单主机Kubernetes集群。

#### 安装calicoctl

有需要的话，可以考虑安装 [calicoctl](https://docs.projectcalico.org/getting-started/clis/calicoctl/install).

## 4. 加入你的节点

### 4.1 逐一加入节点

节点是您的工作负载（容器和Pod等）运行的地方。要将新节点添加到群集，请对每台计算机执行以下操作：

- SSH到机器
- 成为root（例如sudo su -）
- 运行在`master`机器上`kubeadm init`成功后，最后输出的命令。例如：

```shell
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

加入后可以在`master`机器上使用`kubectl get nodes`验证是否加入成功，输出如下。  

```shell
NAME            STATUS   ROLES    AGE     VERSION
k8s-master      Ready    master   111m    v1.18.5
k8s-worker-01   Ready    <none>   10m     v1.18.5
k8s-worker-02   Ready    <none>   9m33s   v1.18.5
k8s-worker-03   Ready    <none>   9m8s    v1.18.5
```

`worker`加入后状态变成`Ready`还是需要一段时间的，所以可以用`wacth`命令来观察状态变化，确认都可以`Ready`。

如果没有令牌，则可以通过在控制平面节点上运行以下命令来获取令牌：

```shell
kubeadm token list
```

默认情况下，令牌会在24小时后过期。如果要在当前令牌过期后将节点加入集群，则可以通过在控制平面节点上运行以下命令来创建新令牌：

```shell
kubeadm token create
```

### 4.2 （可选）从控制平面节点以外的计算机控制集群

为了使kubectl在其他计算机（例如笔记本电脑）上与您的集群通信，您需要将管理员kubeconfig文件从控制平面节点复制到工作站，如下所示：

```shell
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

>注意：
上面的示例假定为root用户启用了SSH访问。如果不是这种情况，您可以复制admin.conf文件以供其他用户访问，而scp改用该其他用户。

### 4.3 （可选）将API服务器代理到本地主机

如果要从群集外部连接到API服务器，则可以使用 `kubectl proxy`：

```shell
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

现在，您可以在本地访问API服务器:

```shell
http://localhost:8001/api/v1

```

## 5. 验证及命令行工具



### 5.1 使用Sonobuoy验证集群是否正常运行

[Sonobuoy](https://github.com/vmware-tanzu/sonobuoy)是一种诊断工具，通过以可访问且无损的方式运行一组插件（包括Kubernetes一致性测试），可以更轻松地了解Kubernetes集群的状态。这是一种可自定义，可扩展且与群集无关的方法，可以生成有关群集的清晰，有用的报告。

[How to Validate Your Kubernetes Cluster With Sonobuoy](https://medium.com/better-programming/how-to-validate-your-kubernetes-cluster-with-sonobuoy-c91b282908fe).

`sonobuoy run --wait`命令执行一次需要一个多小时，慎重选择。
**注意：** `sonobuoy run ----mode quick`使用--mode quick会大大缩短Sonobuoy的运行时间。它仅运行一个测试，可帮助快速验证您的Sonobuoy和Kubernetes配置。


### 5.2 kubectl 备忘单

官方文档，可以看看：[kubectl 备忘单](https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/)。  

## 6. Nginx-Ingress 、MetalLB安装

由于负载均衡在云平台基本都属于基础设施直接提供，使用 `kubernetes` 官方的 `ingress-nginx` 在裸机部署的时候，需要注意：  

[裸机注意事项](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/) 中推荐了纯软件解决方案：[`MetalLB`](https://metallb.universe.tf/) .


[kubernetes LoadBalancer 服务部署](https://www.jianshu.com/p/4595ca7e29c8)

## 7. Dashboard安装

[Web 界面 (Dashboard)](https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/)

[Kubernetes Dashboard 安装与使用](https://developer.aliyun.com/article/745086)

[Kubernetes 私有集群负载均衡器终极解决方案 MetalLB](https://blog.csdn.net/easylife206/article/details/106009973)

[Kubernetes 私有集群负载均衡器终极解决方案 MetalLB](https://cloud.tencent.com/developer/article/1631910)

[Kubernetns LB方案：无需云厂商的动态DNS和负载均衡](https://cloud.51cto.com/art/202008/624993.htm)

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

默认只能`localhost`访问，要在笔记本访问，最简单的办法是端口转发[访问K8s Dashboard的几种方式](https://segmentfault.com/a/1190000023130407)：

```shell
ssh -L localhost:8001:localhost:8001 -NT k8s@10.8.60.240
```

然后在工作电脑访问：`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`

## 8. 管理集群中的 TLS 认证

[利用 CSR API 创建用户](https://k8s.imroc.io/security/user/create-user-using-csr-api/)

[管理集群中的 TLS 认证](https://kubernetes.io/zh/docs/tasks/tls/managing-tls-in-a-cluster/)

## 9. 各种坑

[搭建Kubernetes集群时DNS无法解析问题的处理过程](https://juejin.im/post/6844903638368780301)

[利用NFS动态提供Kubernetes后端存储卷](https://jimmysong.io/kubernetes-handbook/practice/using-nfs-for-persistent-storage.html)