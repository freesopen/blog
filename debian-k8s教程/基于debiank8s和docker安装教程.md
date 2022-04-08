





# 基于debian k8s 和docker 安装教程

## 0 准备工具 

比如ssh 开启 root 远程访问，linux 系统ip 固定，镜像源用http://ftp.cn.debian.org (官方中国镜像)

## **1 安装docker-ce**

### step 1: 安装必要的一些系统工具



```
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common curl wget gnupg  gnupg2   gnupg1 
```

​	注意sudo 报错： 请用root 下apt install sudo 

### step 2: 安装GPG证书
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | apt-key add -
```


### Step 3: 写入软件源信息
```
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable"
例如：
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

https://mirrors.aliyun.com/docker-ce/linux/debian  bullseye stable 
```


### Step 4: 更新并安装Docker-CE
```
sudo apt-get -y update
sudo apt-get -y install docker-ce  docker-compose 
 
```



### 安装指定版本的Docker-CE:

#### Step 1: 查找Docker-CE的版本:

```
apt-cache madison docker-ce

docker-ce | 17.03.1~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages

docker-ce | 17.03.0~ce-0~ubuntu-xenial | https://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
```



#### Step 2: 安装指定版本的Docker-CE: 

(VERSION例如上面的17.03.1~ce-0~ubuntu-xenial)

```
sudo apt-get -y install docker-ce=[VERSION]

```

```


```

##### #推荐

```
cat <<EOF  | sudo tee /etc/docker/daemon.json
{"exec-opts": ["native.cgroupdriver=systemd"],
    "registry-mirrors": ["http://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"],
     "live-restore": true
}
EOF
systemctl restart docker.service
```

##### #比较详细配置参考

```

{
  "graph": "/data/docker", #这个地方是放docker数据，镜像啥的
  "storage-driver": "overlay2",
  "insecure-registries": ["registry.access.redhat.com","quay.io","harbor.kubenetes.online"],#定义了个仓库
  "registry-mirrors": ["https://q2ddddke.mirror.aliyuncs.com"],#定义了国内镜像
  "exec-opts": ["native.cgroupdriver=systemd"],#这个跟cgroupdriver有关,不然后边2个node节点kubelet会报错
  "live-restore": true
}
```



#报错

```
error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: “cgroupfs” is different from docker cgroup driver: “systemd”
解决方式
#修改daemon.json
vim /etc/docker/daemon.json
#添加如下属性
{"exec-opts": ["native.cgroupdriver=systemd"],
        "registry-mirrors":["https://reg-mirror.qiniu.com/","https://docker.mirrors.ustc.edu.cn"]
}

然后重启docker 和kubelet 
systemctl daemon-reload
systemctl restart docker
```



## 2安装k8s

### 1.k8s 官网

地址： https://kubernetes.io/zh

#### 1.1 软件地址

https://developer.aliyun.com/mirror/ 

### 2 安装方式在线

#### 2.1安装  



```
apt-get install ipvsadm ipset sysstat conntrack libseccomp2 apt-transport-https ca-certificates curl software-properties-common
apt-get update && apt-get install -y apt-transport-https

curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -


cat <<EOF  | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF 

sudo apt-get update

设置桥接流量来自官网（https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/）

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```



##### 2.1.1安装apt 方式

最新版 ，需要的可以看下面指定版本1.22.0版本

```
sudo apt install kubelet  kubeadm kubectl
```

需要coredns 1.8.4

执行 

```
kubeadm 22
docker pull coredns/coredns:1.8.4
docker tag coredns/coredns:1.8.4 registry.aliyuncs.com/google_containers/coredns:v1.8.4

kubeadm 21
docker pull coredns/coredns:1.8.0
docker tag coredns/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns/coredns:v1.8.0

kubeadm 23
docker pull coredns/coredns:1.8.6
docker tag coredns/coredns:1.8.6 registry.aliyuncs.com/google_containers/coredns:v1.8.6
```

##### 2.1.2 iptables 很重要不然k8s node 可以会有

 ，根据man文档介绍，旧的iptables使用getsockopt/setsockopt的内核接口，而新的iptables使用nf_tables API。使用Debian的人可能都知道，Debian  10的一大变动就是nftables替换iptables，但是宣传里也说新的iptables能够转化为底层的nftables规则，我也是不太谨慎地相信了，从这个案例看出，还是有地方不能兼容

```text
update-alternatives --config iptables
```

上述命令敲完后，可以选iptables-legacy，这样就一切都work。网上查到，不止weave会有这个问题，calico和kube-proxy都有些问题，新版calico似乎已经支持nft，但要通过配置指定。目前的状态，用上面命令修改iptables的软链指向还是相对靠谱方法。

#### 2.2卸载

```
apt remove --purge kubelet  kubeadm kubectl
```



#### 2.3 指定版本 

```
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
sudo apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00

```

稳住版本号

```
sudo apt-mark hold kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00

sudo apt-mark hold kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00

```

### 3 k8s 初始化

#### 3.1 master

**初始化master**

```

kubeadm init --apiserver-advertise-address=192.168.3.100 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.21.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16

pod-network-cidr（要和网络插件看齐）
```

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

​	注意：cni 插件采用calico

​	官网地址：https://docs.projectcalico.org/getting-started/kubernetes/quickstart

#### **3.2 cni** 网络插件 calico 

​	 下载cni 插件

​		**--pod-network-cidr=10.244.0.0/16 后面ip 要和custom-resources.yaml 中cni 对应**

```
wget -O tigera-operator.yaml https://docs.projectcalico.org/manifests/tigera-operator.yaml
wget -O custom-resources.yaml https://docs.projectcalico.org/manifests/custom-resources.yaml
 
```

#注意修改 custom-resources.yaml

**pod-network-cidr（要和网络插件看齐） 和kubeadm init 的pod-network-cidr 保持一致**

查看是否安装成功cni

```
watch kubectl get pods -n calico-system
```

```
kubectl get nodes -o wide
```

```
#token 24小时有效
kubeadm token create --print-join-command
```

#debian 系统注意点 

#### 3.3 node 点加入k8s 集群

**后面token 有3.1 生成**

```
kubeadm join 192.168.3.100:6443 --token 1hjj4g.5etokasyguls6117 --discovery-token-ca-cert-hash sha256:b4d25a54e7319964ed38a2b1b50835489d7f5c3b74f1f3c474fc1d308d8eb4ab
```

#报错的

        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.8. Latest validated version: 19.03
        error execution phase preflight: [preflight] Some fatal errors occurred:
            [ERROR Swap]: running with swap on is not supported. Please disable swap
    [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
    To see the stack trace of this error execute with --v=5 or higher
<u>需要关闭虚拟内存</u>

<u>swapoff -a 和注释/etc/fstab 对应的</u>

#### 3.4安装web 可视化界面

官网地址这个比较省资源

```
https://www.kuboard.cn/
```

```
sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 8080:80/tcp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.3.100:8080" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3
  #升级
  
  https://kuboard.cn/install/v3-upgrade.html#%E5%A6%82%E6%9E%9C%E4%BB%A5-docker-run-%E8%BF%90%E8%A1%8C-kuboard
  
docker stop $(docker ps -a | grep "eipwork/kuboard" | awk '{print $1 }')
docker rm $(docker ps -a | grep "eipwork/kuboard" | awk '{print $1 }')

 

 
  sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 18080:80/tcp \
  -p 10081:10081/udp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.3.100:18080" \
  -e KUBOARD_AGENT_SERVER_UDP_PORT="10081" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3.1.6.1
```

默认账号 admin/Kuboard123 

接下来按提示操作

其他还有kubesphere，rancher,等等

#### 3.5 rancher 操作

```
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  --privileged \
  -v /home/rancher:/var/lib/rancher \
 rancher/rancher:latest

 docker run -d --restart=unless-stopped   -p 80:80 -p 443:443   rancher/rancher:latest
 docker run -d -v /tmp/rancher:/tmp/rancher --restart=unless-stopped  --privileged -p 80:80 -p 443:443 rancher/rancher:stable
```



#### 4二进制安装

## 3微服务spring-cloud-alibaba

### 3.1搭建NFS Server

master 和node 都要安装



```bsh
 apt-get install nfs-common nfs-kernel-server
```

启动

```
systemctl start nfs-kernel-server
```

配置

```
执行命令 `vim /etc/exports`，创建 exports 文件，文件内容如下：
#/home/nfs/ 共享文件目录

/home/nfs/ *(insecure,rw,sync,no_root_squash)
# 创建共享目录，如果要使用自己的目录，请替换本文档中所有的 /root/nfs_root/

mkdir /home/nfs/

systemctl enable nfs-kernel-server

systemctl start nfs-kernel-server
exportfs -r
```



#### 3.1.1[#](https://kuboard.cn/learning/k8s-intermediate/persistent/nfs.html#在客户端测试nfs)在客户端测试nfs

```sh
# showmount -e $(nfs服务器的IP)
showmount -e 192.168.3.100
# 输出结果如下所示
Export list for 192.168.3.100:/home/nfs/ *
```

#### 3.1.2 客户机挂载

执行以下命令挂载 nfs 服务器上的共享目录到本机路径 `/root/nfsmount`

```sh
mkdir /home/nfs/
# mount -t nfs $(nfs服务器的IP):/home/nfs /home/nfs
mount -t nfs 192.168.3.100:/home/nfs /home/nfs
# 写入一个测试文件
echo "hello nfs server" > /home/nfs/test2.txt
```

在 nfs 服务器上执行以下命令，验证文件写入成功

```sh
cat /home/nfs/test2.txt
```

参考文档

https://kuboard.cn/learning/k8s-intermediate/persistent/nfs.html#%E5%9C%A8%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B5%8B%E8%AF%95nfs

### 3.2安装nacos-k8s

##### 

安装helm

```fallback
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

有问题可采用去官网下载

```shell
https://github.com/helm/helm/releases
```

然后解压

```shell
cp linux-amd64/helm /usr/local/bin/
	
```

#debian/ubuntu 安装apt 方式

```
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
 
```

国内镜像

```
删除默认的源
helm repo remove stable
2、增加新的国内镜像源
helm repo add stable https://burdenbear.github.io/kube-charts-mirror/
或者
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

3、查看helm源添加情况
helm repo list
4、搜索测试

helm search mysql
```



```
# 配置微软源
helm repo add stable http://mirror.azure.cn/kubernetes/charts
# 配置阿里源
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
# 配置google源
helm repo add google https://kubernetes-charts.storage.googleapis.com/

# 更新
helm repo update
```

