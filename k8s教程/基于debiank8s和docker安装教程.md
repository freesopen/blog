



# 基于debian k8s 和docker 安装教程

## 0 准备工具 

比如ssh 开启 root 远程访问，linux 系统ip 固定，镜像源用http://ftp.cn.debian.org (官方中国镜像)

## **1 安装docker-ce**

### step 1: 安装必要的一些系统工具



```
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common curl wget
```

​	注意sudo 报错： 请用root 下apt install sudo 

### step 2: 安装GPG证书
```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | apt-key add -
```


### Step 3: 写入软件源信息
```
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable"

https://mirrors.aliyun.com/docker-ce/linux/debian  bullseye stable 
```


### Step 4: 更新并安装Docker-CE
```
sudo apt-get -y update
sudo apt-get -y install docker-ce
sudo apt install docker-compose 
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
创建/etc/docker/daemon.json
{
  "registry-mirrors": [
 "https://docker.mirrors.ustc.edu.cn"
 ]
}

```

#推荐

```
{
    "registry-mirrors": ["http://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"]
}
systemctl restart docker.service
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
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
#修复不能添加问题 
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
 
```



#### 2.2卸载

```
apt remove --purge kubelet  kubeadm kubectl
```



#### 2.3 指定版本

```
sudo apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
sudo apt-get install -y kubelet=1.19.0-00 kubeadm=1.19.0-00 kubectl=1.19.0-00
```

稳住版本号

```
sudo apt-mark hold kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00
sudo apt-mark hold kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
sudo apt-mark hold kubelet=1.19.0-00 kubeadm=1.19.0-00 kubectl=1.19.0-00
```

### 3 k8s 初始化

#### 3.1 master

**初始化master**

```
kubeadm init --apiserver-advertise-address=192.168.3.100 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.20.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16


kubeadm init --apiserver-advertise-address=192.168.3.100 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.21.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16


```

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

​	注意：cni 插件采用calico

​	官网地址：https://docs.projectcalico.org/getting-started/kubernetes/quickstart

​	 下载cni 插件

​		**--pod-network-cidr=10.244.0.0/16 后面ip 要和custom-resources.yaml 中cni 对应**

```
wget -O tigera-operator.yaml https://docs.projectcalico.org/manifests/tigera-operator.yaml
wget -O custom-resources.yaml https://docs.projectcalico.org/manifests/custom-resources.yaml
 
```

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

kubeadm join 192.168.3.100:6443 --token phrkir.z1tjj10vqgq1kl6c --discovery-token-ca-cert-hash sha256:c00d682742e32989778c77ee10a330a2fcd3e61899bbe259306526385133b6ec

#### 3.2 node 点加入k8s 集群

**后面token 有3.1 生成**

```
kubeadm join 192.168.3.100:6443 --token bjvat6.5jejaxe0w8a9a3bv     --discovery-token-ca-cert-hash sha256:8ec679f5543730ef867ad7d11ccfd985279e216c4a6a04a592cd7662a126b337
```

#

        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.8. Latest validated version: 19.03
        error execution phase preflight: [preflight] Some fatal errors occurred:
            [ERROR Swap]: running with swap on is not supported. Please disable swap
    [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
    To see the stack trace of this error execute with --v=5 or higher
<u>需要关闭虚拟内存</u>

<u>swapoff -a 和注释/etc/fstab 对应的</u>

#### 3.3安装web 可视化界面

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
  sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 8080:80/tcp \
  -p 10081:10081/udp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://192.168.3.49:8080" \
  -e KUBOARD_AGENT_SERVER_UDP_PORT="10081" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  eipwork/kuboard:v3.1.4.3
```

默认账号 admin/Kuboard123 

接下来按提示操作

其他还有kubesphere，rancher,等等

#### 3.4 rancher 操作

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

### 3.1安装nacos-k8s



```bsh
 apt-get install nfs-common nfs-kernel-server
```

启动

```
systemctl start nfs-kernel-server
```

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

配置源

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

