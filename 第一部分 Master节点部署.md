# Master节点部署
## 环境参数
## 部署详情
### 1、更新内核
[root@k8s-master ~]# yum update -y

### 2、关闭防火墙
[root@k8s-master ~]# systemctl disable firewalld
[root@k8s-master ~]# systemctl stop firewalld

### 3、下载二进制安装文件
[root@k8s-master ~]# wget https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
[root@k8s-master ~]# wget https://github.com/coreos/etcd/releases/download/v3.2.9/etcd-v3.2.9-linux-amd64.tar.gz

### 4、Etcd配置
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# cp etcd /usr/bin
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# cp etcdctl /usr/bin
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# vim /usr/lib/systemd/system/etcd.service
######################etcd.service######################
[Unit]
Description=etcd.service

[Service]
Type=notify
TimeoutStartSec=0
Restart=always
WorkingDirectory=/var/lib/etcd
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd 

[Install]
WantedBy=multi-user.target
######################etcd.service######################

[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# mkdir -p /var/lib/etcd
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# mkdir -p /etc/etcd/
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]#  vim /etc/etcd/etcd.conf

######################etcd.conf######################
#[Member]
#ETCD_CORS=""
ETCD_DATA_DIR="/var/lib/etcd/"
#ETCD_WAL_DIR=""
#ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
ETCD_NAME="default"
#ETCD_SNAPSHOT_COUNT="100000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_QUOTA_BACKEND_BYTES="0"
#ETCD_MAX_REQUEST_BYTES="1572864"
#ETCD_GRPC_KEEPALIVE_MIN_TIME="5s"
#ETCD_GRPC_KEEPALIVE_INTERVAL="2h0m0s"
#ETCD_GRPC_KEEPALIVE_TIMEOUT="20s"
#
#[Clustering]
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://172.17.172.197:2379"
#ETCD_DISCOVERY=""
#ETCD_DISCOVERY_FALLBACK="proxy"
#ETCD_DISCOVERY_PROXY=""
#ETCD_DISCOVERY_SRV=""
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_STRICT_RECONFIG_CHECK="true"
#ETCD_ENABLE_V2="true"
#
#[Proxy]
#ETCD_PROXY="off"
#ETCD_PROXY_FAILURE_WAIT="5000"
#ETCD_PROXY_REFRESH_INTERVAL="30000"
#ETCD_PROXY_DIAL_TIMEOUT="1000"
#ETCD_PROXY_WRITE_TIMEOUT="5000"
#ETCD_PROXY_READ_TIMEOUT="0"
#
#[Security]
#ETCD_CERT_FILE=""
#ETCD_KEY_FILE=""
#ETCD_CLIENT_CERT_AUTH="false"
#ETCD_TRUSTED_CA_FILE=""
#ETCD_AUTO_TLS="false"
#ETCD_PEER_CERT_FILE=""
#ETCD_PEER_KEY_FILE=""
#ETCD_PEER_CLIENT_CERT_AUTH="false"
#ETCD_PEER_TRUSTED_CA_FILE=""
#ETCD_PEER_AUTO_TLS="false"
#
#[Logging]
#ETCD_DEBUG="false"
#ETCD_LOG_PACKAGE_LEVELS=""
#ETCD_LOG_OUTPUT="default"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#ETCD_AUTO_COMPACTION_RETENTION="0"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#ETCD_METRICS="basic"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple"
######################etcd.conf######################

[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# systemctl daemon-reload
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# systemctl enable etcd.service
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# systemctl start etcd.service
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# systemctl status etcd.service
● etcd.service
   Active: active (running) since Thu 2018-05-03 09:53:30 CST; 5s ago
...
[root@iZod5e1vrxu935Z etcd-v3.2.9-linux-amd64]# etcdctl cluster-health
member 8e9e05c52164694d is healthy: got healthy result from http://172.17.172.197:2379
cluster is healthy

### 5、Kubernetes 配置

#### 1)kube-apiserver 配置

[root@iZod5e1vrxu935Z bin]# cp kube-apiserver /usr/bin
[root@iZod5e1vrxu935Z bin]# cp kube-controller-manager /usr/bin
[root@iZod5e1vrxu935Z bin]# cp kube-scheduler /usr/bin

[root@iZod5e1vrxu935Z bin]# vim /usr/lib/systemd/system/kube-apiserver.service

######################kube-apiserver.service######################
[Unit]
Description=Kubernetes API Server
After=etcd.service
Wants=etcd.service

[Service]
EnvironmentFile=/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver  \
        $KUBE_ETCD_SERVERS \
        $KUBE_API_ADDRESS \
        $KUBE_API_PORT \
        $KUBE_SERVICE_ADDRESSES \
        $KUBE_ADMISSION_CONTROL \
        $KUBE_API_LOG \
        $KUBE_API_ARGS 
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
######################kube-apiserver.service######################

[root@iZod5e1vrxu935Z bin]# vim /etc/kubernetes/apiserver

######################apiserver######################
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
KUBE_API_PORT="--insecure-port=8080"
KUBE_ETCD_SERVERS="--etcd-servers=http://172.17.172.197:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=169.169.0.0/16"
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_LOG="--logtostderr=false --log-dir=/home/k8s-t/log/kubernets --v=2"
KUBE_API_ARGS=" "
######################apiserver######################

[root@iZod5e1vrxu935Z bin]# mkdir -p /etc/kubernetes
[root@iZod5e1vrxu935Z bin]# mkdir -p /home/k8s-t/log/kubernets

[root@iZod5e1vrxu935Z bin]# systemctl daemon-reload 
[root@iZod5e1vrxu935Z bin]# systemctl enable kube-apiserver.service
Created symlink from /etc/systemd/system/multi-user.target.wants/kube-apiserver.service to /usr/lib/systemd/system/kube-apiserver.service.
[root@iZod5e1vrxu935Z bin]# systemctl start kube-apiserver.service
[root@iZod5e1vrxu935Z bin]#  systemctl status kube-apiserver.service

#### 2)kube-controller-manager 配置
[root@iZod5e1vrxu935Z bin]# vim /usr/lib/systemd/system/kube-controller-manager.service
[root@iZod5e1vrxu935Z bin]# vim /etc/kubernetes/controller-manager
[root@iZod5e1vrxu935Z bin]# systemctl daemon-reload 
[root@iZod5e1vrxu935Z bin]# systemctl enable kube-controller-manager.service
[root@iZod5e1vrxu935Z bin]# systemctl start kube-controller-manager.service
[root@iZod5e1vrxu935Z bin]# systemctl status kube-controller-manager.service

#### 3)kube-scheduler 配置
[root@iZod5e1vrxu935Z bin]# vim /usr/lib/systemd/system/kube-scheduler.service
[root@iZod5e1vrxu935Z bin]# vim /etc/kubernetes/scheduler
[root@iZod5e1vrxu935Z bin]# systemctl daemon-reload 
[root@iZod5e1vrxu935Z bin]# systemctl enable kube-scheduler.service
Created symlink from /etc/systemd/system/multi-user.target.wants/kube-scheduler.service to /usr/lib/systemd/system/kube-scheduler.service.
[root@iZod5e1vrxu935Z bin]# systemctl start kube-scheduler.service
[root@iZod5e1vrxu935Z bin]# systemctl status kube-scheduler.service

#### 4)kubectl 配置
[root@iZod5e1vrxu935Z bin]# cp kubectl /usr/bin
[root@iZod5e1vrxu935Z bin]# kubectl version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:52:23Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
[root@iZod5e1vrxu935Z bin]# kubectl get componentstatus
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-0               Healthy   {"health": "true"}   

[root@iZod5e1vrxu935Z bin]# kubectl config set-cluster kubernetes --server=http://172.17.172.197:8080
Cluster "kubernetes" set.
[root@iZod5e1vrxu935Z bin]# kubectl config set-context kubernetes --cluster=kubernetes
Context "kubernetes" created.
[root@iZod5e1vrxu935Z bin]# kubectl config use-context kubernetes
Switched to context "kubernetes".

[root@iZod5e1vrxu935Z bin]# kubectl -s http://172.17.172.197:8080 version
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:52:23Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.1", GitCommit:"3a1c9449a956b6026f075fa3134ff92f7d55f812", GitTreeState:"clean", BuildDate:"2018-01-04T11:40:06Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}
[root@iZod5e1vrxu935Z bin]# kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
172.17.90.213   Ready     <none>    14s       v1.9.1
