# Kubernetes
## Ubuntu 搭建 k8s 集群
### 1 节点规划

| 角色     | IP            | 配置  |
| -------- | ------------- | ----- |
| console  | 192.168.10.3  | 4C/4G |
| master   | 192.168.10.11 | 4C/4G |
| worker01 | 192.168.10.12 | 4C/4G |
| worker02 | 192.168.10.13 | 4C/4G |

### 2 卸载 minikube
```
minikube delete --all

sudo rm -rf ~/.kube ~/.minikube &&
sudo rm -rf /usr/local/bin/minikube &&
sudo rm -rf /etc/kubernetes/ &&
sudo docker system prune -af --volumes

# 删除 kubectl 别名
vim ~/.bashrc
alias kubectl="minikube kubectl --"
```

### 3 配置 hostname
除 console 主机，其余主机根据上面节点规划，修改 hostname 为具体角色
```
sudo vi /etc/hostname

# 示例
liugx@master:~$ cat /etc/hostname
master

liugx@worker01:~$ cat /etc/hostname
worker01
```

### 4 安装并配置 docker
- 各节点安装并配置 docker
```
sudo apt install docker.io

# 示例
liugx@master:~$ docker version
Client:
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.2
 Git commit:        20.10.12-0ubuntu2~20.04.1
 Built:             Wed Apr  6 02:16:12 2022
 OS/Arch:           linux/arm64
 Context:           default
 Experimental:      true

Server:
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.2
  Git commit:       20.10.12-0ubuntu2~20.04.1
  Built:            Thu Feb 10 15:03:35 2022
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.5.9-0ubuntu1~20.04.6
  GitCommit:
 runc:
  Version:          1.1.0-0ubuntu1~20.04.2
  GitCommit:
 docker-init:
  Version:          0.19.0
  GitCommit:
```

- 在 “/etc/docker/daemon.json” 里把 cgroup 的驱动程序改成 systemd，然后重启 Docker 的守护进程
```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker && \
sudo systemctl daemon-reload && \
sudo systemctl restart docker
```

### 5 配置 iptables
- 为了让 Kubernetes 能够检查、转发网络流量，你需要修改 iptables 的配置，启用“br_netfilter”模块
```

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1 # better than modify /etc/sysctl.conf
EOF

sudo sysctl --system
```

### 6 关闭 swap 分区
```
sudo swapoff -a && \
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab

# 重启，如果是虚拟机的话，建议重启后拍个快照
sudo reboot

```

### 7 安装 kubeadm
- 1) 安装依赖，然后更换国内软件源
```
sudo apt install -y apt-transport-https ca-certificates curl

curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

sudo apt update
```
- 2) 获取 kubeadm、kubelet 和 kubectl 这三个安装必备工具
```
sudo apt install -y kubeadm=1.23.3-00 kubelet=1.23.3-00 kubectl=1.23.3-00

# 验证
kubeadm version
kubectl version --client
```

- 3) 锁定版本
```
sudo apt-mark hold kubeadm kubelet kubectl
```

### 8 安装 Master 节点
- 说明
```
kubeadm 的用法非常简单，只需要一个命令 kubeadm init 就可以把组件在 Master 节点上运行起来，不过它还有很多参数用来调整集群的配置，你可以用 -h 查看。
这里我只说一下我们实验环境用到的 3 个参数：
--image-repository，使用国内镜像仓库
--pod-network-cidr，设置集群里 Pod 的 IP 地址段。
--apiserver-advertise-address，设置 apiserver 的 IP 地址，对于多网卡服务器来说很重要（比如 VirtualBox 虚拟机就用了两块网卡），可以指定 apiserver 在哪个网卡上对外提供服务。
--kubernetes-version，指定 Kubernetes 的版本号。下面的这个安装命令里，我指定了 Pod 的地址段是“10.10.0.0/16”，apiserver 的服务地址是“192.168.10.210”，Kubernetes 的版本号是“1.23.3”：
```
```
sudo kubeadm init \
--image-repository registry.aliyuncs.com/google_containers \
--apiserver-advertise-address=192.168.10.11 \
--pod-network-cidr=10.10.0.0/16 \
--kubernetes-version v1.23.3
```
- 安装完成后，它还会提示出接下来要做的工作：
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

意思是要在本地建立一个“.kube”目录，然后拷贝 kubectl 的配置文件，你只要原样拷贝粘贴就行。
```

- 另外还有一个很重要的“kubeadm join”提示，其他节点要加入集群必须要用指令里的 token 和 ca 证书，所以这条命令务必拷贝后保存好：
```
kubeadm join 192.168.10.11:6443 --token ku5uju.pjrijinryujcjx7y \
	--discovery-token-ca-cert-hash sha256:8703bf76e896d0923be754618ee19f5cc53497cbe7ada8e14dae44bc11900dfe
```

- 验证
```
kubectl version
kubectl get node

你会注意到 Master 节点的状态是“NotReady”，这是由于还缺少网络插件，集群的内部网络还没有正常运作。
```

### 9 安装 Flannel 网络插件
- 说明
```
Kubernetes 定义了 CNI 标准，有很多网络插件，这里我选择最常用的 Flannel，可以在它的 GitHub 仓库里（https://github.com/flannel-io/flannel/）找到相关文档。它安装也很简单，只需要使用项目的“kube-flannel.yml”在 Kubernetes 里部署一下就好了。不过因为它应用了 Kubernetes 的网段地址，你需要修改文件里的“net-conf.json”字段，把 Network 改成刚才 kubeadm 的参数 --pod-network-cidr 设置的地址段。比如在这里，就要修改成“10.10.0.0/16”：
```
- vi kube-flannel.yml
```
# chrono @ 2022-04

# https://github.com/flannel-io/flannel/blob/master/Documentation/kube-flannel.yml

# change net-conf.json
#   "Network": "10.10.0.0/16"

---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.10.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
       #image: flannelcni/flannel-cni-plugin:v1.0.1 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
       #image: flannelcni/flannel:v0.17.0 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.17.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
       #image: flannelcni/flannel:v0.17.0 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.17.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
```

- 安装
```
kubectl apply -f kube-flannel.yml

# 等待一会儿执行
kubectl get node
```

# 10 安装 Worker 节点
- 用之前拷贝的那条 kubeadm join 命令就可以了，记得要用 sudo 来执行
```
sudo kubeadm join 192.168.10.11:6443 --token ku5uju.pjrijinryujcjx7y \
--image-repository registry.aliyuncs.com/google_containers \
--discovery-token-ca-cert-hash sha256:8703bf76e896d0923be754618ee19f5cc53497cbe7ada8e14dae44bc11900dfe
```

# 11 安装 console
- 它只需要安装一个 kubectl，然后复制“config”文件就行，你可以直接在 Master 节点上用“scp”远程拷贝，例如
```
scp `which kubectl` liugx@192.168.10.3:/usr/local/bin/
scp ~/.kube/config liugx@192.168.10.3:~/.kube/
```
