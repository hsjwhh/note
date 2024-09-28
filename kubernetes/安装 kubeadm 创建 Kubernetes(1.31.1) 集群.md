**以下操作适用于 Kubernetes 1.31.1 版本，安装在全新安装的 Debian12 系统上**

### 1. 系统准备

#### 1.1 更新系统

确保你的系统软件包是最新的：

```bash
sudo apt update && sudo apt upgrade -y
```

#### 1.2 关闭 Swap

Kubernetes 要求禁用 Swap。

```bash
sudo swapoff -a
```

持久化禁用 Swap：

```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

#### 1.3 加载所需的内核模块

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

确保这些模块在重启后依旧加载：

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

#### 1.4 设置系统参数

配置网络设置：

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```

使其生效：

```bash
sudo sysctl --system
```

#### 1.5 安装 containerd 或其它 CRI

> 在 Kubernetes 1.31 中，`containerd` 是推荐的容器运行时。**建议配置添加镜像加速网站，详见《Containerd 配置拉取镜像加速》**

##### 1.5.1 安装

在 Debian 或 Ubuntu 上：

```bash
sudo apt-get update
sudo apt-get install -y containerd
```

在 CentOS 或 RHEL 上：

```bash
sudo yum install -y containerd
```

安装完成后，`containerd` 会自动启动。如果没有启动，可以使用以下命令：

```bash
sudo systemctl enable containerd
sudo systemctl start containerd
```

##### 1.5.2. 配置 containerd

- 生成 `containerd` 默认配置文件:
  默认情况下，`containerd` 是可以直接工作的，但为了适配 Kubernetes，你需要对其配置进行调整：

  ```bash
  sudo mkdir -p /etc/containerd
  containerd config default | sudo tee /etc/containerd/config.toml
  ```

- 修改配置文件 `/etc/containerd/config.toml`:
  找到 `plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options`，并将 `SystemdCgroup` 设置为 `true`。这个配置告诉 `containerd` 使用 `systemd` 管理容器的 cgroup，这对于与 Kubernetes 配合时非常重要。如下所示：

  ```toml
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
  ```

- 重新启动 containerd：

  ```bash
  sudo systemctl restart containerd
  ```

### 2. 安装 kubeadm, kubelet 和 kubectl [^1]

#### 2.1 添加 Kubernetes APT 仓库

**以下指令适用于 Kubernetes 1.31**

- 更新 `apt` 包索引并安装使用 Kubernetes `apt` 仓库所需要的包：

  ```bash
  sudo apt-get update
  # apt-transport-https 可能是一个虚拟包（dummy package）；如果是的话，你可以跳过安装这个包
  sudo apt-get install -y apt-transport-https ca-certificates curl gpg
  ```

- 下载用于 Kubernetes 软件包仓库的公共签名密钥。所有仓库都使用相同的签名密钥，因此你可以忽略 URL 中的版本：
  ```bash
  # 如果 `/etc/apt/keyrings` 目录不存在，则应在 curl 命令之前创建它，请阅读下面的注释。
  # sudo mkdir -p -m 755 /etc/apt/keyrings
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  ```

  > 在低于 Debian 12 和 Ubuntu 22.04 的发行版本中，`/etc/apt/keyrings` 默认不存在。 应在 curl 命令之前创建它。

#### 2.2 安装 Kubernetes 组件

- 添加 Kubernetes `apt` 仓库。
  请注意，此仓库仅包含适用于 Kubernetes 1.31 的软件包； 对于其他 Kubernetes 次要版本，则需要更改 URL 中的 Kubernetes 次要版本以匹配你所需的次要版本 （你还应该检查正在阅读的安装文档是否为你计划安装的 Kubernetes 版本的文档）。

  ```bash
  # 此操作会覆盖 /etc/apt/sources.list.d/kubernetes.list 中现存的所有配置。
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

- 更新 `apt` 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
  ```bash
  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  ```

  kubelet 现在每隔几秒就会重启，因为它陷入了一个等待 kubeadm 指令的死循环。

### 3. 部署 Kubernetes 控制平面

#### 3.1 初始化控制平面节点
首先确保主机的主机名和 IP 地址解析正确，自行设置主机命及固定 IP，必要时修改 Hosts 文件：
```bash
hostnamectl set-hostname master-node
```

在控制平面节点上运行以下命令：
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

初始化完成后，将会看到一段用于加入工作节点的命令输出，例如：
```bash
kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
> **保留此命令以供后续使用**。

#### 3.2 配置 kubectl（仅限控制平面节点）
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 4. 部署网络插件

Kubernetes 集群需要一个网络插件来管理 Pod 的通信。这里我们选择 Calico：
```bash
# 建议将文件下载下来后使用
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 5. 加入工作节点

#### 5.1 在每个工作节点上运行以下命令
```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 6. 验证集群状态

在控制平面节点上，确保所有节点都已加入并正常运行：
```bash
kubectl get nodes
```

所有节点的状态应该为 `Ready`，表示集群已成功启动。

[^1]: https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
