### 安装依赖

```shell
yum install socat conntrack-tools -y
```

### 获取K8S安装资源（rpm包）

```shell
rpm -ivh kubeadm.rpm
rpm -ivh kubectl.rpm
rpm -ivh kubelet.rpm
rpm -ivh cni.rpm
rpm -ivh cri-tool.rpm
```

### 修改docker配置

```shell
cat << EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["example.xxx"],
  "insecure-registries": ["example.xxx"],
  "data-root": "/data/docker",
  "exec-opts": ["native.cgroupdrivver=systemd"]
}
systemctl daemon-reload
systemctl restart docker
```

### 下载K8S镜像

```shell
kubeadm config images pull 
# --image-repository 该参数可以指定repo 内网代理时使用
# 下载镜像后需要打tag，替换为k8s.gcr.io
# k8s.gcr.io/coredns/coredns:v1.8.6
# 该镜像通过代理时可能拉不下来，是因为代理的目录少了一级coredns
```

### 关闭交换分区

```shell
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

### 准备VIP

```shell
# 允许在master的三个节点间使用VIP来访问，并通过kubelet实现master的灾难迁移
docker pull ghcr.io/kube-vip/kube-vip:v0.4.0
mkdir -pv /etc/kubernetes/manifests/
# VIP_IP是你要使用的vip地址
export VIP=VIP_IP
# ens160是你要使用的网卡名称, 该镜像会使用ip addr将vip添加到网卡上
export INTERFACE=ens160
alias kube-vip="docker run --network host --rm ghcr.io/kube-vip/kube-vip:v0.4.0"
kube-vip manifest pod \
    --interface $INTERFACE \
    --address $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee /etc/kubernetes/manifests/kube-vip.yaml
  
kubeadm init \
  -control-plane-endpoint $INTERFACE \
  --apiserver-bind-port 6443 \
  --upload-certs \
  --kubernetes-version v1.23.4 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=SystemVerification

kubectl apply -f xxx/calico.yaml
```




