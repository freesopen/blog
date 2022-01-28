# 创建kubectl create

# 1.0 基本api

kubectl api-versions

# 2.0命令创建部署



kubectl create deployment nginx --image=nginx

 kubectl expose deployment nginx --port=80 --type=NodePort

1生成方式

kubectl create deployment web --image=nginx -o yaml --dry-run  >my.yaml

2 根据部署的生成

kubectl get deploy nginx -o=yaml --export >m2.yaml

