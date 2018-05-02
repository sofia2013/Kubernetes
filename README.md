# Kubernetes

## 第二部分 Node节点配置

### 环境参数

操作系统：CentOS 7

IP ：172.31.240.48

Kubernetes版本：1.9.1

Docker版本：1.13.1 

#### 1、更新内核
yum update -y

#### 2、关闭防火墙

[root@node ~]# systemctl disable firewalld

[root@node ~]# systemctl stop firewalld

#### 3、安装docker

[root@node ~]# yum install docker-1.13.1  

#### 4、下载kubernetes

[root@node ~]# wget https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz

#### 5、配置Kubelet

[root@node ~]# mkdir -p /etc/kubernetes  

[root@node ~]# mkdir -p /var/lib/kubelet

[root@node ~]# vim /usr/lib/systemd/system/kubelet.service


#### kubelet.service
```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/bin/kubelet \
  --address=172.31.240.48 \
  --hostname-override=172.31.240.48 \
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
  --cluster-domain=cluster.local. \
  --allow-privileged=true \
  --cgroup-driver=systemd \
  --fail-swap-on=false \
  --runtime-cgroups=/systemd/system.slice 
  --logtostderr=true \
  --v=2
RestartSec=5

[Install]
WantedBy=multi-user.target

```

[root@node ~]# vim /etc/kubernetes/kubelet.kubeconfig

#### kubelet.kubeconfig

```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: http://172.31.240.42:8080
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: ""
  name: system:node:kube-master
current-context: system:node:kube-master
kind: Config
preferences: {}
users: []

```

#### 5、启动Kubelet服务

[root@node ~]# systemctl daemon-reload 

[root@node ~]# systemctl enable kubelet.service

[root@node ~]# systemctl start kubelet.service

[root@node ~]# systemctl status kubelet.service
