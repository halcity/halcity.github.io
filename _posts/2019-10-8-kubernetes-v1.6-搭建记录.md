**1. 初始化环境配置**
- 关闭防火墙
- 关闭SWAP
- 配置bridge转发
```bash
[root@master ~]# vi /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
[root@master ~]# sysctl -p
```

**2. 安装docker**

使用清华或者ustc的镜像源进行安装

**3. 安装kubelet kubeadm kubectl**
```bash
[root@master ~]# vi kubernetes.repo
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
[root@master ~]# yum repolist
[root@master ~]# yum install -y kubelet kubeadm kubectl
```
**4. 查看kubeadm初始化所需image**

``` [root@k8smaster ~]# kubeadm config images list ```

**5. 提前下载image并更改标签（方法一）**
``` bash 
docker pull gcr.azk8s.cn/kube-apiserver:$K8S_VERSION
docker tag gcr.azk8s.cn/kube-apiserver:$K8S_VERSION k8s.gcr.io/kube-apiserver:$K8S_VERSION
docker rmi gcr.azk8s.cn/kube-apiserver:$K8S_VERSION
```
或者用py写的一个wrapper：https://github.com/silenceshell/docker-wrapper

**6. 初始化kubeadm**

``` [root@k8smaster ~]# sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --kubernetes-version=1.16.0 ```

init 常用主要参数：
- --kubernetes-version: 指定Kubenetes版本，如果不指定该参数，会从google网站下载最新的版本信息。
- --pod-network-cidr: 指定pod网络的IP地址范围，它的值取决于你在下一步选择的哪个网络网络插件，比如我在本文中使用的是Calico网络，需要指定为192.168.0.0/16。
- --apiserver-advertise-address: 指定master服务发布的Ip地址，如果不指定，则会自动检测网络接口，通常是内网IP。
- --feature-gates=CoreDNS: 是否使用CoreDNS，值为true/false，CoreDNS插件在1.10中提升到了Beta阶段，最终会成为Kubernetes的缺省选项。

**7.  配置Flannel网络组件**
``` bash
[root@k8smaster ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

**8. 修改Role角色，容忍Mster污点**
``` bash
[root@k8smaster ~]# kubectl label node k8snode node-role.kubernetes.io/worker=worker
[root@k8smaster ~]# kubectl taint node k8smaster node-role.kubernetes.io/master-
```
