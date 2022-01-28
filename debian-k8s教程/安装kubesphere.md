

### 安装

### KubeSphere  本次采用ubutnu 16.04 演示

### 依赖项要求

![](images\yilai.png)

```
apt install socat conntrack ebtables ipset
```

要先安装docker k8s基础20 版本  ，然后还有安装openssl 和ntf

```bash
./kk create cluster  --with-kubesphere v3.1.1  
```
