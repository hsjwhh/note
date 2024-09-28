# 部署 Kubernetes Dashboard

为了在已部署的 Kubernetes 集群上使用 Kubernetes Dashboard 进行可视化管理，可以使用最新的 Helm 安装方法。以下是详细的教程，专门针对已经配置好的 Kubernetes master 节点（在 Debian 12 上运行）。

### 前提条件
1. **Helm 已安装**：确保你已经在集群上安装了 Helm。如果未安装，可以执行以下命令来安装 Helm：

   ```bash
   # curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
   sudo apt-get install apt-transport-https --yes
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
   sudo apt-get update
   sudo apt-get install helm
   ```

2. **Kubernetes 已正确部署**：确认你的 Kubernetes 集群正常工作并且 `kubectl` 可用。

### 第一步：添加 Kubernetes Dashboard Helm 仓库
首先，需要将 Kubernetes Dashboard 仓库添加到 Helm 中。

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
```

### 第二步：部署 Kubernetes Dashboard

接下来，使用 Helm 部署 Dashboard。此步骤将在名为 `kubernetes-dashboard` 的命名空间中创建 Dashboard，并使用 Helm 安装它。

```bash
# 正常安装
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
# 如果上一步添加仓库由于网络无法访问，可以去 GitHub 下载最新的 dashboard release 版本安装
# https://github.com/kubernetes/dashboard/releases
# kubernetes-dashboard-7.6.1.tgz
# helm upgrade --install kubernetes-dashboard kubernetes-dashboard-7.6.1.tgz --create-namespace --namespace kubernetes-dashboard
```

### 第三步：验证部署

执行以下命令，检查 Dashboard 部署是否正常：

```bash
kubectl get pods -n kubernetes-dashboard
```

你应该能看到类似的输出，显示所有相关的 pods 处于 `Running` 状态。

### 第四步：创建访问凭据
> https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md#creating-a-service-account

Dashboard 默认不会提供外部访问，因此你需要创建一个服务账户和权限来访问它。

1. 创建服务账号，Creating a Service Account：

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

2. 创建绑定权限的角色，Creating a ClusterRoleBinding：

```yaml
cat <<EOF | kubectl apply -f -
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
EOF
```

3. 通过以下命令获取访问令牌，用于登录 Dashboard，Getting a Bearer Token for ServiceAccount：

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

4. 为ServiceAccount获取一个长效的Bearer

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"
type: kubernetes.io/service-account-token
EOF
```

5. 创建长效token后，我们可以执行以下命令来获取保存在 Secret 中的令牌

```bash
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```

### 第五步：访问 Dashboard

Kubernetes Dashboard 通常在集群内部运行，因此需要通过 `kubectl proxy` 进行本地访问：

```bash
kubectl proxy
```

在本地机器上，打开浏览器并访问以下 URL：

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

在登录页面中，选择使用 `Token` 进行登录，并粘贴之前生成的令牌。

### 第六步：使用外部访问（可选）

如果希望从集群外部访问 Dashboard，可以配置 Ingress 或暴露服务。

#### 例如，可以使用 `NodePort` 暴露 Dashboard：

```bash
KUBE_EDITOR="nano" kubectl edit svc  -n kubernetes-dashboard kubernetes-dashboard-kong-proxy
```

找到 `type: ClusterIP` 行并将其改为：

```yaml
type: NodePort
```

然后保存文件。执行 `kubectl get svc -n kubernetes-dashboard`，你会看到 `NodePort` 的分配端口号，通过 `<NodeIP>:<Port>` 可以从外部访问 Dashboard。

#### 或者可以创建一个
为了在外部访问 Kubernetes Dashboard，通常需要暴露 Dashboard 的服务。你可以按照以下步骤操作，创建一个合适的服务让外部访问。

##### 1. **暴露 Kubernetes Dashboard**

你可以为 Kubernetes Dashboard 创建一个 `NodePort` 或 `LoadBalancer` 类型的服务。以下是使用 `NodePort` 暴露 Dashboard 的步骤。

##### 暴露 Dashboard 服务为 NodePort

```bash
kubectl expose deployment kubernetes-dashboard-kong-proxy -n kubernetes-dashboard --type=NodePort --name=kubernetes-dashboard
```

执行后，这会创建一个名为 `kubernetes-dashboard` 的服务，类型为 `NodePort`，可以通过集群中的任意节点进行访问。

##### 2. **查看服务的端口**

运行以下命令查看新创建的服务信息：

```bash
kubectl get svc -n kubernetes-dashboard
```

你会看到 `kubernetes-dashboard` 服务的 `NodePort` 信息，比如：

```
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
kubernetes-dashboard   NodePort    10.110.161.135   <none>        443:32423/TCP                   1m
```

在这个例子中，`443:32423/TCP` 表示你可以通过集群中任何一个节点的 IP 地址，使用 `32423` 端口来访问 Dashboard。

##### 3. **访问 Dashboard**

现在，你可以在浏览器中通过以下 URL 访问 Kubernetes Dashboard：

```
https://<Node-IP>:<NodePort>
```

例如：

```
https://192.168.1.100:32423
```

记得替换 `<Node-IP>` 为你的节点 IP 地址，`<NodePort>` 为上一步中获得的实际端口号。


### 第七步：验证 Dashboard

通过访问上述 URL，可以使用 Token 登录并开始使用 Kubernetes Dashboard 管理你的集群。

### 可选配置：使用 SSL 证书

如果你需要 HTTPS 访问 Dashboard，可以参考 Kubernetes Dashboard 的 Helm 文档，添加自定义的 Ingress 控制器和 SSL 证书。

---

这是最新的基于 Helm 的 Kubernetes Dashboard 安装流程。如果你需要进一步的自定义或者更多高级选项，可以参考 [Kubernetes Dashboard 官方 Helm 文档][def]

[def]: https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard