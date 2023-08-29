---
title: Kubernetes
date: 2022/12/24 20:24:31
updated: 2023/3/23 22:53:04
categories:
- DevOps
tags:
- k8s
---

## 安装

- 安装docker

- 准备

```bash
#各个机器设置自己的域名
hostnamectl set-hostname xxxx

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

```bash
echo "10.0.0.6  cluster-endpoint" >> /etc/hosts
```

- 初始化master节点

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward

kubeadm init \
--apiserver-advertise-address=10.0.0.6 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=169.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```

成功信息

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join cluster-endpoint:6443 --token ldz1g0.8dj4fljdtsfl2m4o \
    --discovery-token-ca-cert-hash sha256:5bfdb3147f6725c7cd7c1a66f3bac8ab5c314d91b50d7c9a4e864f1b1698218c \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token ldz1g0.8dj4fljdtsfl2m4o \
    --discovery-token-ca-cert-hash sha256:5bfdb3147f6725c7cd7c1a66f3bac8ab5c314d91b50d7c9a4e864f1b1698218c 
```

```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```bash
#查看集群所有节点
kubectl get nodes

#根据配置文件，给集群创建资源
kubectl apply -f xxxx.yaml

#查看集群部署了哪些应用？
docker ps   ===   kubectl get pods -A
# 运行中的应用在docker里面叫容器，在k8s里面叫Pod
kubectl get pods -A
```

```bash
curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml
```

## work节点加入master节点

```bash
# 防火墙设置
echo 1 > /proc/sys/net/ipv4/ip_forward
```

```bash
kubeadm token create --print-join-command
kubeadm join cluster-endpoint:6443 --token abt2er.44mplleknac1xy9i \
    --discovery-token-ca-cert-hash sha256:c0df301ef9f6969cbbee4c6dc9978b46748dee1fdedf8c418da53ceed5ace61a 
```

## 安装dashboard

- 下载yaml文件

```bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```

- 安装dashboard

```bash
kubectl apply -f recommended.yaml
```

- 设置访问端口

```bash
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

# 修改type: NodePort

# 获取访问端口
kubectl get svc -A |grep kubernetes-dashboard
```

- 创建访问账号

```yaml
#创建访问账号，准备一个yaml文件； vi dash-user.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```bash
kubectl apply -f dash-user.yaml
```

- 获取登录token

```bash
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IjlfZ2RNYzY0WUJWNzBSaVpjWTc2V3hveXFBUTFUTGZMNHYwZ05tYnpmLWsifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY5bmJtIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmNmM2YmJkOC04ZDljLTQ0MzItYjMwYi1iYTJhMDBiYzFkYTEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.F8-EjaSAVJjUF4dFx_5iIruadPwhVX-zqTh2vmXWNU6qaCcsCsbtm_RXoTPwI2zmBpk3LrxtofoTfpWZC4jNlO1x0bXgQcsmPNximBe12b01HrdzN8Ov_C8FjbDgffMkLBVl4Et91ZGjmzgyD7xnqUiC0dsuOQkcYAtNimtmBszi8w9vHR9_MfBTA5C_GOlTxcGtS8smpJn04W5-1lzaBF0D_gpZVxfpgVvV70aW7uvJ643n-qYGEoJCeSOyNMpykaauykost-S0QZ9dHD05XA9M2UPh2hyOubDSSGPNOevMd76bUiXiJ7sSx40mBIw0C7xjubXREXC3ULQDot-3Mw
```
![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/93771671884658935.png)






