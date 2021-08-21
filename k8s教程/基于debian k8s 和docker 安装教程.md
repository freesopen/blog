

# 基于debian k8s 和docker 安装教程

## 0 准备工具 

比如openssh 开启 root 远程访问，linux 系统ip 固定，镜像源用http://ftp.cn.debian.org (官方中国镜像)

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
cat <<EOF /etc/apt/sources.list.d/kubernetes.list
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
docker pull coredns/coredns:1.8.4
docker tag coredns/coredns:1.8.4 registry.aliyuncs.com/google_containers/coredns:v1.8.4
```



#### 2.2卸载

```
apt remove --purge kubelet  kubeadm kubectl
```



#### 2.3 指定版本

```
sudo apt-get install -y kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00
```

稳住版本号

```
sudo apt-mark hold kubelet=1.20.0-00 kubeadm=1.20.0-00 kubectl=1.20.0-00

```

### 3 k8s 初始化

#### 3.1 master

**初始化master**

```
kubeadm init --apiserver-advertise-address=192.168.3.100 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.20.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16


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

#### 3.2 node 点加入k8s 集群

**后面token 有3.1 生成**

```
kubeadm join 192.168.3.100:6443 --token 4dfwyc.t6lu91gtt1pxd1xe \
        --discovery-token-ca-cert-hash sha256:aa008c02a03003d10486b2dc1413e77a58655665d138dea1aac8e5bf0da52e6d
```

#### 3.3安装web 可视化界面

官网地址

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
```

默认账号 admin/Kuboard123 

接下来按提示操作

