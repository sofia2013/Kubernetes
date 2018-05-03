# 单机版Kubernetes-1.5集群环境安装和配置
## 环境参数
### 操作系统 
CentOS 7（3.10.0-693.21.1.el7.x86_64 ）
### 网络参数
IP 172.17.172.197
## 安装Kubernetes
### 1、	更新操作系统内核
yum update
### 2、	关闭CentOS 自带的防火墙服务
```
systemctl disable firewalld
systemctl stop firewalld
```
### 3、	安装软件
```
yum install -y etcd
yum install -y kubernetes
yum install -y *rhsm*
```
### 4、	修改配置
```
vim /etc/sysconfig/docker
```
将OPTIONS内容设置为：
OPTIONS='--selinux-enabled=false --insecure-registry grc.io'
```
vim /etc/kubernetes/apiserver
```
把--admission_control参数中的ServiceAccount删除
### 5、	按顺序启动所有的服务
```
systemctl start etcd
systemctl start docker 
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kubelet
systemctl start kube-proxy
```
